---
title: RESTCONF server
weight: 10
tags:
  - go
  - restconf
  - server
description: >
   Minimal code to add RESTCONF support to your application
---

After you captured your management into a root Node, you'll likely want to serve that mangement capability thru a network aware interface.

Here we add support for RESTCONF to our car application in the `main()` method. There is essentially these steps :
1. Identify the path or paths to your YANG files
2. Create a `Device` object
3. Add the management browser to you application
4. Create a RESTCONF server
5. Apply a startup config

[Full Source]()

```go
package main

import (
	"flag"
	"log"
	"strings"

	"github.com/freeconf/examples/car"
	"github.com/freeconf/restconf"
	"github.com/freeconf/restconf/device"
	"github.com/freeconf/yang/source"
)

// Connect everything together into a server to start up
func main() {
	flag.Parse()

	// Your app here
	app := car.New()

	// where the yang files are stored
	ypath := source.Path("../yang")

	// Device is just a container for browsers.  Needs to know where YANG files are stored
	d := device.New(ypath)

	// Device can hold multiple modules, here we are only adding one
	if err := d.Add("car", car.Manage(app)); err != nil {
		panic(err)
	}

	// Select wire-protocol RESTCONF to serve the device.
	restconf.NewServer(d)

	// apply start-up config normally stored in a config file on disk
	config := `{
		"fc-restconf":{
			"web":{
				"port":":8090"
			}
		},
        "car":{"speed":10}
	}`
	if err := d.ApplyStartupConfig(strings.NewReader(config)); err != nil {
		panic(err)
	}

	if !*testMode {
		// wait for ctrl-c
		log.Printf("server started")
		select {}
	}
}

var testMode = flag.Bool("test", false, "do not run in background (i.e. driven by unit test)")

```
