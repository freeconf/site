---
title: Edit lifecycle
weight: 1000
description: >
  In-depth details on developing your applications using FreeCONF and 
  understanding the sequence of callbacks when editing node structures.
---

This is an in-depth, advanced description about how Node implementations handle edits.

When initiating an edit on your application you might want to call custom functions to create data structures at the beginning of the edit, at the end or at some point along the way.  You might also want to implement custom locking. This document will go thru your options using FreeCONF library.

```go
type Node interface {

    // Hook before an edit is made. opportunity to reject potential edit or create locks
    BeginEdit(r NodeRequest) error

    // Called to navigate and process edits
    Child(r ChildRequest) (child Node, err error)
    Next(r ListRequest) (next Node, key []val.Value, err error)
    Field(r FieldRequest, hnd *ValueHandle) error


    // Hook after an edit is made. opportunity to trigger persistance, finalize edit
    // or free locks
    EndEdit(r NodeRequest) error
}
```

### Edit: Delete a node
```
[Root]
    [Parent]
        [Target]    <-- Delete Target
            [Child]
                [Child-Child]
```
|Node| BeginEdit | End Edit |
|---|---|---|
|Root  |3. EditRoot=false,New=false,Delete=false | 6. EditRoot=false,New=false,Delete=false |
|Parent|2. EditRoot=false,New=false,Delete=false | 5. EditRoot=false,New=false,Delete=false |
|Target|1. EditRoot=true,New=false,Delete=true | 4. EditRoot=true,New=false,Delete=true |

NOTES: 
1. The `[Child]` and `[Child-Child]` nodes never get called when their parent is deleted
2. Every single ancester node of `[Target]` would be called on every edit no matter how deep the tree was.  This would mean you could implement a single global write lock on a `[Root]` for any write edit if that is what you needed.  Or a single write to database of entire tree on any edit.

### Edit:Create a new node thru a target node
```
[Root]
    [Parent]
        [Target]    <-- New Child-Child request passed to Target
            [Child]
                [Child-Child]
```
|Node| BeginEdit | End Edit |
|---|---|---|
|Root  |3. EditRoot=false,New=false,Delete=false | 10. EditRoot=false,New=false,Delete=false |
|Parent|2. EditRoot=false,New=false,Delete=false | 9. EditRoot=false,New=false,Delete=false |
|Target|1. EditRoot=true,New=false,Delete=false | 8. EditRoot=true,New=false,Delete=false |
|Child|4. EditRoot=true,New=false,Delete=false | 7. EditRoot=true,New=false,Delete=false |
|Child-Child|5. EditRoot=true,New=true,Delete=false | 6. EditRoot=true,New=true,Delete=false |


## Edit: Edit leaf on a node
```
[Root]
    [Parent]
        [Target]    <-- Edit leaf property on Target
            [Child]
                [Child-Child]
```
|Node| BeginEdit | End Edit |
|---|---|---|
|Root  |3. EditRoot=false,New=false,Delete=false | 6. EditRoot=false,New=false,Delete=false |
|Parent|2. EditRoot=false,New=false,Delete=false | 5. EditRoot=false,New=false,Delete=false |
|Target|1. EditRoot=true,New=false,Delete=false | 4. EditRoot=true,New=false,Delete=false |

