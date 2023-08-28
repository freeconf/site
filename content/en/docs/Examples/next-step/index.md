---
title: Next Step
weight: 1
tags:
  - go
  - python
  - node
description: >
  After "Getting Started"
---

This document goes past [Getting Started]({{< ref "../../gettingstarted" >}}) by exploring YANG's fundamental management types:
* configuration 
* metrics 
* RPCs
* notifications

We'll demonstrate thru example by writing software that represents a car.  This car has configuration such as speed, metrics such as the odometer, rpcs such as start and stopping the car and events such as getting a flat tire. 

You can structure your code however you like, but here we are breaking this example into five separate files:
1. Car main source
2. Car unit test
3. YANG to describe management interface
4. Management source using FreeCONF
5. Management unit test

## 1. Car main source

This is just normal code to model the car. Notice there are no references to FreeCONF library, this is just code as you might normally write it.

{{< tabs name="car_src" >}}
{{% tab name="Go" %}}
file : `car.go`
```go
package car

import (
	"container/list"
	"fmt"
	"math/rand"
	"time"
)

// ////////////////////////
// C A R - example application
//
// This has nothing to do with FreeCONF, just an example application written in Go.
// that models a running car that can get flat tires when tires are worn.
type Car struct {
	Tire []*Tire

	Miles   float64
	Running bool

	// When the tires were last rotated
	LastRotation int64

	// Default speed value is in yang model file and free's your code
	// from hardcoded values, even if they are only default values
	// units milliseconds/mile
	Speed int

	// How fast to apply speed. Default it 1s or "miles per second"
	PollInterval time.Duration

	// Listeners are common on manageable code.  Having said that, listeners
	// remain relevant to your application.  The manage.go file is responsible
	// for bridging the conversion from application to management api.
	listeners *list.List
}

// CarListener for receiving car update events
type CarListener func(UpdateEvent)

// car event types
type UpdateEvent int

const (
	CarStarted UpdateEvent = iota + 1
	CarStopped
	FlatTire
)

func (e UpdateEvent) String() string {
	strs := []string{
		"unknown",
		"carStarted",
		"carStopped",
		"flatTire",
	}
	if int(e) < len(strs) {
		return strs[e]
	}
	return "invalid"
}

func New() *Car {
	c := &Car{
		listeners:    list.New(),
		Speed:        1000,
		PollInterval: time.Second,
	}
	c.NewTires()
	return c
}

// Stop will take up to poll_interval seconds to come to a stop
func (c *Car) Stop() {
	c.Running = false
}

func (c *Car) Start() {
	if c.Running {
		return
	}
	go func() {
		c.Running = true
		c.updateListeners(CarStarted)
		defer func() {
			c.Running = false
			c.updateListeners(CarStopped)
		}()
		for c.Speed > 0 {
			poll := time.NewTicker(c.PollInterval)
			for c.Running {
				for range poll.C {
					c.Miles += float64(c.Speed)
					for _, t := range c.Tire {
						t.endureMileage(c.Speed)
						if t.Flat {
							c.updateListeners(FlatTire)
							return
						}
					}
				}
			}
		}
	}()
}

// OnUpdate to listen for car events like start, stop and flat tire
func (c *Car) OnUpdate(l CarListener) Subscription {
	return NewSubscription(c.listeners, c.listeners.PushBack(l))
}

func (c *Car) NewTires() {
	c.Tire = make([]*Tire, 4)
	c.LastRotation = int64(c.Miles)
	for pos := 0; pos < len(c.Tire); pos++ {
		c.Tire[pos] = &Tire{
			Pos:  pos,
			Wear: 100,
			Size: "H15",
		}
	}
}

func (c *Car) ReplaceTires() {
	for _, t := range c.Tire {
		t.Replace()
	}
	c.LastRotation = int64(c.Miles)
}

func (c *Car) RotateTires() {
	x := c.Tire[0]
	c.Tire[0] = c.Tire[1]
	c.Tire[1] = c.Tire[2]
	c.Tire[2] = c.Tire[3]
	c.Tire[3] = x
	for i, t := range c.Tire {
		t.Pos = i
	}
	c.LastRotation = int64(c.Miles)
}

func (c *Car) updateListeners(e UpdateEvent) {
	fmt.Printf("car %s\n", e)
	i := c.listeners.Front()
	for i != nil {
		i.Value.(CarListener)(e)
		i = i.Next()
	}
}

// T I R E
type Tire struct {
	Pos  int
	Size string
	Flat bool
	Wear float64
	Worn bool
}

func (t *Tire) Replace() {
	t.Wear = 100
	t.Flat = false
	t.Worn = false
}

func (t *Tire) checkIfFlat() {
	if !t.Flat {
		// emulate that the more wear a tire has, the more likely it will
		// get a flat, but there is always a chance.
		t.Flat = (t.Wear - (rand.Float64() * 10)) < 0
	}
}

func (t *Tire) endureMileage(speed int) {
	// Wear down [0.0 - 0.5] of each tire proportionally to the tire position
	t.Wear -= (float64(speed) / 100) * float64(t.Pos) * rand.Float64()
	t.checkIfFlat()
	t.checkForWear()
}

func (t *Tire) checkForWear() bool {
	return t.Wear < 20
}

///////////////////////
// U T I L

// Subscription is handle into a list.List that when closed
// will automatically remove item from list.  Useful for maintaining
// a set of listeners that can easily remove themselves.
type Subscription interface {
	Close() error
}

// NewSubscription is used by subscription managers to give a token
// to caller the can close to unsubscribe to events
func NewSubscription(l *list.List, e *list.Element) Subscription {
	return &listSubscription{l, e}
}

type listSubscription struct {
	l *list.List
	e *list.Element
}

// Close will unsubscribe to events.
func (sub *listSubscription) Close() error {
	sub.l.Remove(sub.e)
	return nil
}

```
{{% /tab %}}
{{% tab name="Python" %}}
file : `car.py`
```python
import freeconf.nodeutil
import time
import threading
import random

# If these strings match YANG enum ids then they will be converted automatically.  Ints would
# work as well
EVENT_STARTED = "carStarted"
EVENT_STOPPED = "carStopped"
EVENT_FLAT_TIRE = "flatTire"


# Simple application, no connection to management 
# C A R
# Your application code.
#
# Notice there are no reference to FreeCONF in this file.  This means your
# code remains:
# - unit test-able
# - Not auto-generated from model files
# - free of annotations/tags
class Car():

    def __init__(self):
        
        # metrics/state
        self.miles = 0
        self.running = False
        self.thread = None

        # potential configuable fields
        self.speed = 1000
        self.new_tires()
        self.poll_interval = 1.0 #secs

        # Listeners are common on manageable code.  Having said that, listeners
        # remain relevant to your application.  The manage.go file is responsible
        # for bridging the conversion from application to management api.
        self.listeners = []

    def start(self):
        if self.running:
            return
        self.thread = threading.Thread(target=self.run, name="Car")
        self.thread.start()

    def reset(self):
        self.miles = 0

    def stop(self):
        """
          Will take up to poll_interval seconds to come to a stop
        """
        self.running = False

    def run(self):
        try:
            self.running = True
            self.update_listeners(EVENT_STARTED)
            while self.running:
                time.sleep(self.poll_interval)
                self.miles = self.miles + self.speed
                for t in self.tire:
                    t.endure_mileage(self.speed)
                    if t.flat:
                        self.update_listeners(EVENT_FLAT_TIRE)
                        return
        finally:
            self.running = False
            self.update_listeners(EVENT_STOPPED)

    def on_update(self, listener):
        """
        listen for car events like start, stop and flat tire
        """
        self.listeners.append(listener)
        def closer():
            self.listeners.remove(listener)
        return closer

    def update_listeners(self, event):
        print(f"car {event}")
        for l in self.listeners:
            l(event)    

    def new_tires(self):
        self.tire = []
        for pos in range(4):
            self.tire.append(Tire(pos))
        self.last_rotation = self.miles

    def replace_tires(self):
        for t in self.tire:
            t.replace()
        self.last_rotation = int(self.miles)
        self.start()

    def rotate_tires(self):
        first = self.tire[0]
        self.tire[0] = self.tire[1]
        self.tire[1] = self.tire[2]
        self.tire[2] = self.tire[3]
        self.tire[3] = first
        for i in range(len(self.tire)):
            self.tire[i].pos = i
        self.last_rotation = int(self.miles)


class Tire:
    def __init__(self, pos):
        self.pos = pos
        self.wear = 100
        self.size = "H15"
        self.flat = False
        self.worn = False

    def replace(self):
        self.wear = 100
        self.flat = False
        self.worn = False

    def check_if_flat(self):
        if not self.flat:
            # emulate that the more wear a tire has, the more likely it will
            # get a flat, but there is always a chance.
            self.flat = (self.wear - (random.random() * 10)) < 0

    def endure_mileage(self, speed):
        # Wear down [0.0 - 0.5] of each tire proportionally to the tire position
        self.wear -= (float(speed) / 100) * float(self.pos) * random.random()
        self.check_if_flat()
        self.check_for_wear()    

    def check_for_wear(self):
        self.wear < 20

```
{{% /tab %}}
{{< /tabs >}}


