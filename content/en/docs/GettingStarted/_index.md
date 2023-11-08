---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 2
tags:
  - server
  - restconf
  - node
  - go
description: >
  5 minute guide to using FreeCONF today.
---


## Step 1.) Get the source

{{< tabs name="install" >}}
{{% tab name="Go" %}}

From an active Go project with `go` command in the `PATH` run this command

```bash
go get github.com/freeconf/restconf
```
{{% /tab %}}
{{% tab name="Python" %}}

Install latest Python package.  

```bash
pip install https://github.com/freeconf/lang/releases/download/v0.1.0-alpha/freeconf-0.1.0-py3-none-any.whl
fc-lang-install -v
```

The `fc-lang-install` script should not need root and simply downloads a single file containing FreeCONF core to `~/.freeconf/bin`.

{{% /tab %}}
{{< /tabs >}}

## Step 2.) Create a basic YANG file

Create file `./hello.yang` to describe your application management.

```
module hello {
	leaf message {
		type string;
	}
}
```

## Step 3.) Create an application

There is a lot of flexibility on how you initialize RESTCONF, here is one way.

{{< tabs name="app_src" >}}
{{% tab name="Go" %}}

Create file `main.go`

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
	ypath := source.Any(source.Path("."), restconf.InternalYPath)

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

	// select RESTCONF as management protocol. gNMI is option as well
	restconf.NewServer(d)

	// this will apply configuration and starting RESTCONF web server
	if err := d.ApplyStartupConfigFile("./startup.json"); err != nil {
		log.Fatal(err)
	}

	select {}
}
```
{{% /tab %}}

{{% tab name="Python" %}}

Create file `getting-started.py`

```python
from freeconf import restconf, source, device, parser, node, source, nodeutil
from threading import Event

# Represents a basic python application. There are no requirements imposed
# by FreeCONF on how you develop your application
class MyApp:
    def __init__(self):
        self.message = None

app = MyApp()

# The remaining is FreeCONF specific and shows how to build a management interface
# to you python application

# specify all the places where you store YANG files
ypath = source.any(
    source.path("."),
    source.restconf_internal_ypath()
)

# load and validate your YANG file
mod = parser.load_module_file(ypath, "hello")

# device hosts one or more management "modules" into a single instance that you
# want to export in management interface
dev = device.Device(ypath)

# connect your application to your management implementation.
# there are endless ways to to build your management interface from code generation,
# to reflection and any combination there of.  A lot more information in docs.
mgmt = nodeutil.Node(app)

# connect parsed YANG to your management implementation.  Browser is a powerful way
# to dynamically control your application that is useful in unit tests or other contexts
# but here we construct it to serve our API 
b = node.Browser(mod, mgmt)

# register your app management browser in device.  Device can hold any number of browser
# objects.
dev.add_browser(b)

# select RESTCONF as management protocol. gNMI is option as well or any custom or 
# future protocol
s = restconf.Server(dev)

# this will apply configuration including starting RESTCONF web server
dev.apply_startup_config_file("./startup.json")

# simple python trick to wait until ctrl-c shutdown
Event().wait()
```

{{% /tab %}}
{{< /tabs >}}


## Step 4.) Create a startup config

Create this file with static configuration for your application and accompanying RESTCONF server in the file `startup.json`

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

## Step 5.) Startup your application

{{< tabs name="run" >}}
{{% tab name="Go" %}}

```bash
go run ./main.go
```
{{% /tab %}}

{{% tab name="Python" %}}

```bash
python3 ./getting-started.py
```

{{% /tab %}}
{{< /tabs >}}

## Step 6.) Test your API

```bash
# read data
curl http://localhost:8080/restconf/data/hello:

# set data
curl -X PATCH -d '{"message":"goodbye"}' http://localhost:8080/restconf/data/hello:

# read back new data
curl http://localhost:8080/restconf/data/hello:
```

## Done

That's it.  See [YANG Primer]({{< ref "../Reference/yang-primer" >}}) for more information on the YANG modeling langage and [Basic Components]({{< ref "../Reference/basic-components" >}}) on to integrate with your existing code base.
