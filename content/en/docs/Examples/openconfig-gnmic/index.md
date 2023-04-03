---
title: gNMIc 
weight: 21
tags:
  - gnmi
  - openconfig
  - client
description: >
  Using the gNMIc client to connect to your application  
---

gNMIc is a utility that can connect to any appliction that supports gNMI. Here are some working examples of using `gnmic` to connect to the car example application.

## Setup

1. [Install gNMIc]()
2. Run gNMI server
```bash
cd ../gnmi-server
go run .
```

## Get

file: `get.yml`
```yaml
address: localhost:8090
insecure: true
get-path: "car:"

```

Command: `gnmic --config get.yml get`
```json
[
  {
    "source": "localhost:8090",
    "timestamp": 1680355915557520285,
    "time": "2023-04-01T09:31:55.557520285-04:00",
    "updates": [
      {
        "Path": "car:",
        "values": {
          "": {
            "lastRotation": 0,
            "miles": 1160,
            "running": false,
            "speed": 10,
            "tire": [
              {
                "flat": false,
                "pos": 0,
                "size": "15",
                "wear": 100,
                "worn": false
              },
              {
                "flat": false,
                "pos": 1,
                "size": "15",
                "wear": 73.34043889415778,
                "worn": false
              },
              {
                "flat": false,
                "pos": 2,
                "size": "15",
                "wear": 40.16247998820154,
                "worn": false
              },
              {
                "flat": true,
                "pos": 3,
                "size": "15",
                "wear": 6.759124269512249,
                "worn": false
              }
            ]
          }
        }
      }
    ]
  }
]
```
