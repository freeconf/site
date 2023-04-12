---
title: What can it do?
weight: 5
tags:
  - project idea
description: > 
  Use cases on FreeCONF's utility today.
---

It is important to remember FreeCONF is a software library.  If you are looking for final applications to download see some options in [Resources]({{< relref "../reference/resources">}}).  With a library however you are in complete control of building what you need.  

## Ideal

While this is the goal:

![Idea](/arch-ideal.png)

The basic unit starts with this:

![Basic Unit](/arch-basic-unit.png)

With this basic unit, you can do a lot.

## Just a few example use cases

1. **Document existing configuration** - Describe your current config in YANG and generate docs.  You'll have to make sure your application code and your YANG stay in sync but this is true of any manual documentation. This way you can separate content from format. YANGs ability to reuse definitions it very useful.
1. **Validating configuration parser** - Use FreeCONF to load your configuration files into your application. Follows the [Inversion of Control](https://en.wikipedia.org/wiki/Inversion_of_control) pattern. Your configuration document will be accurate and configuration will be validated before it hits your code.
2. **Edit live configuration** - Loading configuration from files or REST is no different. You can expose just the configuration that you want to support live changing like debug log level or manage a list of users.
3. **Decoupling metrics systems** - Unlike configuration or structured logging, implementing metrics almost always requires coupling your application code to a proprietary metrics service locking you in.  Use FreeCONF to decouple that. You don't need to capture RPCs, config or events in YANG.  See one of the metrics examples like [Prometheus]({{< relref "../examples/fcprom" >}}) to see how it works.
4. **Wrap existing REST APIs** - Having a schema to a legacy API helps consumers and also makes the API available to RESTCONF and gNMI tools.
5. **General purpose REST API** - I may be biased, but RESTCONF APIs are far more powerful, easier to maintain and easier to consume then OpenAPIs. "Management" is really a perspective. Putting items in your web cart is really just managing objects in another object.
6. **Admin Portal** - While this may seem out of scope for FreeCONF, the ability to organize large sets of configuration quickly while providing both a REST API and  [dynamic user interfaces]({{< relref "../examples/webui" >}}) is quite useful. ![possible architecture](/arch-mgmt-system.png)  You can send configuration to endpoints by calling Ansible playbooks or editing Chef databags.
7. **Bridge to other protocols** - Just like RESTCONF and gNMI are just wire protocols to the basic unit, you can write a bridge to other protcols like JMX, SNMP, DOCSYS, COMI or proprietary protocols.
8. **File Parser** - Generate code based from YANG to decode bytes in a file or wire protocol like msgpack or protobuf that do not contain self-describing meta data.
9. **Generate OpenAPI definitions** - YANG is truly a superset of all other protocols. Partly because it is very complete but also because you can extend the YANG to include meta data used in other definition systems like defining 404 errors. [Here is one project that converts definitions](https://github.com/bartoszm/yang2swagger).  But you can create your own fairly easily by converting the YANG to JSON using `fc-yang` utility and then use jinja or your favorite templing system to convert to openapi definitions.
10. **Generate gRPC definitions** - Rationale and strategy is same as above
11. **Validate YANG files** - Just use the YANG parser in FreeCONF to parse YANG files and look for specific things or ensure they are syntactically correct.
12. **Generating end-to-end tests** - Walk each item in the YANG and generate isolated tests for just that item.


This is not the end of the list by no means, just ideas.