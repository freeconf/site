---
title: Node/maps
tags:
  - go
  - restconf
  - server
  - node
weight: 36  
description: >
  Mapping a Go `map[string]*Friend` to a YANG `list`
---

Handing YANG `lists` are bit more complex then handling YANG `containers`.  As usual, FreeCONF imposes no limits on you regarding how to iterate your list.

## Use Cases
* A map of Go structures that are editable.

[Full Source](https://github.com/freeconf/examples/node-list-map)

file: `chipmunk.yang`
```
module chipmunk {
	namespace "freeconf.org";
	prefix "c";

	list friends {
		key name;
		
		leaf name {
			type string;
		}
	}
}
```

```go
package chipmonk

import (
	"reflect"
	"strings"

	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/val"
)

func manage(cmunk *Chipmunk) node.Node {
	return &nodeutil.Basic{
		OnChild: func(r node.ChildRequest) (child node.Node, err error) {
			switch r.Meta.Ident() {
			case "friends":
				return manageChipmunkFriends(cmunk), nil
			}
			return nil, nil
		},
	}
}

// While this is a bit of code, having the ability to customize any part of this
// is rather important.  You will likely find patterns and ways to reuse code here
// or maybe even use Go generics.
func manageChipmunkFriends(cmunk *Chipmunk) node.Node {
	return &nodeutil.Basic{

		// Use OnNextItem instead of OnNext when you
		OnNextItem: func(r node.ListRequest) nodeutil.BasicNextItem {

			// sort by name. You can lazy load this in GetByRow
			// if you only want to build this if nec.
			index := node.NewIndex(cmunk.Friend)
			index.Sort(func(a, b reflect.Value) bool {
				return strings.Compare(a.String(), b.String()) < 0
			})

			// keep a reference depending on if found by row, by key or
			// just created
			var found *Friend

			return nodeutil.BasicNextItem{
				New: func() error {
					name := r.Key[0].String()
					found = &Friend{Name: name}
					cmunk.Friend[name] = found
					return nil
				},
				GetByKey: func() error {
					name := r.Key[0].String()
					found = cmunk.Friend[name]
					return nil
				},
				DeleteByKey: func() error {
					name := r.Key[0].String()
					delete(cmunk.Friend, name)
					return nil
				},
				GetByRow: func() ([]val.Value, error) {
					if r.Row < index.Len() {
						if x := index.NextKey(r.Row); x != node.NO_VALUE {
							name := x.String()
							found = cmunk.Friend[name]
							return []val.Value{val.String(name)}, nil
						}
					}
					return nil, nil
				},
				Node: func() (node.Node, error) {
					if found != nil {
						return nodeutil.ReflectChild(found), nil
					}
					return nil, nil
				},
			}
		},
	}
}

```

## Additional Source

file: `chipmunk.go`
```go
package chipmonk

type Friend struct {
	Name string
}

type Chipmunk struct {
	Friend map[string]*Friend
}

```

file: `manage_test.go`
```go
package chipmonk

import (
	"testing"

	"github.com/freeconf/yang/fc"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/parser"
	"github.com/freeconf/yang/source"
)

func TestManage(t *testing.T) {
	cmunk := &Chipmunk{Friend: map[string]*Friend{"joe": {Name: "joe"}}}
	ypath := source.Dir(".")
	m := parser.RequireModule(ypath, "chipmunk")
	b := node.NewBrowser(m, manage(cmunk))

	actual, err := nodeutil.WriteJSON(b.Root())
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, `{"friends":[{"name":"joe"}]}`, actual)
}

```
