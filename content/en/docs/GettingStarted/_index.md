---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 2
description: >
  5 minute guide to using FreeCONF today.
---

{{% pageinfo color="primary" %}}
FreeCONF is available in Go, but Python will be available mid 2023 with other language support to follow.
{{% /pageinfo %}}

## Step 1.) Get the source

From an active Go project with `go` command in the `PATH` run this command

```bash
go get  github.com/freeconf/restconf
```

## Step 2.) Extract core YANG files

This is necesssary for some built in services that require YANG files

```bash
mkdir yang
go run github.com/freeconf/yang/cmd/fc-yang get -dir 'yang'
```

## Step 3.) Create a basic YANG file

Create file `./yang/hello.yang` to describe your application management.

```
module hello {
    leaf message {
        type string;
    }
}
```

## Step 4.) Create an application

There is a lot of flexibility on how you initialize RESTCONF, here is one way

```go
package main

import (
	"log"

	"github.com/freeconf/restconf"
	"github.com/freeconf/restconf/device"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/source"
)

// This is not specific to restconf, just example data structure for your
// application
type MyApp struct {
	Message string
}

func main() {
	// your app instance
	app := MyApp{}

	// where to find YANG files
	ypath := source.Dir("./yang")

	// organize modules.
	d := device.New(ypath)

	// register your application module. you can register as many as you want here.
	//   param 1 - name of the module, "hello.yang" must exist in ypath
	//   param 2 - code that connects (bridges) from your App to yang interface
	//             there are many options in nodeutil package to base your
	//             implementation on.  Here we use reflection because our yang file aligns
	//             with out application data structure.
	rootNode := nodeutil.Reflect{}.Object(&app)
	d.Add("hello", rootNode)

	// create the RESTCONF server
	restconf.NewServer(d)

	// this will apply configuration and startup web server
	if err := d.ApplyStartupConfigFile("./startup.json"); err != nil {
		log.Fatal(err)
	}

	select {}
}
```

## Step 5.) Create a startup config

Create this file with static configuration for your application and accompanying RESTCONF server

```json
{
    "fc-restconf" : {
        "web" : {
            "port": ":8080"
        }
    },
    "hello" : {
        "message" : "hello"
    }
}
```

## Step 6.) Startup your application

```bash
go run ./main.go
```

## Step 7.) Test your API

```bash
# read data
curl http://localhost:8080/restconf/data/hello:

# set data
curl -X PUT -d '{"message":"goodbye"}' http://localhost:8080/restconf/data/hello:

# read back new data
curl http://localhost:8080/restconf/data/hello:
```

## Done

That's it.  See [YANG Primer]({{< ref "../Reference/yang-primer" >}}) for more information on the YANG modeling langage and [Basic Components]({{< ref "../Reference/basic-components" >}}) on to integrate with your existing code base.