## 2. Car unit test

Shown here for completeness. 

{{< tabs name="car_test" >}}
{{% tab name="Go" %}}
file : `car_test.go`
```go
package car

import (
	"fmt"
	"testing"
	"time"

	"github.com/freeconf/yang/fc"
)

// Quick test of car's features using direct access to fields and methods
// again, nothing to do with FreeCONF.
func TestCar(t *testing.T) {
	c := New()
	c.PollInterval = time.Millisecond
	c.Speed = 1000

	events := make(chan UpdateEvent)
	unsub := c.OnUpdate(func(e UpdateEvent) {
		fmt.Printf("got event %s\n", e)
		events <- e
	})
	t.Log("waiting for car events...")
	c.Start()

	fc.AssertEqual(t, CarStarted, <-events)
	fc.AssertEqual(t, FlatTire, <-events)
	fc.AssertEqual(t, CarStopped, <-events)
	c.ReplaceTires()
	c.Start()

	fc.AssertEqual(t, CarStarted, <-events)
	unsub.Close()
	c.Stop()
}

```
{{% /tab %}}
{{% tab name="Python" %}}
file : `test_car.py`
```python
#!/usr/bin/env python3
import unittest 
import queue
import car

class TestCar(unittest.TestCase):

    # Quick test of car's features using direct access to fields and methods
    def test_server(self):
        c = car.Car()
        c.poll_interval = 0.01
        c.speed = 1000

        events = queue.Queue()
        def on_update(e):
            print(f"got event {e}")
            events.put(e)
        unsub = c.on_update(on_update)
        print("waiting for car events...")
        c.start()
        
        self.assertEqual(car.EVENT_STARTED, events.get())
        self.assertEqual(car.EVENT_FLAT_TIRE, events.get())
        self.assertEqual(car.EVENT_STOPPED, events.get())
        c.replace_tires()
        c.start()

        self.assertEqual(car.EVENT_STARTED, events.get())
        unsub()

        c.replace_tires()

        c.stop()

        
if __name__ == '__main__':
    unittest.main()

```
{{% /tab %}}
{{< /tabs >}}


