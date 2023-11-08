---
title: Development Guide
weight: 2
tags:
  - go
  - python
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

**Code**
{{< tabs name="app_src" >}}
{{% tab name="Go" %}}
```go
package demo

type App struct {
	users  *UserService
	fonts  *FontManager
	bagels *BagelMaker
}

func NewApp() *App {
	return &App{
		users:  &UserService{},
		fonts:  &FontManager{},
		bagels: &BagelMaker{},
	}
}

type UserService struct{}
type FontManager struct{}
type BagelMaker struct{}

```
{{% /tab %}}
{{% tab name="Python" %}}
```python

class App:

    def __init__(self):
        self.users = {}
        self.fonts = {}
        self.bagels = {}

```
{{% /tab %}}
{{< /tabs >}}

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

{{< tabs name="manage_src" >}}
{{% tab name="Go" %}}
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
{{% /tab %}}

{{% tab name="Python" %}}
```python
from freeconf import nodeutil

def manage_app(app):

    def child(req):
        if req.meta.ident == 'users':
            return nodeutil.Node(app.users)
        elif req.meta.ident == 'fonts':
            return nodeutil.Node(app.fonts)
        elif req.meta.ident == 'bagels':
            return nodeutil.Node(app.bagels)
        return None
    
    # while this could easily be nodeutil.Node, we illustrate a Basic
    # node should you want essentially an abstract class that stubs all 
    # this calls with reasonable default handlers
    return nodeutil.Basic(on_child=child)

```
{{% /tab %}}
{{< /tabs >}}


### Additional Files
{{< tabs name="test_src" >}}

{{% tab name="Go" %}}
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
	a := NewApp()
	ypath := source.Dir(".")
	m := parser.RequireModule(ypath, "my-app")
	bwsr := node.NewBrowser(m, manage(a))

	root := bwsr.Root()
	defer root.Release()
	actual, err := nodeutil.WriteJSON(root)
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, `{"users":{},"fonts":{},"bagels":{}}`, actual)
}

```
{{% /tab %}}

{{% tab name="Python" %}}
file: `test_manage.py`
```python
#!/usr/bin/env python3
import unittest 
from freeconf import parser, nodeutil, node, source
from manage import manage_app
from app import App 

class TestManage(unittest.TestCase):

    def test_manage(self):
        app = App()
        ypath = source.path("..")
        m = parser.load_module_file(ypath, 'my-app')
        mgmt = manage_app(app)
        bwsr = node.Browser(m, mgmt)
        root = bwsr.root()
        try:
            actual = nodeutil.json_write_str(root)
            self.assertEqual('{"users":{},"fonts":{},"bagels":{}}', actual)
        finally:
            root.release()

if __name__ == '__main__':
    unittest.main()

```
{{% /tab %}}
{{< /tabs >}}

