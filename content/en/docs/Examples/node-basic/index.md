---
title: Node/Basic
weight: 30
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
import freeconf.nodeutil


class ManageApp(freeconf.nodeutil.Basic):

    def __init__(self, app):
        super().__init__()
        self.app = app        

    def child(self, req):
        if req.meta.ident == 'users':
            return freeconf.nodeutil.Reflect(self.app.users)
        elif req.meta.ident == 'fonts':
            return freeconf.nodeutil.Reflect(self.app.fonts)
        elif req.meta.ident == 'bagels':
            return freeconf.nodeutil.Reflect(self.app.bagels)
        return None

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
import freeconf.parser
import freeconf.nodeutil
from manage import ManageApp
from app import App 

class TestManage(unittest.TestCase):

    def test_manage(self):
        app = App()
        p = freeconf.parser.Parser()
        m = p.load_module('..', 'my-app')
        mgmt = ManageApp(app)
        bwsr = freeconf.node.Browser(m, mgmt)
        root = bwsr.root()
        try:
            root.upsert_into(freeconf.nodeutil.json_write("tmp"))
        finally:
            root.release()
        with open("tmp", "r") as f:
            self.assertEqual('{"users":{},"fonts":{},"bagels":{}}', f.read())


if __name__ == '__main__':
    unittest.main()

```
{{% /tab %}}
{{< /tabs >}}