## 3. YANG to describe management interface

Come up with YANG file for the desired management API for your car.  If you need help understanding YANG files there are a lot of references on the internet including this [YANG primer](../../reference/yang-primer).

file : `car.yang`
```yang
// Every yang file has a single module (or sub-module) definition.  The name of the module
// must match the name of the file. So module definition for "car" would be in "car.yang".
// Only exception to this rule is advanced naming schemes that introduce version into 
// file name.
module car {

	description "Car goes beep beep";
	
	revision 2023-03-27;  // date YYYY-MM-DD is typical but you can use any scheme 

	// globally unique id when used with module name should this YANG file mingles with other systems
	namespace "freeconf.org"; 

	prefix "car"; // used when importing definitions from other files which we don't use here

    // While any order is fine, it does control the order of data returned in the management
	// interface or the order configuration is applied. You can mix order of metrics and 
	// config, children, rpcs, notifications as you see fit

	
	// begin car root config...

	leaf speed {
		description "how many miles the car travels in one poll interval";
	    type int32;
		units milesPerSecond;
		default 1000;
	}

	leaf pollInterval {
		description "time between traveling ${speed} miles";
		type int32;
		units millisecs;
		default 1000;
	}

	// begin car root metrics...

	leaf running {
		description "state of the car moving or not";
		type boolean;
		config false;
	}

	leaf miles {
		description "odometer - how many miles has car moved";
		config false;
	    type decimal64 {
			fraction-digits 2;
		}
	}

	leaf lastRotation {
		description "the odometer reading of the last tire rotation";
		type int64;
		config false;
	}

	// begin children objects of car...

	list tire {
		description "rubber circular part that makes contact with road";
		
		// lists are most helpful when you identify a field or fields that uniquely identifies
		// the items in the list. This is not strictly neccessary.
		key pos;

        leaf pos {
			description "numerical positions of 0 thru 3";
            type int32;
        }

		// begin tire config...

        leaf size {
			description "informational information of the size of the tire";
            type string;
            default "H15";
        }

        // begin tire metrics

		leaf worn {
			description "a somewhat subjective but deterministic value of the amount of
			  wear on a tire indicating tire should be replaced soon";
            config false;
            type boolean;
        }

        leaf wear {
			description "number representing the amount of wear and tear on the tire. 
			  The more wear on a tire the more likely it is to go flat.";
            config false;			
            type decimal64 {
				fraction-digits 2;
			}
        }

        leaf flat {
			description "tire has a flat and car would be kept from running. Use
			   replace tire or tires to get car running again";
            config false;
            type boolean;
        }

		// begin tire RPCs...

		action replace {
			description "replace just this tire";

			// simple rpc with no input or output.

			// Side note: you could have designed this as an rpc at the root level that
			// accepts tire position as a single argument but putting it here makes it
			// more natural and simple to call.
		}
	}

	// In YANG 'rpc' and 'action' are identical but for historical reasons you must only 
	// use 'rpc' only when on the root and 'action' when inside a container or list. 

	// begin car RPCs...

	rpc reset {
		description "reset the odometer"; // somewhat unrealistic of a real car odometer
	}

    rpc rotateTires {
        description "rotate tires for optimal wear";
    }

    rpc replaceTires {
        description "replace all tires with fresh tires and no wear";
    }

	rpc start {
		description "start the car if it is not already started";
	}

	rpc stop {
		description "stop the car if it is not already stopped";
	}

	// begin of car events...

    notification update {
        description "important state information about your car";

		leaf event {
			type enumeration {
				enum carStarted {

					// optional. by default the values start at 0 and increment 1 past the 
					// previous value.  Numbered values may be even ever be used in your programs
					// and therefore irrelevant. Here I define a value just to demonstrate I could.
					value 1; 

				}
				enum carStopped;
				enum flatTire;				
			}
		}
    }
}
```

