---
title: "YANG Primer"
weight: 1
---

YANG is a language utilized for modeling the management functions of applications. The primary functions of a YANG file are to enable remote tools to comprehend a server's management functions, enable servers to accurately provide defined management functions, and generate documentation for individuals seeking to understand a server's management functions. As a software engineer who implements management functions, it is crucial to understand how to create YANG files. Although there are numerous books on the subject, this document will provide a high-level understanding. It is recommended to consult the [YANG RFC](https://datatracker.ietf.org/doc/html/rfc7950) for a comprehensive guide on the language.

If you are familiar with other interface definition languages such as gRPC or OpenAPI, you will find several similarities with YANG. However, YANG extends beyond these languages to account for management-specific aspects.

It is essential to note that YANG is not limited to RESTCONF and can be employed with other communication protocols, such as NETCONF, and essentially any other communication protocol.

## Data Definitions

## module 

Every YANG file starts `module {}` statement.  All further definitions are contained inside the `{}` brackets. 

```
module car {
  prefix "c";
  revision 2023-03-11;

  // all further definitions here
}
```

There is always just a single module.

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.1)

### leaf

```
module car {
  leaf driver {
    type string;
  }
}
```

If a server used this YANG file, then possible response for requesting data via `curl` might be:


```
curl https://server/restconf/data/car:

{
 "driver": "joe"
}
```

While the base URL of `/restconf/data/` is not strictly neccessary, it is pretty standard for all RESTCONF servers.

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.6)

### leaf types

| Name | Description |
|------|-------------|
| binary              | Any binary data                     |
| bits                | A set of bits or flags              |
| boolean             | "true" or "false"                   |
| decimal64           | 64-bit signed decimal number        |
| empty               | A leaf that does not have any value |
| enumeration         | One of an enumerated set of strings |
| identityref         | A reference to an abstract identity |
| instance-identifier | A reference to a data tree node     |
| int8                | 8-bit signed integer                |
| int16               | 16-bit signed integer               |
| int32               | 32-bit signed integer               |
| int64               | 64-bit signed integer               |
| leafref             | A reference to a leaf instance      |
| string              | A character string                  |
| uint8               | 8-bit unsigned integer              |
| uint16              | 16-bit unsigned integer             |
| uint32              | 32-bit unsigned integer             |
| uint64              | 64-bit unsigned integer             |
| union               | Choice of member types              |

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.4)

### container 

```
module car {
  container engine {
  }
}
```

Possible request/responses:

```
curl https://server/restconf/data/car:

{
 "engine": {}
}
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.5)

### list

```
module car {
  list cylinders {
    leaf firingOrder {
      type int32;
    }
  }
}
```

Possible request/responses:

```
curl https://server/restconf/data/car:

{
  "cylinders":[
    {
      "firingOrder": 1
    }
    {
      "firingOrder": 4
    }
    {
      "firingOrder": 3
    }
    {
      "firingOrder": 2
    }
  ]
}
```

Most list will have a `key` when referencing a particular item in the list.

```
module car {
  list cylinders {
    key num;
    leaf num {
      type int32;
    }
    leaf firingOrder {
      type int32;
    }
  }
}
```

```
curl https://server/restconf/data/car:

{
  "cylinders":[
    {
      "num": 1,
      "firingOrder": 1
    }
    {
      "num": 2,
      "firingOrder": 4
    }
    {
      "num": 3,
      "firingOrder": 3
    }
    {
      "num": 4,
      "firingOrder": 2
    }
  ]
}

curl https://server/restconf/data/car:cylinders=1

{
  "num": 1,
  "firingOrder": 1
}
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.8)

### container/leaf

```
module car {
  leaf driver {
    type string;
  }
  container engine {
    leaf cylinders {
      type int32;
    }
  }
}
```

Possible request/responses:

```
curl https://server/restconf/data/car:

{
 "driver": "joe"
 "engine": {
  "cylinders": 6
 }
}
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.6)

### leaf-list

```
module car {
  leaf-list owners {
    type string;
  }
}
```

Possible request/responses:

```
curl https://server/restconf/data/car:

{
 "owners": ["joe","mary"]
}
```

leaf-lists can have all the same types as leaf, only it would container multiples of said type.

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.7)

### rpc

```
module car {
  rpc start {
  }
}
```

Possible request/responses:

```
curl -X POST https://server/restconf/data/car:start

# no response but car should start otherwise you'd get an error
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.14)

### rpc/input/output

```
module car {
  rpc drive {
    input {
      leaf throttle {
        type int32;
      }
    }
    output {
      leaf acceleration {
        type int32;
      }
    }
  }
}
```

Possible request/responses:

```
curl -X POST -d '{"throttle":32}' https://server/restconf/data/car:drive

{
  "acceleration": 30
}
```
[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.14.2)


### container/action/input/output

For historical reasons, `action` is exactly like `rpc` except `rpc` are only allowed inside `module` and `action` is used everywhere else.

```
module car {
  
  rpc drive {} // correct

  container engine {
    action start {} // correct
  }
}
```

```
module car {
  
  action drive {} // INCORRECT, only rpc here
  
  container engine {
    rpc start {} // INCORRECT, only action here
  }
}
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.15)

### notification

```
module car {
  notification flatTire {}   
}
```

Possible request/responses:

```
curl https://server/restconf/data/car:flatTire

