---
title: "Resources"
weight: 20
description: >
  Links of useful resources on the Internet
---

FreeCONF plays an important part of a larger community of people bringing together specifications, information, products and projects.  If you would like to submit a link, please [submit a merge request](https://github.com/freeconf/site).

## FreeCONF
* [YANG parser source](https://github.com/freeconf/yang)
* [RESTCONF support source](https://github.com/freeconf/restconf)
* [gNMI support source](https://github.com/freeconf/gnmi)
* [Examples source](https://github.com/freeconf/examples)
* [freeconf.org site repo](https://github.com/freeconf/gnmi)

## Information/Specifications
* [YANG/RESTCONF](https://en.wikipedia.org/wiki/YANG) on wikipedia
* [RFCs]({{< relref "compliance/rfcs" >}})
* [Network Programmability with YANG](https://a.co/d/bs8uOut) - Book by [Benoît Claise](https://www.claise.be/) on RESTCONF/YANG and related technologies.
* [gNMI](https://github.com/openconfig/reference/blob/master/rpc/gnmi/gnmi-specification.md) is gRPC over HTTP/2 based protocol as an alternative to RESTCONF.

## Examples
* [Industry YANG files](https://www.yangcatalog.org/) - From yangcatalog.org project. Useful to see if others have modeled similar applications and/or examples models.
* [More Industry YANG files](https://github.com/openconfig/public/tree/master/release/models) - Same as aboive for the OpenConfig quasi-standard
* [Bartender](https://github.com/dhubler/bartend) - Open source, robotic bartender built with FreeCONF.

## Tools and implementations
* [gNMIc](https://gnmic.openconfig.net/) - a gNMI CLI client that provides full support for Capabilities, Get, Set and Subscribe RPCs with collector capabilities.  Of particular interest is the [terminal](https://gnmic.openconfig.net/user_guide/prompt_suggestions/) that supports tab complete on YANG.
* [Ultra Config](https://ultraconfig.com.au/) - Manage configurations in the cloud.
* [MG Soft](https://www.mg-soft.si/) - Thick client to walk RESTCONF compatible endpoints
* [YumaWorks](https://www.yumaworks.com/) - C++ Drivers for RESTCONF among other things
* [Ansible - RESTCONF](https://github.com/ansible-collections/ansible.netcommon) - Support for RESTCONF and related protocols with documentation [here](https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/netconf_connection.html) 
* [Ansible - URI](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/uri_module.html) - RESTCONF client is so simple and Ansible support for REST APIs might be easier to use.
* [ygot](https://github.com/openconfig/ygot) - Alternative to FreeCONF for generating Go code from YANG
* [Watsen Networks SZTP Support](https://watsen.net/docs/sztpd/current/admin-guide/) - For bootstrapping startup configs even before call home requests.
* [Terraform via gNMI](https://networkop.co.uk/post/2019-04-tf-yang/) - Early experiment but could be used to start something more.
* [gNMI Gateway](https://github.com/openconfig/gnmi-gateway) - Includes Prometheus exporter

## Honorable Mention

FreeCONF doesn't currently support these tools but they are in reach.  If these protocol were added to FreeCONF or these projects added support for RESTCONF or gNMI.

### NETCONF

[NETCONF](https://tools.ietf.org/html/rfc6241) is a TCP/IP socket based protocol

* [Yuma123](https://github.com/vlvassilev/yuma123) - Open source implementation of NETCONF
* [SaltStack](https://medium.com/@anthonypjshaw/netops-with-saltstack-and-pynso-3ce45211501)
* [Chef Support](https://www.juniper.net/documentation/en_US/junos-chef11.10/topics/concept/using-chef-for-junos.html)

### SNMP

[SNMP](https://tools.ietf.org/html/rfc1157) is still used for metrics today and might provide some value and would be a simple enough protocol to integrate with. 

* [InfluxDB Support](https://www.influxdata.com/integration/snmp/)