## 4. Management source using FreeCONF

Here we use FreeCONF's nodeutil.Node that uses reflection to implement the managment functions. 

This is far from the only way to implement this, you can [generate code](../codegeneration/) or implement your own logic that implements the `node.Node` interface.

{{< tabs name="manage_src" >}}
{{% tab name="Go" %}}
file : `manage.go`
```go
package car

import (
	"reflect"
	"time"

	"github.com/freeconf/yang/meta"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
)

// ///////////////////////
// C A R    M A N A G E M E N T
//
// Manage your car application using FreeCONF library according to the car.yang
// model file.
//
// Manage is root handler from car.yang. i.e. module car { ... }
func Manage(car *Car) node.Node {

	// We're letting reflect do a lot of the work when the yang file matches
	// the field names and methods in the objects.  But we extend reflection
	// to add as much custom behavior as we want
	return &nodeutil.Node{

		// Initial object. Note: as the tree is traversed, new Node instances
		// will have different values in their Object reference
		Object: car,

		// implement RPCs
		OnAction: func(n *nodeutil.Node, r node.ActionRequest) (node.Node, error) {
			switch r.Meta.Ident() {
			case "reset":
				// here we implement custom handler for action just as an example
				// If there was a Reset() method on car the default OnAction
				// handler would be able to call Reset like all the other functions
				car.Miles = 0
			default:
				// all the actions like start, stop and rotateTire are called
				// thru reflecton here because their method names align with
				// the YANG.
				return n.DoAction(r)
			}
			return nil, nil
		},

		// implement yang notifications (which are really just event streams)
		OnNotify: func(p *nodeutil.Node, r node.NotifyRequest) (node.NotifyCloser, error) {
			switch r.Meta.Ident() {
			case "update":
				// can use an adhoc struct send event
				sub := car.OnUpdate(func(e UpdateEvent) {
					msg := struct {
						Event int
					}{
						Event: int(e),
					}
					// events are nodes too
					r.Send(nodeutil.ReflectChild(&msg))
				})

				// we return a close **function**, we are not actually closing here
				return sub.Close, nil
			}

			// there is no default implementation at this time, all notification streams
			// require custom handlers.
			return p.Notify(r)
		},

		// implement fields that are not automatically handled by reflection.
		OnRead: func(p *nodeutil.Node, r meta.Definition, t reflect.Type, v reflect.Value) (reflect.Value, error) {
			if l, ok := r.(meta.Leafable); ok {
				// use "units" in yang to tell what to convert.
				//
				// Other useful ways to intercept custom reflection reads:
				// 1.) incoming reflect.Type t
				// 2.) field name used in yang or some pattern of the name (suffix, prefix, regex)
				// 3.) yang extension
				// 4.) any combination of these
				if l.Units() == "millisecs" {
					return reflect.ValueOf(v.Int() / int64(time.Millisecond)), nil
				}
			}
			return v, nil
		},

		// Generally the reverse of what is handled in OnRead
		OnWrite: func(p *nodeutil.Node, r meta.Definition, t reflect.Type, v reflect.Value) (reflect.Value, error) {
			if l, ok := r.(meta.Leafable); ok {
				if l.Units() == "millisecs" {
					d := time.Duration(v.Int()) * time.Millisecond
					return reflect.ValueOf(d), nil
				}
			}
			return v, nil
		},
	}
}

```
{{% /tab %}}
{{% tab name="Python" %}}
file : `manage.py`
```python
from freeconf import nodeutil, val

#
# C A R    M A N A G E M E N T
#  Bridge from model to car app.
#
# manage is root handler from car.yang. i.e. module car { ... }
def manage(c):

    # implement navigation by containers and lists defined in yang file
    def child(p, req):
        if req.meta.ident == 'tire':
            return manage_tires(c)
        return p.child(req)

    # implement RPCs (action or rpc)
    def action(p, req):
        if req.meta.ident == 'stop':
            c.stop()
        elif req.meta.ident == 'start':
            c.start()
        elif req.meta.ident == 'rotateTires':            
            c.rotate_tires()
        elif req.meta.ident == 'replaceTires':            
            c.replace_tires()
        elif req.meta.ident == 'reset':            
            c.reset()
        else:
            return p.action(req)
        return None
    
    # implement yang notifications which are really just events
    def notify(p, r):
        if r.meta.ident == 'update':
            def listener(event):
			    # events are nodes too
                r.send(nodeutil.Reflect({
                    "event": event
                }))
            closer = c.on_update(listener)
            return closer
        
        return p.notify(r)
    
    # implement fields that are not automatically handled by reflection.
    def field(p, r, w):
        if r.meta.ident == 'pollInterval':
            if r.write:
                c.poll_interval = float(w.v) / 1000 # ms to secs
            else:
                return val.Val(int(c.poll_interval * 1000)) # secs to ms
        else:
            return p.field(r, w)
        return None

	# Extend and Reflect form a powerful combination, we're letting reflect do a lot of the CRUD
	# when the yang file matches the field names.  But we extend reflection
	# to add as much custom behavior as we want
    return nodeutil.Extend(
        base = nodeutil.Reflect(c),
        on_action = action, on_notify=notify, on_child=child, on_field=field)


# manage_tires handles list of tires.
def manage_tires(c):
    def next(r):
        key = r.key
        found = None
        if key:
            # request for a specific tire by key (pos)
            pos = key[0].v
            if pos < len(c.tire):
                found = c.tire[pos]
        elif r.row < len(c.tire):
            # request for the nth tire in list
            found = c.tire[r.row]
            key = [ val.Val(r.row) ]

        if found:
            return manage_tire(found), key
    return nodeutil.Basic(on_next=next)

# manage_tire handles each tire node.  Everything *inside* list tire { ... }
def manage_tire(t):
    def action(p, r):
        if r.meta.ident == "replace":
            t.replace()
        return None
	# again, let reflection do a lot of the work with one extension to handle replace tire
    # action
    return nodeutil.Extend(
        base=nodeutil.Reflect(t),
        on_action=action
    )

```
{{% /tab %}}
{{< /tabs >}}

