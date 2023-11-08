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


{{< tabs name="app_src" >}}
{{% tab name="Go" %}}
```go
package demo

import "fmt"

type Bird struct {
	Name string
	X    int
	Y    int
}

func (b *Bird) GetCoordinates() string {
	return fmt.Sprintf("%d,%d", b.X, b.Y)
}

```
{{% /tab %}}
{{% tab name="Python" %}}
```python

class Bird():

    def __init__(self, name, x, y):
        self.name = name
        self.x = x
        self.y = y

    def coordinates(self):
        return f'{self.x},{self.y}'
```
{{% /tab %}}
{{< /tabs >}}


```
module bird {
	namespace "freeconf.org";
	prefix "b";

	leaf name {
		type string;
	}

	leaf location {
		config false;
		type string {
			pattern "[0-9.]+,[0-9.]+";
		}
	}
}

```

{{< tabs name="manage_src" >}}
{{% tab name="Go" %}}
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
				hnd.Val = val.String(b.GetCoordinates())
			default:
				// delegates to ReflectChild for name
				return p.Field(r, hnd)
			}
			return nil
		},
	}
}

```
{{% /tab %}}
{{% tab name="Python" %}}
```python
from freeconf import nodeutil, val

def manage_bird(bird):
    base = nodeutil.Node(bird)
    
    def on_field(parent, req, write_val):
        if req.meta.ident == "location":
            return val.Val(bird.coordinates())
        return parent.field(req, write_val)
    
    return nodeutil.Extend(base, on_field=on_field)
```
{{% /tab %}}
{{< /tabs >}}

### Addition Files

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
	b := &Bird{Name: "sparrow", X: 99, Y: 1000}
	ypath := source.Dir(".")
	m := parser.RequireModule(ypath, "bird")
	bwsr := node.NewBrowser(m, manage(b))

	root := bwsr.Root()
	defer root.Release()
	actual, err := nodeutil.WriteJSON(root)
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, `{"name":"sparrow","location":"99,1000"}`, actual)
}

```
{{% /tab %}}
{{% tab name="Python" %}}
file: `test_manage.py`
```python
#!/usr/bin/env python3
import unittest 
from freeconf import parser, nodeutil, source, node
from manage import manage_bird
from bird import Bird 

class TestManage(unittest.TestCase):

    def test_manage(self):
        app = Bird("sparrow", 99, 1000)
        ypath = source.path('..')
        m = parser.load_module_file(ypath, 'bird')
        mgmt = manage_bird(app)
        bwsr = node.Browser(m, mgmt)
        root = bwsr.root()
        try:
            actual = nodeutil.json_write_str(root)
            self.assertEqual('{"name":"sparrow","location":"99,1000"}', actual)
        finally:
            root.release()

if __name__ == '__main__':
    unittest.main()

```
{{% /tab %}}
{{< /tabs >}}
