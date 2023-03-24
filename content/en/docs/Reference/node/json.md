---
title: JSON
weight: 100
description: >
  Read and write JSON as FreeCONF nodes
---

## Usecases
* have some JSON to feed into a node
* need to get a node into JSON form
* feeding or verifying API in unit tests

## Special notes

* RESTCONF server already handles all the JSON encoding and decoding so this is really for special cases outside that
* when writing, enumerations by default are strings.  when reading, either strings or numeric values will work

## Reading JSON

```go
nFromStr := nodeutil.ReadJSON(`{"fruit":"banana"}`)
nFromRdr := nodeutil.ReadJSONIO(myOpenFile)
```

## Writing JSON

```go
jsonStr1, err := nodeutil.WriteJSON(sel)
jsonStr2, err := nodeutil.WritePrettySON(sel)

// more customizations
wtr := &JSONWtr{Out:someOpenFile, Pretty:true}
sel.UpsertInto(wtr.Node())
```
