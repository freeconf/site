---
title: Ansible
weight: 25
tags:
  - server
  - ansible
  - config
  - integration
description: >
  Configure any RESTCONF client from ansible
---

Ansible can be used to get, set and call functions in RESTCONF.  You can use the [Ansible RESTCONF module](https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/restconf_config_module.html) or you can use the [Ansible uri module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/uri_module.html) to make RESTCONF calls as I've done here.

## Setup

{{< readfile file="/content/en/docs/Examples/common/get-example-source.md" >}}

### Setup ansible
````
cd ansible
python3 -m venv venv
source ./venv/bin/activate
python -m pip install -r requirements.txt
````

file: `requirements.txt`
```
ansible==7.0.0
ansible-core==2.15.8

```

## Running

### Inventory file

```yaml
all:
  hosts:
    car:
      http_port: 8090
      ansible_host: localhost

```

### Get Config

Command: `ansible-playbook -i hosts.yml get-config.yml`

File: `get-config.yml`

```yaml
---
- name: get configure
  hosts: car
  connection: httpapi
  gather_facts: False

  tasks:
  - name: get current speed
    uri:
      url: "http://{{ ansible_host }}:{{ http_port }}/restconf/data/car:?depth=1"
    register: results

  - debug:
      var: results.json
```

Example Output:
```bash
****
TASK [get current speed] *******************************************************
ok: [car]

TASK [debug] *******************************************************************
ok: [car] => {
    "results.json": {
        "lastRotation": 13100,
        "miles": 20900,
        "running": false,
        "speed": 300
    }
}

PLAY RECAP *********************************************************************
car                        : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

### Set Config 

Command: `ansible-playbook -i hosts.yml set-config.yml`

File: `set-config.yml`

```yaml
---
- name: configure
  hosts: car
  connection: httpapi
  gather_facts: False

  tasks:
  - name: change speed
    uri:
      url: "http://{{ ansible_host }}:{{ http_port }}/restconf/data/car:"
      method: PATCH
      body_format: json
      body: |
        {
            "speed":10
        }

```

### Run RPCs

Here we run the `replaceTires` RPC

```yaml
---
- name: rpc
  hosts: car
  connection: httpapi
  gather_facts: False

  tasks:
  - name: replace tires
    uri:
      url: "http://{{ ansible_host }}:{{ http_port }}/restconf/data/car:replaceTires"
      method: POST

```

