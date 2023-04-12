---
title: Slack
weight: 27
description: >
  Send notifications to Slack
---

![Car Updates](images/screenshot.png)

## Demonstrates 

* How to send YANG notifications to Slack

## Details

The `fcslack.Service` will selectively subscribe to a list of paths in the local application and send the event payload to a designated channel in Slack.  

![architecture Car Tire Wear](images/architecture.png)

If any of the paths are invalid, you will get an error. This is good.  Unlike scraping logs, if you don't have your regex correct, you silently miss the event.

This example can be easily extended to send events to Kafka, AWS SQS, Pager Duty, etc.

## Running the example

{{< readfile file="/content/en/docs/Examples/common/get-example-source.md" >}}

### Setup Slack workspace, application and bot token

You will have to setup a Slack Application, register the application with your Slack workspace and generate a bot token. 

{{< youtube id="n-7l-N541u0" title="Creating a bot token" >}}

## Create an environment file with your settings

file : `fcslack/cmd/env.sh`
```bash

# Should start with xoxb- which means it is a "Bot token"
export SLACK_API_TOKEN=xoxb-5555555555555-5555555555555-5xyZ5xyZ5xyZ5xyZ5xyZ5xyZ

# Any channel you want, you can control each message if you want in your application
export SLACK_CHANNEL=testing-integration

```

## Run car application with your settings

```bash
cd fcslack/cmd
source ./env.sh
go run .
```

You should see the `{"event":"carStarted"}` event right away in your slack channel, then in minute or so see the `{"event":"flatTire"}` and the `{"event":"carStopped"}` events.

## Conclusion

FreeCONF is a library to build your own solutions.  This code can work with any models so once you build it once you can use it everywhere.  Also, you can have multiple systems subscribing to the same event streams so maybe you start with slack and then weave in PagerDuty integration once you're confident this is a reliable event stream.

### Using this example code

If you wanted to use the approach here as is, you could import `fcslack` into your application directly by calling `go get github.com/freeconf/example` but this example code may change without notice.  It's intention is to give you a starter project to customize as needed.

### Ideas of Expanding

* Have a yang extension that will require fcslack to subscribe to ensure that event is always sent.


### main.go

```go
package main

import (
	"flag"
	"fmt"
	"log"
	"os"
	"strings"

	"github.com/freeconf/examples/car"
	"github.com/freeconf/restconf"

	"github.com/freeconf/restconf/device"

	"github.com/freeconf/yang/source"

	"github.com/freeconf/examples/fcslack"
)

func main() {
	flag.Parse()

	// Your app here
	app := car.New()

	// where the yang files are stored
	ypath := source.Path("../../yang:../../car:..")

	// Device is just a container for browsers.  Needs to know where YANG files are stored
	d := device.New(ypath)

	// Device can hold multiple modules, here we are only adding one
	if err := d.Add("car", car.Manage(app)); err != nil {
		panic(err)
	}

	s := fcslack.NewService(d)

	if err := d.Add("fc-slack", fcslack.Manage(s)); err != nil {
		panic(err)
	}

	restconf.NewServer(d)

	// apply start-up config normally stored in a config file on disk
	config := fmt.Sprintf(`{
		"fc-restconf":{
			"debug": true,
			"web":{
				"port":":8090"
			}
		},
		"fc-slack" : {
			"client" : {
				"apiToken": "%s"
			},
			"subscription": [
				{
					"channel" : "%s",
					"module": "car",
					"path": "update"
				}
			]
		},
        "car":{"speed":100}
	}`, os.Getenv("SLACK_API_TOKEN"), os.Getenv("SLACK_CHANNEL"))

	// bootstrap config for all local modules
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