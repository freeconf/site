---
title: Unit Testing
weight: 101
description: >
  Strategies unit testing management `nodes`
---

You don't need any special utilities to unit test your management, just a few useful techniques.  If you look thru a lot of the [node implementations](/tags/node/) you'll see a lot of unit tests exibiting how to test nodes.

## Testing without full application

As your application, management and YANG file grow, loading the full application each time just to test a piece might become cumbersome. We can use a feature of YANG to import from another YANG file and FreeCONF's ability to make this easy to test just our unit.

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

type Here struct {
	Penny int
}

func TestUnitTestingPartialYang(t *testing.T) {

	// where car.yang is stored
	ypath := source.Dir(".")

	// Define new YANG module on the fly that references the application
	// YANG file but we pull in just what we want
	m, err := parser.LoadModuleFromString(ypath, `
		module x {
			import deep {
				prefix "d";
			}

			// pull in just the piece we are interested in. Here it is
			// just a single penny
			uses d:here;
		}
	`)
	if err != nil {
		t.Fatal(err)
	}

	// We create a "browser" to just a unit of our application
	h := &Here{Penny: 1}
	manage := node.NewBrowser(m, nodeutil.ReflectChild(h))

	// Example : test getting config
	actual, err := nodeutil.WriteJSON(manage.Root())
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, `{"penny":1}`, actual)
}

```
