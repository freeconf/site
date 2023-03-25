---
title: Extend nodes
weight: 2
description: >
  When you want to make adjustments to an existing node's implementation
---

## Use cases:
* Few isolated changes to an existing node
* Wrap a CRUD node but customize editing operations, actions or notifications

## Special notes
* If [`Reflect`]({{< relref "reflect" >}}) was a cake and [`Basic`]({{< relref "basic" >}}) was the plate under the cake, then `Extend` would be the frosting.
* `Extend` is exactly like `Basic` but let's you delegate anything to another node.  So most of `Basic's` documentation also applies here.


## `Reflect` with one exception

{{% side-by-side %}}
**If you have application code like this...**
```go
type Bird struct {
	Name     string
	Location Coordinates
}
```
||
**...and YANG like this...**
```
module anyName {
	leaf name {
		type string;
	}
	leaf location {
		type string;
	}
}
```
{{% /side-by-side %}}

**...then your node code can be this.**
```go
bird := &Bird{}
manage := &nodeutil.Extend{
    Base: nodeutil.ReflectChild(bird), // handles Name
    OnField: func(p node.Node, r node.FieldRequest, hnd *node.ValueHandle) error {
        switch r.Meta.Ident() {
        case "location":
            if r.Write {
                bird.Location.Set(hnd.Val.String())
            } else {
                hnd.Val = val.String(bird.Location.Get())
            }
        default:
            // delegates to ReflectChild
            return p.Field(r, hnd)
        }
        return nil
    },
}
```

**tip(s):**
* When you have a lot of fields of the same type, see [Reflect.OnField]({{< relref "reflect#field-coersion" >}})

[Full Source](https://github.com/freeconf/restconf/blob/master/example/site/extend_test.go)