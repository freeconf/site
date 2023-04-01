---
title: Node/Basic
weight: 30
tags:
  - go
  - server
  - node
description: >
  Building Go nodes by using abstract class
---

## Use cases:
* high-level routing areas
* `list` nodes
* areas with not a lot of CRUD
* bridges to systems that are not Go structs (e.g. DB, YAML, external REST APIs, etc.)
* part of code generation

## Highlevel routing

**Go Code**
```go
package demo

type App struct {
	users  *UserService
	fonts  *FontManager
	bagels *BagelMaker
}

type UserService struct{}
type FontManager struct{}
type BagelMaker struct{}

```

**YANG**
```
module my-app {
    namespace "freeconf.org";
    prefix "a";

    container users {
        // ... more stuff here
    }
    container fonts {
        // ... more stuff here
    }
    container bagels {
        // ... more stuff here
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

func manage(a *App) node.Node {
	return &nodeutil.Basic{
		OnChild: func(r node.ChildRequest) (node.Node, error) {
			switch r.Meta.Ident() {
			case "users":
				return nodeutil.ReflectChild(a.users), nil
			case "fonts":
				return nodeutil.ReflectChild(a.fonts), nil
			case "bagels":
				return nodeutil.ReflectChild(a.bagels), nil
			}
			return nil, nil
		},
	}
}

```

You cannot use `Reflect` here because fields are private.

### Additional Files
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
	a := &App{}
	ypath := source.Dir(".")
	m := parser.RequireModule(ypath, "my-app")
	bwsr := node.NewBrowser(m, manage(a))

	root := bwsr.Root()
	actual, err := nodeutil.WriteJSON(root)
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, `{}`, actual)
}

```
