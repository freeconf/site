---
title: Node Details
weight: 5
description: >
  Using the FreeCONF API to build your management interface
---

Nodes are used to bridge your application to the YANG modeled management interface.  As a software engineer you decide how those nodes are designed. Each management request constructs the node heirarchy to navigate to the part of the application to perform the management operation.  When the management operation is complete, the nodes are garbage collected.  Parallel requests construct different set of nodes so you can keep state in your request logic.

Unlike gRPC or OpenAPI, FreeCONF was designed to allow you to work *with* your existing application data structures and functions, not separate, generated ones. One reason for the difference is RESTCONF let's the client pick and choose the data they want returned.  Generating full data structures only to be partially populated is inefficient at best.  Luckily FreeCONF gives you many options for how to implement your nodes, from a blank slate, to reflection, to code generation to anywhere in between.

Each node starts from a root node registered with the `Browser` object.  Each node controls how its child nodes are constructed and therefore you can mix node implementations to suit each part of your existing application's design.

If you're not sure where to start, typically a YANG model uses the similar naming for fields and data structures of the application so I would start with [`Reflect`]({{< relref "../../examples/node-reflect" >}}) and [`Extend`]({{< relref "../../examples/node-extend" >}}). If your code becomes too repetative, then you can start to look at strategies for generating code to replace parts of your current code.  This phased approach is useful because you'll know what code to generate and replace.

