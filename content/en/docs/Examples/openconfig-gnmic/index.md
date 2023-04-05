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

[gNMIc](https://gnmic.openconfig.net/) is a utility that can connect to any appliction that supports gNMI protocol including applications [using the FreeCONF library]({{< relref "../gnmi-server" >}}). You can use `gnmic` from the terminal or use it to [send metrics to InfluxDB, Prometheus, Kafka, NATS](https://gnmic.openconfig.net/user_guide/outputs/output_intro/) and possibly others.

Here are some working examples of using `gnmic` from terminal to connect to the car example application.

## FreeCONF examples source code

Get the FreeCONF example code

```bash
git clone https://github.com/freeconf/examples fc-examples
```

## Setup

1. [Install the gNMIc](https://gnmic.openconfig.net/#installation) client executable
2. Run gNMI server with car application registered with server
```bash
cd ./gnmi-server
go run .
```

## Using gnmic

In a separate terminal, go to the directory containing gnmic commands.

```bash
cd openconfig-gnmi
```

## Example commands

```sh
#!/usr/bin/env bash

set -euf -o pipefail

# Get Data Object
gnmic --config car.yml get --path car:/

# Get Data Value
gnmic --config car.yml get --path car:/speed

# Get Data : use model.
gnmic --config car.yml get --model car --path /


# Set Data Object
gnmic --config car.yml set --update  car:/:::json:::'{"speed":300}'

# Set Data Value
gnmic --config car.yml set --update-path  car:/speed --update-value 300


# Subscribe
timeout 5s \
  gnmic --config car.yml sub --model car --path / --sample-interval 1s --heartbeat-interval 2s || true

# Subscribe to just tire metrics : use model
timeout 5s \
  gnmic --config car.yml sub --mode once --model car --path /tire || true

# Subscribe to just tire metrics
timeout 5s \
  gnmic --config car.yml sub --mode once --path car:/tire || true

# Subscribe to just tire 1 metrics
timeout 5s \
  gnmic --config car.yml sub --mode once --path car:/tire=1 || true
```

## Example Output

Command: `gnmic --config car.yml --model car --path /`
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
