---
title: Reflect Nodes
weight: 1
description: >
  When your application objects and field names mostly align with your YANG model
---

NOTE: You don't need perfect alignment with names, any small variations can be managed by combining Reflect and [Extend]({{< relref "extend" >}}).  To that end, do not expect magical powers from Reflect to coerse your field types to YANG types, again, that's where [Extend]({{< relref "extend" >}}) comes in.

**Items of note:**
* Currently Reflect doesn't attempt to use reflection to implement `notifications` or `actions/rpcs` but again, you can use `Reflect` this `Extend`.
* Names in YANG can be `camelCase`, `kabob-case` or `snake_case` interchangablely

## Simple Example

Perfect alignment of field names align with names in YANG.

{{% side-by-side %}}
**If you have application code like this...**
```go
type App struct {
    Me            *User
    Users         []*User
    Size          int
    HarmlessValue string 
}

type User struct {
    FullName string
}
```
||
**...and YANG like this...**
```
module anyName {
    container me {
        // full-name, full_name or fullName all work
        leaf full-name {
            type string;
        }
    }
    list users {
        key full-name;
        // could be grouping/uses here, doesn't matter
        leaf full-name {
            type string;
        }
    }
    leaf size {
        type int32;
    }
}
```
{{% /side-by-side %}}

**...then your node code can be this.**
```go
app := &App{}
node := nodeutil.ReflectChild(app)
```

## Map Example

Reflect also supports Go's `map` interface. While this Go code's lack of data structures might make this difficult to use in Go, if you don't need to handle this data in Go, then this is completely acceptable.  Remember, you can add constraints to yang to validate data.

{{% side-by-side %}}
**If you have application code like this...**
```go
type App struct {
    Info map[string]interface{}
}
```
||
**...and YANG like this...**
```
module anyName {
    container info {
        leaf name {
            type string;
        }
        container more {
            leaf size {
                type decimal64;
            }
        }
        list stuff {
            key anything;
            leaf anything {
                type enumeraion {
                    enum one;
                    enum two;
                }
            }
        }
    }
}
```
{{% /side-by-side %}}

**...then your node code can be this.**
```go
app := &App{}
node := nodeutil.ReflectChild(app)
```

## Field Coersion

Reflect can convert strings to numbers, numbers to enums, enums to string, number types to other number types.  If a conversion represents a loss of data or a type can cannot be convert safely, then an error is returned.  To handle the conversion of these values yourself, you can simply use Extend.  If however you have a lot of fields of the same type can can use the `OnField` 

{{% side-by-side %}}
**If you have application code like this...**
```go
type App struct {
    LastModified time.Time
}
```
||
**...and YANG like this...**
```
module anyName {
    leaf lastModified {        
        type int64; // unix seconds
    }
}
```
{{% /side-by-side %}}

**...then your node code can be this.**
```go
timeHandler := nodeutil.ReflectField{
    When: nodeutil.ReflectFieldByType(reflect.TypeOf(time.Time{})),
    OnRead: func(leaf meta.Leafable, fieldname string, elem reflect.Value, fieldElem reflect.Value) (val.Value, error) {
        t := fieldElem.Value().(time.Time)
        return val.Int64(t.Unix()), nil
    },
    OnWrite: func(leaf meta.Leafable, fieldname string, elem reflect.Value, fieldElem reflect.Value, v val.Value) error {
        t := time.Unix(v.Value.(int64), 0)
        fieldElem.Set(t)
        return nil
    },
}
node := nodeutil.Reflect{OnField: []nodeutil.ReflectField{timeHandler}}.Node(app)
```

## Starting with a List Example

As we've seen, when `Reflect` comes across a list it knows how to handle adding, removing and navigating.  If however, you are mixing node types (e.g. Basic, Extend, etc.) and you want to match a YANG `list` and to a Go `slice`, you can use this option.

```go
updateCubs := func(v reflect.Value) {
    bear.Cubs = v.Value.([]*Cub)
}
node := nodeutil.ReflectList(cubs, updateCubs)
```

