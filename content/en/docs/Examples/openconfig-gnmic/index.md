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

file: `car.yml`
```yaml
address: localhost:8090
insecure: true


```

Command: `gnmic --config car.yml --model car --path ""`
```json
[
  {
    "source": "localhost:8090",
    "timestamp": 1680564190640829839,
    "time": "2023-04-03T19:23:10.640829839-04:00",
    "updates": [
      {
        "Path": "",
        "values": {
          "": {
            "lastRotation": 0,
            "miles": 4100,
            "running": true,
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
                "wear": 79.28663660297573,
                "worn": false
              },
              {
                "flat": false,
                "pos": 2,
                "size": "15",
                "wear": 59.94265362733487,
                "worn": false
              },
              {
                "flat": false,
                "pos": 3,
                "size": "15",
                "wear": 38.2541625477213,
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

```sh
#!/usr/bin/env bash

set -euf -o pipefail

# Get Data : full path
gnmic --config car.yml get --path car:/

# Get Data : use model
gnmic --config car.yml get --model car --path ""

# Set Data
gnmic --config car.yml set --update  car:/:::json:::'{"speed":300}'

# Subscribe
timeout 5s \
  gnmic --config car.yml sub --model car --path "" --sample-interval 1s --heartbeat-interval 2s || true

# Subscribe to just tire metrics : use model
timeout 5s \
  gnmic --config car.yml sub --mode once --model car --path "/tire" || true

# Subscribe to just tire metrics : full path
timeout 5s \
  gnmic --config car.yml sub --mode once --path "car:/tire" || true

```
