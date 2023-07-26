---
title: Node/Reflect
weight: 1
tags:
  - node
  - go
weight: 31
description: >
  When your application objects and field names mostly align with your YANG model
---

## Use cases: 
* CRUD on Go structs
* CRUD on Go maps or slices

## Special notes
* You don't need perfect alignment with Go field names and YANG to use `Reflect`. Let `Reflect` do the heavy lifting for you and capture small variations by combining with [Extend]({{< relref "../node-extend" >}}).  To that end, do not expect magical powers from `Reflect` to coerse your custom field types to YANG types.
* Currently `Reflect` doesn't attempt to use reflection to implement `notifications` or `actions/rpcs` but again, you can combine `Reflect` with `Extend`.
* Names in YANG can be `camelCase`, `kabob-case` or `snake_case` interchangablely. Your Go public field are obviously in `CamelCase`.

## Simple example

When you happen to have perfect alignment of field names to names in YANG.

**If you have application code like this...**
```go
package demo

type Contacts struct {
	Me            *User
	Users         []*User
	Size          int
	HarmlessValue string
}

type User struct {
	FullName string
}

```

**...and YANG like this...**
```
module contacts {
    
    namespace "freeconf.org";
    prefix "a";

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

**...then your node code can be this.**
```go
package demo

import (
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
)

func manage(app *Contacts) node.Node {
	return nodeutil.ReflectChild(app)
}

```

**...and you test like this.**
```go
package demo

import (
	"testing"

	"github.com/freeconf/yang/fc"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/parser"
	"github.com/freeconf/yang/source"
)

func TestManageContact(t *testing.T) {
	app := &Contacts{}
	ypath := source.Dir(".")
	m := parser.RequireModule(ypath, "contacts")
	b := node.NewBrowser(m, manage(app))

	actual, err := nodeutil.WriteJSON(b.Root())
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, `{"size":0}`, actual)
}

```


## Map example

Reflect also supports Go's `map` interface. While this Go code's lack of data structures that might make this difficult to use in Go, if you don't need to handle this data in Go, then this is completely acceptable.  Remember, you can add constraints to yang to ensure the data is validated properly.  

**If you have application code like this...**
```go
package demo

type JunkDrawer struct {
	Info map[string]interface{}
}

```
||
**...and YANG like this...**
```
module junk-drawer {
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
                type enumeration {
                    enum one;
                    enum two;
                }
            }
        }
    }
}

```

**...then your node code can be this.**
```go
package demo

import (
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
)

func manageJunkDrawer(app *JunkDrawer) node.Node {
	return nodeutil.ReflectChild(app)
}

```

**...and you test like this.**
```go
package demo

import (
	"testing"

	"github.com/freeconf/yang/fc"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/parser"
	"github.com/freeconf/yang/source"
)

func TestManageJunk(t *testing.T) {
	app := &JunkDrawer{Info: make(map[string]interface{})}
	ypath := source.Dir(".")
	m := parser.RequireModule(ypath, "junk-drawer")
	b := node.NewBrowser(m, manageJunkDrawer(app))

	actual, err := nodeutil.WriteJSON(b.Root())
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, `{"info":{}}`, actual)
}

```

**tips:** 
* useful for building validated RESTCONF APIs quickly to be filled in later with structs
* good for handling a bulk set of configs

## Field coersion

Reflect can do a decent job converting Go primatives to and from YANG leaf types: strings to numbers, numbers to enums, enums to strings, number types to other number types, etc..  If a conversion represents a loss of data or a type can cannot be convert safely, then an error is returned.  To handle the conversion of these values yourself, you can use `Extend` or `Reflect.OnField` .  `Reflect.OnField` is more suited over `Extend` when have a lot of fields of the same type that you want to reuse in a lot of places and not one-offs.

**If you have application code like this...**
```go
package demo

import "time"

type Timely struct {
	LastModified time.Time
}

```
**...and YANG like this...**
```
module timely {
    namespace "freeconf.org";
    prefix "t";
    
    leaf lastModified {        
        type int64; // unix seconds
    }
}
```


**...then your node code can be this.**
```go
package demo

import (
	"reflect"
	"time"

	"github.com/freeconf/yang/meta"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/val"
)

var timeHandler = nodeutil.ReflectField{
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

func manageTimely(t *Timely) node.Node {
	return nodeutil.Reflect{
		OnField: []nodeutil.ReflectField{
			timeHandler,
		},
	}.Object(t)
}

```

## Adhoc structs

Create an anonymous struct or a just a map.  Useful for handing RPC requests or responses. Here we use it to create whole app.

```go
package demo

import (
	"testing"

	"github.com/freeconf/yang/fc"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/parser"
)

// Load a YANG module from a string!
var mstr = `
module mystery {
	namespace "freeconf.org";
	prefix "m";

	leaf name {
		type string;
	}
}
`

func TestMystery(t *testing.T) {

	m, err := parser.LoadModuleFromString(nil, mstr)
	fc.AssertEqual(t, nil, err)

	t.Run("struct", func(t *testing.T) {

		// create a node from an adhoc struct
		app := struct {
			Name string
		}{
			"john",
		}
		manage := nodeutil.ReflectChild(&app)

		// verify works
		b := node.NewBrowser(m, manage)
		actual, err := nodeutil.WriteJSON(b.Root())
		fc.AssertEqual(t, nil, err)
		fc.AssertEqual(t, `{"name":"john"}`, actual)
	})

	t.Run("map", func(t *testing.T) {

		// create a node from map
		app := map[string]interface{}{"name": "mary"}
		manage := nodeutil.ReflectChild(app)

		// verify works
		b := node.NewBrowser(m, manage)
		actual, err := nodeutil.WriteJSON(b.Root())
		fc.AssertEqual(t, nil, err)
		fc.AssertEqual(t, `{"name":"mary"}`, actual)
	})
}

```

