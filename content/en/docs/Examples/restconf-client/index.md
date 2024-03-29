---
title: RESTCONF client
weight: 11
tags:
  - client
  - go
  - restconf
  - json
description: >
  Connecting to a RESTCONF server
---

You can communicate with a RESTCONF server using a basic HTTP library.  For simple applications this is a reasonable approach.   If you want to "level-up" your API access to be able to walk the schema (i.e. YANG) then you can use the RESTCONF client in FreeCONF.

```go
package demo

import (
	"fmt"
	"strings"
	"testing"

	"github.com/freeconf/examples/car"
	"github.com/freeconf/restconf"
	"github.com/freeconf/restconf/client"
	"github.com/freeconf/restconf/device"
	"github.com/freeconf/yang/fc"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/source"
)

func connectClient() {

	// YANG: just need YANG file ietf-yang-library.yang, not the yang of remote system as that will
	// be downloaded as needed
	ypath := restconf.InternalYPath

	// Connect
	proto := client.ProtocolHandler(ypath)
	dev, err := proto("http://localhost:9998/restconf")
	if err != nil {
		panic(err)
	}

	// Get a browser to walk server's management API for car
	car, err := dev.Browser("car")
	if err != nil {
		panic(err)
	}
	root := car.Root()
	defer root.Release()

	// Example of config: I feel the need, the need for speed
	// bad config is rejected in client before it is sent to server
	n, err := nodeutil.ReadJSON(`{"speed":100}`)
	if err != nil {
		panic(err)
	}
	err = root.UpsertFrom(n)
	if err != nil {
		panic(err)
	}

	// Example of metrics: Get all metrics as JSON
	sel, err := root.Find("?content=nonconfig")
	if err != nil {
		panic(err)
	}
	defer sel.Release()
	metrics, err := nodeutil.WriteJSON(sel)
	if err != nil {
		panic(err)
	}
	if metrics == "" {
		panic("no metrics")
	}

	// Example of RPC: Reset odometer
	sel, err = root.Find("reset")
	if err != nil {
		panic(err)
	}
	defer sel.Release()
	if _, err = sel.Action(nil); err != nil {
		panic(err)
	}

	// Example of notification: Car has an important update
	sel, err = root.Find("update")
	if err != nil {
		panic(err)
	}
	defer sel.Release()
	unsub, err := sel.Notifications(func(n node.Notification) {
		msg, err := nodeutil.WriteJSON(n.Event)
		if err != nil {
			panic(err)
		}
		fmt.Println(msg)
	})
	if err != nil {
		panic(err)
	}
	defer unsub()

	// Example of multiple modules: This is the FreeCONF server module
	rcServer, err := dev.Browser("fc-restconf")
	if err != nil {
		panic(err)
	}

	// Example of config: Enable debug logging on FreeCONF's remote RESTCONF server
	serverSel := rcServer.Root()
	defer serverSel.Release()
	n, err = nodeutil.ReadJSON(`{"debug":true}`)
	if err != nil {
		panic(err)
	}
	serverSel.UpsertFrom(n)
	if err != nil {
		panic(err)
	}
}

func TestClient(t *testing.T) {

	// setup -  start a server
	ypath := source.Path("../yang")
	serverYPath := source.Any(ypath, restconf.InternalYPath)
	carServer := car.New()
	local := device.New(serverYPath)
	local.Add("car", car.Manage(carServer))
	s := restconf.NewServer(local)
	defer s.Close()
	cfg := `{
		"fc-restconf": {
			"debug": true,
			"web" : {
				"port": ":9998"
			}
		},
		"car" : {
			"speed": 5
		}
	}`
	fc.RequireEqual(t, nil, local.ApplyStartupConfig(strings.NewReader(cfg)))

	connectClient()
}

```

