---
title: Node/maps
tags:
  - go
  - restconf
  - server
  - node
weight: 36  
description: >
  Mapping a Go `map[string]*Friend` to a YANG `list`
---

Handing YANG `lists` are bit more complex then handling YANG `containers`.  As usual, FreeCONF imposes no limits on you regarding how to iterate your list.

## Use Cases
* A map of Go structures that are editable.

[Full Source](https://github.com/freeconf/examples/node-list-map)

file: `chipmunk.yang`
```
module chipmunk {
	namespace "freeconf.org";
	prefix "c";

	list friends {
		key name;
		
		leaf name {
			type string;
		}
	}
}
```

```go
