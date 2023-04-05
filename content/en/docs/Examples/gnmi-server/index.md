---
title: gNMI server
tags:
  - openconfig
  - gnmi
  - server
  - go
weight: 20
description: >
  Adding gNMI server-side support to your application
---

[gNMI](https://www.openconfig.net/docs/gnmi/gnmi-specification/) is a alternative communication protocol to RESTCONF by openconfig.net](https://www.openconfig.net). You might use gNMI over RESTCONF because the services you want to use work with gNMI. With FreeCONF you can easily enable RESTCONF, gNMI or both at the same time.

## How to add gNMI server support to your application

file: `main.go`
```go
package main

import (
	"flag"
	"log"
	"strings"

	"github.com/freeconf/examples/car"
	"github.com/freeconf/gnmi"
	"github.com/freeconf/restconf/device"
	"github.com/freeconf/yang/source"
)

// Connect everything together into a server to start up
func main() {
	flag.Parse()

	// Your app here
	app := car.New()

	// where the yang files are stored
	ypath := source.Path("../yang:../car")

	// Device is just a container for browsers.  Needs to know where YANG files are stored
	d := device.New(ypath)

	// Device can hold multiple modules, here we are only adding one
	if err := d.Add("car", car.Manage(app)); err != nil {
		panic(err)
	}

	// Select wire-protocol gNMI to serve the device
	gnmi.NewServer(d)

	// apply start-up config normally stored in a config file on disk
	config := `{
		"fc-gnmi":{
			"debug": true,
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

## To run this example

```bash
git clone https://github.com/freeconf/examples fc-examples
cd ./gnmi-server
go run .
```

Checkout [FreeCONF gNMIc examples]({{< relref "../openconfig-gnmic" >}}) for interacting with this running service.

## To add gNMI support to your Go application

Just add the Go dependency and setup for `gnmi.NewServer` where ever makes sense to your application.

```bash
go get github.com/freeconf/gnmi
```
