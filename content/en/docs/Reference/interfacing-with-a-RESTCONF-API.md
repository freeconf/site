---
title: Using RESTCONF
weight: 3
description: >
  Interfacing with a RESTCONF API as a tool, CLI or script
---
This is about consuming a RESTCONF API, not building one.  In short, if you know REST then you know RESTCONF.  There are some really cool features however that will find useful and a few conventions that would be good to learn.

## Adjustable data granularity

Traditional REST APIs struggle with what level of information to return for each GET request and this is called [granularity](https://dzone.com/articles/restful-api-design-principle-deciding-levels-of-gr).  Return too little data and scripts will need to make constant trips for more.  Return too much data and majority of it will likely be unused.  Both situations cause delays and additional resource consumption. 

RESTCONF APIs do not have this issue.  Clients can precisely control the data they want from depth, to list pagination. From selecting fields to excluding fields.

With APIs developed with FreeCONF, each implemention has the control to only read the data that was requested. That is, granularity is not implemented as a reponse filters like GraphQL after the data has already been extracted.

## Methods

No surprises here:

- `GET` - Getting configuration, metrics and notifications. Notificatons are in SSE format detailed later in this document.
- `PATCH` - Updating configuration.  If objects are not found, they will be created so this is really an upsert.
- `PUT` - Replacing configuration.  If objects are found, they will removed first then recreated with the given configuration.
- `POST` - Creating read-write data (e.g. config). If objects are not found, **you will** get an error.
- `DELETE` - Deleting read-write data.  If objects are not found, **you will not** get an error.
- `OPTIONS` - Useful to test if user has access to certain data path

## URL

Basic for is this : `/restconf/data/{module}:{path}[?params]` 

- `module` - name of the yang file So `car.yang` would be served at `/restconf/data/car:`
- `path` - drill down into the objects.  So to access the car tires, would be `/restconf/data/car:tires`.  Only tricky part is drilling into lists.  If you wanted to drill into front-left tire, you might use `/restconf/data/car:tires=front-left`. Drilling deeper still might be  `/restconf/data/car:tires=front-left/vendor`
- `?params` - For `GET` methods only, many params will help you limit the amount of data you return to save bandwidth and compute resources. More detailed information can be found in the [specification](https://datatracker.ietf.org/doc/html/rfc8040#section-4.8) but here is a quick summary:
  * `depth=N` - limits the level of data returns to N levels in the hierarchy
  * `content=config` - return only configuration data
  * `content=nonconfig` - return only metric data
  * `with-defaults=trim` - Do not return leaf values if they match the default value.  This is useful for determining what a configuration user may have actually changed versus what configuration a device is actually using.
  * `fields=a;b/c` - returns only select data paths.  Note: you'll need to encode parameters depending on your http client libraries.  For example this would be `fields=a%3db/c`

### Custom URL params

FreeCONF adds a few extra, useful parameters when retrieving data

  * `fc.xfields=a;b/c` - inverse of fields in that it returns all fields except specified.  Again watch the encoding of the `;` as detailed above.
  * `fc.range=b/c!N-M` - returns rows N thru M inclusive in list b/c
  * `fc.max-node-count=N` - increases or decreases the maximum allowed data to be returned.  There are limits by default to ensure unbounded requests do not bog down system. 

# Examples

See [specification](https://datatracker.ietf.org/doc/html/rfc8040#section-4) for more details on how RESTCONF maps to REST.

|Task | Method | Path | Description  |
|----|--------|------|--------------|
| Read | GET | `/restconf/data/car:` | Get's all data (configuration and metrics) for car module |
| Read | GET | `/restconf/data/car:tire` | Get's all data for all tires |
| Read | GET | `/restconf/data/car:tire=1` | Get's all data for first car tire. Yes, seeing an equals in a URL can be disconcerting, but it is legal. |
| Update | PATCH | `/restconf/data/car:cruise`<br>*body:*`{"desiredSpeed":65}` | Set cruise control desired speed |
| Read | GET | `/restconf/data/car:tire?c2-range=!1-2` | Get's all data for car tires 1 and 2 |
| Read | GET | `/restconf/data/car:tire?fields=wear%3did` | Get's only tire id and wear level for all tires. `%3d` is encoded `=`. |
| Read | GET | `/restconf/data/car:tire?content=config&with-defaults=trim` | Get's only configuration that is changed from the default for all tires |
| Create | POST | `/restconf/data/car:navigate`<br>*body:*`{"destination":{"address":"10 Main st."}}` | Add a new destination address to navigation.  This would only work if no naviation address was already set. |
| Delete | DELETE | `/restconf/data/car:navigate/destination` | Remove destination from navigation system.  |
| RPC | POST | `/restconf/data/car:rotateTires` | Run a RPC to rotate the tires |
| RPC | POST | `/restconf/data/car:rotateTires`<br>*body:*`{"order":"clockwise"}` | Run a RPC to rotate the tires in specific order |
| RPC | POST | `/restconf/data/car:rotateTires`<br>*body:*`{"order":"clockwise"}`<br>*response:*`{"charge":30.00}` | Run a RPC to rotate the tires in specific order and return the cost. |
| Event Stream | GET | `/restconf/data/car:`<br>*response:*<br>`{"status":"running"}`<br><br>`{"status":"stopped","reason":"flat"}` | Stream of events regarding the car status |
| Event Stream | GET | `/restconf/data/car:?filter=status%3dstopped`<br>*response:*<br>`{"status":"stopped","reason":"flat"}` | Stream only events that cause car to stop. `%3d` is encoded `=`. |

## Events

RESTCONF delivers events using [SSE(Server State Events)](https://html.spec.whatwg.org/multipage/server-sent-events.html#server-sent-events) over HTTP.  This is simply a stream per event stream.  HTTP/2 allows for an unlimited number of streams over a single connection. Each event is serialized JSON followed by 2 end-of-line characters so you know the event message boundaries.

There is no special library required to read these messages and you can subscribe to as many event streams as you want w/o opening a new connection.

## Subscribing to events in web browser:

```javascript
// this looks like a new connection, but HTTP/2 sends it over existing connection
// to unsubscribe, call  events.close();
const events = new EventSource("/restconf/data/car:updates");
events.addEventListener("message", (e) => {
   console.log(e.data);
 });
```

### Subscribing to events in CLI

```bash
$ curl https://server/restconf/data/car:updates
data: {"tire":{"wear":80}}

data: {"tire":{"wear":70}}
```

### Examples subscription paths:

| Path | Description |
|--|--|
| `updates` | Any changes to car |
| `updates?filter=tire/wear<20` | Any changes to car when the tire wear is less than 20 |

## More

When using REST API to build a web interface, checkout [model assisted web UI.]({{< relref "../examples/webui" >}})
