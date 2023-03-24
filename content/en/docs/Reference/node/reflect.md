---
title: Reflect Nodes
weight: 1
description: >
  When your application objects and field names mostly align with your YANG model
---

## Usecases: 
* CRUD to Go structs
* CRUD to Go maps or slices

## Special Notes
* You don't need perfect alignment with Go field names and YANG to use `Refect`. Let `Reflect` do the bulk of heavy lifting for you and capture small variations by combining with [Extend]({{< relref "extend" >}}).  To that end, do not expect magical powers from `Reflect` to coerse your custom field types to YANG types.
* Currently `Reflect` doesn't attempt to use reflection to implement `notifications` or `actions/rpcs` but again, you can combine `Reflect` with `Extend`.
* Names in YANG can be `camelCase`, `kabob-case` or `snake_case` interchangablely. Your Go public field are obviously in `CamelCase`.

## Simple Example

When you happen to have perfect alignment of field names to names in YANG.

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

Reflect also supports Go's `map` interface. While this Go code's lack of data structures that might make this difficult to use in Go, if you don't need to handle this data in Go, then this is completely acceptable.  Remember, you can add constraints to yang to ensure the data is validated properly.  

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
app.Info = make(map[string]interface{})
n := nodeutil.ReflectChild(app)
```

**tips:** 
* useful for building validated RESTCONF APIs quickly to be filled in later with structs
* good for handling a bulk set of configs

## Field Coersion

Reflect can do a decent job converting Go primatives to and from YANG leaf types: strings to numbers, numbers to enums, enums to strings, number types to other number types, etc..  If a conversion represents a loss of data or a type can cannot be convert safely, then an error is returned.  To handle the conversion of these values yourself, you can use `Extend` or `Reflect.OnField` .  `Reflect.OnField` is more suited over `Extend` when have a lot of fields of the same type that you want to reuse in a lot of places and not one-offs.

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
        t := fieldElem.Interface().(time.Time)
        return val.Int64(t.Unix()), nil
    },
    OnWrite: func(leaf meta.Leafable, fieldname string, elem reflect.Value, fieldElem reflect.Value, v val.Value) error {
        t := time.Unix(v.Value().(int64), 0)
        fieldElem.Set(reflect.ValueOf(t))
        return nil
    },
}
n := nodeutil.Reflect{OnField: []nodeutil.ReflectField{timeHandler}}.Object(myObj)
```

**tips:** 
* You can use a variety of criteria for `When` including information defined in YANG like `units` or YANG extensions.
* You can create a library of handlers and combine them as needed.
* Can be used to access private fields.
* remember, the `OnField` handlers are copied to each nested level of the heirarchy

## Starting with a List Example

As we've seen, when `Reflect` comes across a list it knows how to handle adding, removing and navigating.  If however, you are mixing node types (e.g. `Basic`, `Extend`, etc.) and you want to match a YANG `list` and directly to a Go `slice`, you can use this option.

```go
updateCubs := func(v reflect.Value) {
    bear.Cubs = v.Value().([]*Cub)
}
n := nodeutil.ReflectList(bear.Cubs, updateCubs)
```

