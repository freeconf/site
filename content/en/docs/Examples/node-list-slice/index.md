---
title: Node/slices
tags:
  - go
  - node
weight: 37  
description: >
  Mapping a Go `[]*Cub` to a YANG `list`
---

Handing YANG `lists` are bit more complex then handling YANG `containers`.  As usual, FreeCONF imposes no limits on you regarding how to iterate your list.

## Use Cases
* A slice of Go structures that are editable.

[Full Source](https://github.com/freeconf/examples/node-list-slice)


## Read-only

file: `bear-ro.yang`
```
module bear-ro {
    namespace "freeconf.org";
    prefix "b";

    list cub {
        key name;
        config false;
        leaf name {
            type string;
        }
    }
}
```

```go
package bear

import (
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/val"
)

func manageReadOnly(b *Bear) node.Node {
	return &nodeutil.Basic{
		OnChild: func(r node.ChildRequest) (child node.Node, err error) {
			switch r.Meta.Ident() {
			case "cub":
				return manageCubsReadOnly(b), nil
			}
			return nil, nil
		},
	}
}

func manageCubsReadOnly(b *Bear) node.Node {
	return &nodeutil.Basic{
		OnNext: func(r node.ListRequest) (node.Node, []val.Value, error) {
			key := r.Key
			var found *Cub

			if key != nil {
				name := key[0].String()
				for _, cub := range b.Cubs {
					if cub.Name == name {
						found = cub
						break
					}
				}
			} else if r.Row < len(b.Cubs) {
				found = b.Cubs[r.Row]
				key = []val.Value{val.String(found.Name)}
			}
			if found != nil {
				return nodeutil.ReflectChild(found), key, nil
			}
			return nil, nil, nil
		},
	}
}

```

### Additional Source

file: `bear.go`
```go
package bear

type Cub struct {
	Name string
}

type Bear struct {
	Cubs []*Cub
}

```

file: `manage_ro_test.go`
```go
package bear

import (
	"testing"

	"github.com/freeconf/yang/fc"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/parser"
	"github.com/freeconf/yang/source"
)

func TestManageReadOnly(t *testing.T) {
	bear := &Bear{Cubs: []*Cub{{Name: "bubbie"}}}
	ypath := source.Dir(".")
	m := parser.RequireModule(ypath, "bear-ro")
	b := node.NewBrowser(m, manageReadOnly(bear))

	actual, err := nodeutil.WriteJSON(b.Root())
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, `{"cub":[{"name":"bubbie"}]}`, actual)
}

```

## Read/Write

file: `bear.yang`
```
module bear {
    namespace "freeconf.org";
    prefix "b";

    list cub {
        key name;
        leaf name {
            type string;
        }
    }
}
```

```go
package bear

import (
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/val"
)

func manage(b *Bear) node.Node {
	return &nodeutil.Node{
		Object: b,
		Options: nodeutil.NodeOptions{
			TryPluralOnLists: true,
		},
	}
}

func managex(b *Bear) node.Node {
	return &nodeutil.Basic{
		OnChild: func(r node.ChildRequest) (child node.Node, err error) {
			switch r.Meta.Ident() {
			case "cub":
				return manageCubs(b), nil
			}
			return nil, nil
		},
	}
}

func manageCubs(b *Bear) node.Node {
	return &nodeutil.Basic{
		OnNextItem: func(r node.ListRequest) nodeutil.BasicNextItem {
			var found *Cub
			return nodeutil.BasicNextItem{
				New: func() error {
					name := r.Key[0].String()
					found = &Cub{Name: name}
					b.Cubs = append(b.Cubs, found)
					return nil
				},
				GetByKey: func() error {
					name := r.Key[0].String()
					for _, cub := range b.Cubs {
						if cub.Name == name {
							found = cub
							break
						}
					}
					return nil
				},
				DeleteByKey: func() error {
					name := r.Key[0].String()
					copy := make([]*Cub, 0, len(b.Cubs))
					for _, cub := range b.Cubs {
						if cub.Name != name {
							copy = append(copy, cub)
						}
					}
					b.Cubs = copy
					return nil
				},
				GetByRow: func() ([]val.Value, error) {
					var key []val.Value
					if r.Row < len(b.Cubs) {
						found = b.Cubs[r.Row]
						key = []val.Value{val.String(found.Name)}
					}
					return key, nil
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

### Additional Source

file: `manage_test.go`
```go
package bear

import (
	"testing"

	"github.com/freeconf/yang/fc"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/parser"
	"github.com/freeconf/yang/source"
)

func TestManage(t *testing.T) {
	bear := &Bear{Cubs: []*Cub{{Name: "bubbie"}}}
	ypath := source.Dir(".")
	m := parser.RequireModule(ypath, "bear")
	b := node.NewBrowser(m, manage(bear))

	actual, err := nodeutil.WriteJSON(b.Root())
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, `{"cub":[{"name":"bubbie"}]}`, actual)
}
