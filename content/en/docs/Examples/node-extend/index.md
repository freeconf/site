---
title: Node/Extend
weight: 32
tags:
  - go
  - server
  - node
description: >
  When you want to make adjustments to an existing node's implementation
---

## Use cases:
* Few isolated changes to an existing node
* Wrap a CRUD node but customize editing operations, actions or notifications

## Special notes
* If [`Reflect`]({ { < relref "reflect" > } } ) was a cake and [`Basic`]({ { < relref "basic" > } } ) was the plate under the cake, then `Extend` would be the frosting.
* `Extend` is exactly like `Basic` but let's you delegate anything to another node.  So most of `Basic's` documentation also applies here.

## `Reflect` with one exception

```go
package demo

type Bird struct {
	Name     string
	Location Coordinates
}

type Coordinates struct{}

func (Coordinates) Set(string) {
}

func (Coordinates) Get() string {
	return "0,0"
}

```

```
module bird {
	namespace "freeconf.org";
	prefix "b";

	leaf name {
		type string;
	}
	leaf location {
		type string {
			pattern "[0-9.]+,[0-9.]+";
		}
	}
}

```

```go
package demo

import (
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/val"
)

func manage(b *Bird) node.Node {
	return &nodeutil.Extend{
		Base: nodeutil.ReflectChild(b), // handles Name
		OnField: func(p node.Node, r node.FieldRequest, hnd *node.ValueHandle) error {
			switch r.Meta.Ident() {
			case "location":
				if r.Write {
					b.Location.Set(hnd.Val.String())
				} else {
					hnd.Val = val.String(b.Location.Get())
				}
			default:
				// delegates to ReflectChild
				return p.Field(r, hnd)
			}
			return nil
		},
	}
}

```

### Addition Files

file: `manage_test.go`
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

func TestManage(t *testing.T) {
	b := &Bird{}
	ypath := source.Dir(".")
	m := parser.RequireModule(ypath, "bird")
	bwsr := node.NewBrowser(m, manage(b))

	root := bwsr.Root()
	actual, err := nodeutil.WriteJSON(root)
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, `{"location":"0,0"}`, actual)
}

```