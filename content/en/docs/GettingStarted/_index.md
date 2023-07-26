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
import freeconf.restconf
import freeconf.source
import freeconf.device
import freeconf.parser
import freeconf.nodeutil.reflect
from threading import Event

class MyApp:
    def __init__(self):
        self.message = None

app = MyApp()

# specify all the places you store YANG files
ypath = freeconf.source.any(
    freeconf.source.path("."),
    freeconf.source.restconf_internal_ypath()
)

# device hosts one or more management "modules" into a single instance for export
dev = freeconf.device.Device(ypath)

# load and validate our YANG file
mod = freeconf.parser.load_module_file(ypath, "hello")

# your management implementation to your app instance. there are endless ways to
# to build your management interface from code generation, to reflection and any
# combination there of.  Here we use reflection for our simple app
mgmt = freeconf.nodeutil.reflect.Reflect(app)

# device can host multiple modules, here we just register our one app
b = freeconf.node.Browser(mod, mgmt)
dev.add_browser(b)

# select RESTCONF as management protocol. gNMI is option as well
s = freeconf.restconf.Server(dev)

# this will apply configuration including starting RESTCONF web server
dev.apply_startup_config("./startup.json")

# simple python trick to wait until shutdown
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
