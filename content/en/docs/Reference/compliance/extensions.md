---
title: Extensions
weight: 1000
description: >
  Listing of optional extensions
---

By default FreeCONF's intentions is for strict compliance with RFC.  There are some minor additions that can often be disabled should you require only strict compliance.

## Useful options

### JSON w/o namespaces

If you submit `application/yang-data+json` in either `Accept` or `Content-Type` in HTTP request headers, you get JSON with namespaces per RFC

**Strict Example Response or Request:**
```
{
    "car:engine":{"speed":10,"x:status":running}
}
```

With no MIME types or `?simplified` in URL, you get

**Optional Example Response or Request:**
```
{
    "engine":{"speed":10,"status":running}
}
```

Input can have namespaces or not, doesn't matter.  But if they are supplied, they must be correct. 

**Rational(s):**
* Exposes how YANG files are organized and often that shouldn't matter to API consumer
* noisy, stutter in data returns 
* Only useful for rare name collisions which should be avoided anyway to make APIs more clear anyway

### Simplified base URL

If you submit `application/yang-data+json` in either `Accept` or `Content-Type` in HTTP request headers, these are the base URLS as per the RFC

**Strict Base URLs**
```
/restconf/data/{module:}... - CRUD and `actions`
/restconf/operations/{module:}...  - `rpc`
/restconf/streams/{module:}... - `notifications`
```

With no MIME types or `?simplified` in URL, you get

**Optional Base URL:**
```
/restconf/data/{module:}... - CRUD, `rpcs`, `actions` and `notifications`
```

**Rational(s)**
* Separation was likely for historical reasons and causes unnecessary complicates API usages

### RPC Input/Output Wrapping

If you submit `application/yang-data+json` in either `Accept` or `Content-Type` in HTTP request headers, you get JSON with namespaces per RFC

**Strict Example Input:**
```
{
    "car:input": {
      "gas":30
    }
}
```

**Strict Example Output:**
```
{
    "car:output": {
      "cost":43.56
    }
}
```

With no MIME types or `?simplified` in URL, you get

**Optional Example Input:**
```
{
    "gas":30
}
```

**Optional Example Output:**
```
{
    "cost":43.56
}
```

**Rational:**
* `input` and `output` object wrappers add no value

### Uploading files

[Details]({{< relref "../web-ui#form-processingfile-uploads" >}})

**Rational:**
* Being able to upload files to a REST API is fundamental to any REST API.

### URL Parameters

[FreeCONF specific URL params]({{< relref "../interfacing-with-a-restconf-api#custom-url-params" >}})

**Rational:**
* Very useful when creating web UIs

### Recursive YANG definitions

Referencing a grouping from inside the grouping is not allowed in YANG but is allowed in FreeCONF YANG parser

```
module fileStructure {
  
  grouping directory {
    leaf name {
      type string;
    }
    container parent {
      uses directory;
    }
  }
}
```

**Rational:**
* Recursive relationships should be avoided when possible but sometimes unavoidable so being able to model it is important.  Implementors should use caution to ensure the data model is not recursive as well.  Also when developing tools that navigate the YANG AST, be sure to use flag that denotes a recursive defintion to avoid indefinite recusion.
