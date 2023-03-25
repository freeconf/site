---
title: Dump
weight: 101
description: >
  Debugging aid to see all traffic in/out of a node
---

## Use cases:
* See all operations performed on a node when you're not sure you are getting the right data in or out.

## Usage

```go
  // replace this
  n := myNode()
  // ... with this
  n := nodeutil.Dump(myNode(), os.Stdout)
```

**Example Edit**
```
BeginEdit, new=false, src=car
->speed=int32(10)
EndEdit, new=false, src=car
```

**Example Read**
```
<-speed=int32(10)
<-miles=decimal64(310.000000)
```

**tips:**
* Take a look at the source and create your own dumper that is maybe better.
* You can do this at root node or any part of the tree you want to inspect