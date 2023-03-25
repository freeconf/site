---
title: Basic Nodes
weight: 3
description: >
  You're starting from scratch. i.e. Abstract class.
---

## Usecases:
* high-level routing areas
* `list` nodes
* areas with not a lot of CRUD
* bridges to systems that are not Go structs (e.g. DB, YAML, external REST APIs, etc.)
* part of code generation

## Highlevel Routing

{{% side-by-side %}}
**If you have application code like this...**
```go
type App struct {
    users  *UserService
    fonts  *FontManager
    bagels *BagelMaker
}
```
||
**...and YANG like this...**
```
module anyName {
    container users {
        // ... more stuff here
    }
    container fonts {
        // ... more stuff here
    }
    container bagels {
        // ... more stuff here
    }
}
```
{{% /side-by-side %}}

**...then your node code can be this.**
```go
app := &App{}
node := &nodeutil.Basic{
    OnChild: func(r node.ChildRequest) (node.Node, error) {
        switch r.Meta.Ident() {
            case "users":
                return manageUsers(app.users), nil
            case "fonts":
                return manageFonts(app.fonts), nil
            case "bagels":
                // maybe manageBagels needs access to users as well
                return manageBagels(app.bagels, app.users), nil
        }
        return nil, nil
    },
}
```

You cannot use `Reflect` here because fields are private but also this sets up `manageBagels` with exactly the context it needs.

## Read-only List from Slice

List nodes often deal with managing a Go `slice` or a `map` and returning nodes that deal with items in that list. When doing simple navigation, this is all you need.

{{% side-by-side %}}
**If you have application code like this...**
```go
type Bear struct {
	Cubs []*Cub
}
```
||
**...and YANG like this...**
```
module bear {
	list cub {
		key name;
		config false;
		leaf name {
			type string;
		}
	}
}
```
{{% /side-by-side %}}

**...then your node code can be this.**
```go
bear := &Bear{Cubs: someCubs}
manageCubs := &nodeutil.Basic{
    OnNext: func(r node.ListRequest) (node.Node, []val.Value, error) {
        key := r.Key
        var found *Cub
        if key != nil {
            name := key[0].String()
            for _, cub := range bear.Cubs {
                if cub.Name == name {
                    found = cub
                    break
                }
            }
        } else if r.Row < len(bear.Cubs) {
            found = bear.Cubs[r.Row]
            key = []val.Value{val.String(found.Name)}
        }
        if found != nil {
            return manageCub(found), key, nil
        }
        return nil, nil, nil
    },
}
```

## Editable List from Slice

While you could implement this with `OnNext` as above, there is a nice option called `OnNextItem` to helps you keep things a little cleaner. 

{{% side-by-side %}}
**If you have application code like this...**
```go
type Bear struct {
	Cubs []*Cub
}
```
||
**...and YANG like this...**
```
module bear {
	list cub {
		key name;
		config false;
		leaf name {
			type string;
		}
	}
}
```
{{% /side-by-side %}}

**...then your node code can be this.**
```go
bear := &Bear{}
manageCubs := &nodeutil.Basic{
    OnNextItem: func(r node.ListRequest) nodeutil.BasicNextItem {
        var found *Cub
        return nodeutil.BasicNextItem{
            New: func() error {
                name := r.Key[0].String()
                found = &Cub{Name: name}
                bear.Cubs = append(bear.Cubs, found)
                return nil
            },
            GetByKey: func() error {
                name := r.Key[0].String()
                for _, cub := range bear.Cubs {
                    if cub.Name == name {
                        found = cub
                        break
                    }
                }
                return nil
            },
            DeleteByKey: func() error {
                name := r.Key[0].String()
                copy := make([]*Cub, 0, len(bear.Cubs))
                for _, cub := range bear.Cubs {
                    if cub.Name != name {
                        copy = append(copy, cub)
                    }
                }
                bear.Cubs = copy
                return nil
            },
            GetByRow: func() ([]val.Value, error) {
                var key []val.Value
                if r.Row < len(bear.Cubs) {
                    found = bear.Cubs[r.Row]
                    key = []val.Value{val.String(found.Name)}
                }
                return key, nil
            },
            Node: func() (node.Node, error) {
                if found != nil {
                    return manageCub(found), nil
                }
                return nil, nil
            },
        }
    },
}
```

**tips(s)**
* While this is a bit of code, but no two slices are managed in the same and so having the ability to customize any part of this is rather important. 
* You might find luck resuing this for different slice types with Go's generics
* You can imagine here that if drilling into a single `Cub` by key or by row number and the datasource allowed you get those w/o getting the whole list, this would be rather efficient.

## Editable List from map

{{% side-by-side %}}
**If you have application code like this...**
```go
type Bear struct {
	Cubs []*Cub
}
```
||
**...and YANG like this...**
```
module chipmuck {
	list friends {
		key name;
		leaf name {
			type string;
		}
	}
}	
```
{{% /side-by-side %}}

**...then your node code can be this.**
```go
cmunk := &Chipmuck{}
manageCmunk := &nodeutil.Basic{
    OnNextItem: func(r node.ListRequest) nodeutil.BasicNextItem {
        // get keys in an ordered sequence
        index := node.NewIndex(cmunk.Friend)
        index.Sort(func(a, b reflect.Value) bool {
            return strings.Compare(a.String(), b.String()) < 0
        })
        var found *Friend
        return nodeutil.BasicNextItem{
            New: func() error {
                name := r.Key[0].String()
                found = &Friend{Name: name}
                cmunk.Friend[name] = found
                return nil
            },
            GetByKey: func() error {
                name := r.Key[0].String()
                found = cmunk.Friend[name]
                return nil
            },
            DeleteByKey: func() error {
                name := r.Key[0].String()
                delete(cmunk.Friend, name)
                return nil
            },
            GetByRow: func() ([]val.Value, error) {
                if r.Row < index.Len() {
                    if x := index.NextKey(r.Row); x != node.NO_VALUE {
                        name := x.String()
                        found = cmunk.Friend[name]
                        return []val.Value{val.String(name)}, nil
                    }
                }
                return nil, nil
            },
            Node: func() (node.Node, error) {
                if found != nil {
                    return manageFriend(found), nil
                }
                return nil, nil
            },
        }
    },
}
```

**tips(s)**
* use `index` to build a list of keys so you can a.) return items in order and b.) get items by row number
* you could build the index lazily to avoid doing it if there was no `GetByRow` called.

[Full Source](https://github.com/freeconf/restconf/blob/master/example/site/basic_test.go)