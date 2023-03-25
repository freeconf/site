---
title: Unit testing
weight: 202
description: >
  Easy way to test you management API.
---

You don't need any special utilities to unit test your management, just a few useful techniques.  

## Testing full application management

You just create a management browser into your application without registering it with a device or a server.

```go

import (
	"testing"

	"github.com/freeconf/restconf/testdata"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/parser"
	"github.com/freeconf/yang/source"
)

// Your Application
type SuperHero struct {
	Name string
}

type MarvelStudios struct {
	SuperHeros []*SuperHero
}

// Your Application Management
func manageStudio(studio *MarvelStudios) node.Node {
	return nodeutil.ReflectChild(studio)
}

// Application Management Unit Test
func TestUnitTest(t *testing.T) {
	// Just loading your module in your unit test verifies there is no syntax errors
	ypath := source.Dir("./yang")
	module := parser.RequireModule(ypath, "marvel-studios")

	// Create your app like you normally would
	studio := &MarvelStudios{}

	// Create a management into your app. This doesn't start a server, and it doesn't
	// even touch your app until you tell it to
	manage := node.NewBrowser(module, manageStudio(studio))

	// load a config you want to test or maybe load a config to
	// setup a different test
	cfg := nodeutil.ReadJSON(`{
		"super-heros":[{
			"name" : "spidey"
		}]
	}`)
	manage.Root().UpsertFrom(cfg)

	// you have access to your app, verify data is in there.
	if studio.SuperHeros[0].Name != "spidey" {
		t.Fail()
	}
}
```

## Testing a unit of management

As your application, management and YANG file grow, loading the full application each time just to test a piece might become cumbersome. We can use a feature of YANG to import from another YANG file and FreeCONF's ability to make this easy to test just our unit.


```go
func TestUnitTestingPartialYang(t *testing.T) {

	// where car.yang is stored
	ypath := source.Dir("yang")

	// Define new YANG module on the fly that references the application
	// YANG file but we pull in just what we want
	m, err := parser.LoadModuleFromString(ypath, `
		module x {
			import car {
				prefix "c";
			}

			// pull in just the piece we are interested in. Here it is
			// just a single tire
			uses c:tire;
		}
	`)
	if err != nil {
		t.Fatal(err)
	}

	// We create a "browser" to just a unit of our application
	tire := &Tire{Pos: 10, Size: "x"}
	manage := node.NewBrowser(m, manageTire(tire))

	// Example : test getting config
	actual, err := nodeutil.WriteJSON(manage.Root())
	if err != nil {
		t.Fatal(err)
	}
	expected := `{"pos":10,"size":"x","worn":true,"wear":0,"flat":false}`
	if actual != expected {
		t.Fatal(actual)
	}
}
```
[Full Source](https://github.com/freeconf/restconf/blob/master/example/site/unit_test.go)