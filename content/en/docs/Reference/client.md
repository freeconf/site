---
title: Client
weight: 102
description: >
  Connecting to a RESTCONF server
---

While you can easily interact with a RESTCONF server using any HTTP library, using a RESTCONF client can have advantages.  In particular it will let you walk a server and access the meta in the YANG alongside the data. FreeCONF gives you a client API that is similar to server API once you are connected.

## Example

```go
import (
	"github.com/freeconf/restconf/client"
	"github.com/freeconf/restconf/device"
	"github.com/freeconf/yang/source"
)

...

	// YANG: just need YANG file ietf-yang-library.yang, not the yang of remote system as that will
	// be downloaded as needed
	ypath := source.Path(pathToYangFiles)

	// Connect
	proto := client.ProtocolHandler(ypath)
	dev, err := proto("http://localhost:9999/restconf")

	// Get a browser to walk server's management API for car
	car, err := dev.Browser("car")

	// Example of config: I feel the need, the need for speed
	// bad config is rejected in client before it is sent to server
	err = car.Root().UpsertFrom(nodeutil.ReadJSON(`{"speed":100}`)).LastErr

	// Example of metrics: Get all metrics as JSON
	metrics, err := nodeutil.WriteJSON(car.Root().Find("?content=nonconfig"))

	// Example of RPC: Reset odometer
	err = car.Root().Find("reset").Action(nil).LastErr

	// Example of notification: Car has an important update
	unsub, err := car.Root().Find("update").Notifications(func(n node.Notification) {
		msg, err := nodeutil.WriteJSON(n.Event)
		if err != nil {
			panic(err)
		}
		fmt.Println(msg)
	})
	defer unsub()

	// Example of multiple modules: This is the FreeCONF server module
	rcServer, err := dev.Browser("fc-restconf")

	// Example of config: Enable debug logging on FreeCONF's remote RESTCONF server
	// bad config is rejected in client before it is sent to server
	err = rcServer.Root().UpsertFrom(nodeutil.ReadJSON(`{"debug":true}`)).LastErr

...    
```

[Full Source](https://github.com/freeconf/restconf/blob/master/example/site/client_test.go)