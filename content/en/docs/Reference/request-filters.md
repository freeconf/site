---
title: "Request Filters"
weight: 1000
description: >
  How to register web request filters to read custom request headers
  and how to relay that information to your nodes.
---

## RESTCONF HTTP Request Access

This is useful for custom authorization or general need to access the HTTP request information

**Steps:**

1.) Register a request [`restconf.RequestFilter`](https://pkg.go.dev/github.com/freeconf/restconf#RequestFilter) with RESTCONF [`restconf.Server`](https://pkg.go.dev/github.com/freeconf/restconf#Server) instance

2.) Filter returns a `context.Context` that contains any custom data that you might extract from the HTTP request like HTTP header information, URL parameters or certificate information.

3.) Values from that context.Context will be made available to all your [`node.Node`](https://pkg.go.dev/github.com/freeconf/yang/node#Node) implementations

Example Code:

```go
s := restconf.NewServer(d)
s.Filters = append(s.Filters, filter)
```

```go
func filter(ctx context.Context, w http.ResponseWriter, r *http.Request) (context.Context, error) {
    return context.WithValue(ctx, "mykey", "my-value"), nil
}
```

```go
func myNode() node.Node {
    return nodeutil.Basic{
        OnAction: func(r node.ActionRequest) (node.Node, error) {
            myval := r.Selection.Context.Value("mykey")
        }
    }
}
```
