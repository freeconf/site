---
title: Nodes
weight: 5
description: >
  Using the FreeCONF API to build your management interface
---

Nodes are used to interface from your application to the YANG modeled management interface.  As a software engineer you decide how those nodes are built. Each management request constructs the node heirarchy to navigate to the part of the application to perform the management operation.  When the management operation is complete, the nodes are destroyed.  Each request constructs a different set of nodes.

FreeCONF gives you many options for how to implement your nodes, from implementing the interface to full code generation to anywhere in between.  Each node starting from the registered root node controls child nodes and therefore you can mix node implementations to suit each node's role.

If you're not sure where to start, typically a YANG model uses the similar naming for fields and data structures as the application so I would start with Reflect and Extend. This is the recommended approach for starting out. If your code becomes too repetative, then you can start to generate code to replace parts of your current code and you'll know what code to generate.