data: {"notificaton":{"eventTime":"2013-12-21T00:01:00Z"}}
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.16)

### notification/leaf

```
module car {
  notification flatTire {
    leaf tireCount {
      type int32;
    }
  }   
}
```

Possible request/responses:

```
curl https://server/restconf/data/car:flatTire

data: {"notificaton":{"eventTime":"2013-12-21T00:01:00Z","event":{"tireCount":1}}}
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.16)

### anydata

```
module car {
  anydata glovebox;
}
```

Possible request/responses:

```
curl https://server/restconf/data/car:

{
  "glovebox" {
    "papers" : ["registration", "manual"],
    "napkinCount" : 30
  }
}
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.10)

## Organizational

### group/uses

Grouping is simply a way to reuse a set of definitions

This YANG
```
module car {
  
  leaf driver {
    type string;
  }

  uses engineDef;

  grouping engineDef {
    container engine {
      leaf cylinders {
        type int32;
      }
    } 
  }
}
```

is equivalent to this YANG:

```
module car {
  
  leaf driver {
    type string;
  }

  container engine {
    leaf cylinders {
      type int32;
    }
  } 
}
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.12)

### choice/case

When you want to ensure there is just one of a number of definitions. If you are familiar with gRPC, this is like `oneof`.  Some languages call this `union`:

```
module {
  choice nameDoesntMatter {
    leaf a {
      type string;
    }
    leaf b {
      type string;
    }
    leaf c {
      type string;
    }
  }
}
```
This means if `a` exists, then `b` and `c` cannot.  

This you have multiple items in the option, you can wrap them with a `case` statement.

```
module {
  choice nameDoesntMatter {
    case nameAlsoDoesntMatter1 {
      leaf a {
        type string;
      }
      leaf aa {
        type string;
      }
    }
    leaf b {
      type string;
    }
    leaf c {
      type string;
    }
  }
}
```
This means if `a` and/or `aa` exist, then `b` and `c` cannot.

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.9)

### typedef

typedef are simply a way to reuse a leaf type

This YANG

```
module {
  leaf driver {
    type string;
  }
}
```

is equivalent to this YANG

```
module {
  
  typedef person {
    type string;
  }

  leaf driver {
    type person;
  }
}
```
This would be more useful then `type` has more definitions associated with it and more opportunities to reuse the typedef.

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.3)

## Metrics

Metrics are just definitions that are marked `config false`. 

```
module {

  leaf speed {
    config false;  // <-- Metric HERE
    type int32;
  }

  leaf color {
    type string;
  }
}
```

```
module {

  container stats {
    config false;  // <-- children of a containers are metrics too
    leaf count {
      type int32;
    }
    leaf rate {
      type int32;
    }
  }
}
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-4.2.3)

## Constraints

There are a lot more types of contraints, but here are a few;

### number types

```
module car {
  leaf cylinders {
    type int32 {
      range "1..12";
    }
  }
}
```
You can have any number of `range` items that you need.

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-9.2.4)

### string types

```
module car {
  leaf color {
    type string {
      pattern "[a-z]*"; // lowercase
    }
  }
}
```

You can have any number of `pattern` items that you need.

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-9.4.5)

## Extensions

You can customize YANG files with data that is specific to your application. Extensions will be ignored by any systems that do not support your customizations.

```
module car {
  prefix my;

  extension secret;

  leaf owner {
    type string;
    my:secret;
  }
}
```

extensions have have any number of arguments:

```
module car {
  prefix my;

  extension secret {
    argument "vault";
  }

  leaf owner {
    type string;
    my:secret "safe";
  }

  leaf history {
    type string;
    my:secret "jerry";
  }
}
```

[RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.19)

## More

**Even this is not an exhaustive**, but still more useful contructs:

* **import** - pull in select YANG from another file. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.1.5)
* **include** - pull in all YANG from another file. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.1.6)
* **default** - value to use for leafs when no value is supplied. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.6.1)
* **augment** - like grouping/uses but you can add definitions when using a grouping. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.17)
* **refine** - when using `uses` and you want to alter specifc definition including adding constraints. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.13.2)
* **leafref** - a reference to another leaf's data that must exist. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-9.9)
* **when** - data definitions that only exist when certain data is true. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.21.5)
* **must** - a constraint that is tied to other leaf's data. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.5.3)
* **identity** - system wide enum that can have heirarchies. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.18)
* **feature** - denote parts of the YANG that are only valid if a feeature it on. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.20.1)
* **revision** - track versions of your YANG. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.1.9)
* **error-message** - control the error messaging. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.5.4.1)
* **ordered-by** - control the order of lists. [RFC reference](https://datatracker.ietf.org/doc/html/rfc7950#section-7.7.7)

## Example YANG files

* [toaster.yang](https://github.com/YangModels/yang/blob/main/experimental/odp/toaster.yang) - Used as the "hello world" or "TODOs" application for YANG.  It demonstrates common YANG items.
* [fc-restconf.yang](https://github.com/freeconf/restconf/blob/master/yang/fc-restconf.yang) - If you use FreeCONF, this is the configuration of the RESTCONF service you'd be using to handle your RESTCONF interface.
* [ietf-inet-types.yang](https://github.com/freeconf/restconf/blob/master/yang/ietf-inet-types.yang) - IETF types are useful to import as common set of field types/