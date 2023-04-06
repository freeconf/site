---
title: "What is it?"
weight: 1
description: >
  A bit more details around just what FreeCONF is.
---

FreeCONF is a software library you add to your application to allow it to be managed by DevOps tools.  Whether you use FreeCONF or any other library that supports the same standard, they all appear the same way to the DevOps tools.  This approach is similar to JMX or SNMP but much, much more powerful.

DevOps tools all provide a variety of ways to integrate applications to their tool. This is different.  Here applications export a "map" of their management capabilities that DevOps tools can use to eliminate the integration step and let you immediately start automation.  You can [read more about how]({{< relref "how" >}}) this is done or [look through the examples to get an idea]({{< relref "../examples" >}}).

What a minute! You still have to integrate a library like FreeCONF into every application and DevOps tool. True, but you do it once for every application and once for every DevOps tool and now they all interoperate with each other forever no matter the application or the tool.  This keeps software developers away from your Ansible playbooks, and DevOps engineers away from writing Terraform Go plugins (unless they want to).

FreeCONF is written in Go but work is in progress to make this available to many other computer languages.  To that end, because this is a standard, you can use any software in replace of FreeCONF that implements the standard and still interoperate with all the same DevOps tools.

What sets FreeCONF apart from other solutions is its utilization of well-established IETF standards such as [RESTCONF, YANG and gNMI]({{< relref "value-of-a-standard" >}}).  Without these standards, this approach would just look like another platform requiring community plugins to be useful.  No plugins or repos of playbooks here!

You don't have to be all in! Incorporate as much or as little of FreeCONF to suit your needs and use this alongside whatever you are using now.
