---
title: Errors
weight: 2000
description: >
  Returning 401, 409 and 400 class errors
---

When coding your `node.Node` implementation, if you return an `error` from any function, your http clients will receive a `500` error, but if you want to return an error with a different HTTP status code, you can use or wrap one of the [predefined errors](https://pkg.go.dev/github.com/freeconf/yang/fc)

```go
import (
  "github.com/freeconf/yang/fc"
)

  ...
  // Option #1 - straight 401 error
  return fc.UnauthorizedError 

  // Option #2 - enhanced but still an 401 error
  return fmt.Errorf("Bad ACL %w", fc.UnauthorizedError)
```