## 5. Management unit test

Write unit test for you management API without starting a server.  You can read or write configuration, call functions, listen to events or read metrics all from your unit test.

{{< tabs name="manage_test" >}}
{{% tab name="Go" %}}
file : `manage_test.go`
```go
package car

import (
	"testing"

	"github.com/freeconf/yang/fc"
	"github.com/freeconf/yang/node"
	"github.com/freeconf/yang/nodeutil"
	"github.com/freeconf/yang/parser"
	"github.com/freeconf/yang/source"
)

// Test the car management logic in manage.go
func TestManage(t *testing.T) {

	// setup
	ypath := source.Path("../yang")
	mod := parser.RequireModule(ypath, "car")
	app := New()

	// no web server needed, just your app and management function.
	brwsr := node.NewBrowser(mod, Manage(app))
	root := brwsr.Root()

	// read all config
	currCfg, err := nodeutil.WriteJSON(sel(root.Find("?content=config")))
	fc.AssertEqual(t, nil, err)
	expected := `{"speed":1000,"pollInterval":1000,"tire":[{"pos":0,"size":"H15"},{"pos":1,"size":"H15"},{"pos":2,"size":"H15"},{"pos":3,"size":"H15"}]}`
	fc.AssertEqual(t, expected, currCfg)

	// access car and verify w/API
	fc.AssertEqual(t, false, app.Running)

	// setup event listener, verify events later
	events := make(chan string)
	unsub, err := sel(root.Find("update")).Notifications(func(n node.Notification) {
		event, _ := nodeutil.WriteJSON(n.Event)
		events <- event
	})
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, 1, app.listeners.Len())

	// write config starts car
	err = root.UpdateFrom(nodeutil.ReadJSON(`{"speed":2000}`))
	fc.AssertEqual(t, nil, err)
	fc.AssertEqual(t, 2000, app.Speed)

	// start car
	fc.AssertEqual(t, nil, justErr(sel(root.Find("start")).Action(nil)))

	// should be first event
	fc.AssertEqual(t, `{"event":"carStarted"}`, <-events)
	fc.AssertEqual(t, true, app.Running)

	// unsubscribe
	unsub()
	fc.AssertEqual(t, 0, app.listeners.Len())

	// hit all the RPCs
	fc.AssertEqual(t, nil, justErr(sel(root.Find("rotateTires")).Action(nil)))
	fc.AssertEqual(t, nil, justErr(sel(root.Find("replaceTires")).Action(nil)))
	fc.AssertEqual(t, nil, justErr(sel(root.Find("reset")).Action(nil)))
	fc.AssertEqual(t, nil, justErr(sel(root.Find("tire=0/replace")).Action(nil)))
}

func sel(s *node.Selection, err error) *node.Selection {
	if err != nil {
		panic(err)
	}
	return s
}

func justErr(_ *node.Selection, err error) error {
	return err
}

```
{{% /tab %}}
{{% tab name="Python" %}}
file : `test_manage.py`
```python
#!/usr/bin/env python3
import unittest 
from freeconf import parser, node, source, nodeutil
import manage
import queue
import car
import time

# Test the car management API in manage.py
class TestManage(unittest.TestCase):

    def test_manage(self):
        # where YANG files are
        ypath = source.path("../yang")
        mod = parser.load_module_file(ypath, 'car')

        # create your empty app instance
        app = car.Car()

        # no web server needed, just your app instance and management entry point.
        brwsr = node.Browser(mod, manage.manage(app))

        # get a selection into the management API to begin interaction
        root = brwsr.root()

        # TEST CONFIG GET: read all the config
        curr_cfg = nodeutil.json_write_str(root.find("?content=config"))
        expected = '{"speed":1000,"pollInterval":1000,"tire":[{"pos":0,"size":"H15"},{"pos":1,"size":"H15"},{"pos":2,"size":"H15"},{"pos":3,"size":"H15"}]}'
        self.assertEqual(expected, curr_cfg)

        # verify car starts not running.  Here we are checking from the car instance itsel
        self.assertEqual(False, app.running)

        # SETUP EVENT TEST: here we add a listener to receive events and stick them in a thread-safe
        # queue for assertion checking later
        events = queue.Queue()
        def on_update(n):
            msg = nodeutil.json_write_str(n.event)
            events.put(msg)
        unsub = root.find("update").notification(on_update)

        # TEST CONFIG SET: write config
        root.update_from(nodeutil.json_read_str('{"speed":2000}'))
        self.assertEqual(2000, app.speed)

        # TEST RPC: start car
        root.find("start").action()

        # TEST EVENT: verify first event. will block until event comes in
        self.assertEqual('{"event":"carStarted"}', events.get())

        # TEST EVENT UNSUBSCRIBE: done listening for events
        unsub()

        # TEST RPCS: just hit all the RPCs for code coverage.  You could easily add the
        # underlying car object is changed accordingly
        root.find("rotateTires").action()
        root.find("replaceTires").action()
        root.find("reset").action()
        root.find("tire=0/replace").action()

if __name__ == '__main__':
    unittest.main()

```
{{% /tab %}}
{{< /tabs >}}


## What's Next?

* [YANG primer](../../reference/yang-primer) - to learn more about building your YANG
* [Code generation]() - To assist with reducing the code to write and improving the compile-time safety between code and YANG file
* More examples for building your management interface using [Basic](../node-basic/), [Extends](../node-extend/) and [Reflect](../node-reflect/)
