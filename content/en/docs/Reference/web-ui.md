---
title: "Web UIs"
weight: 1000
description: >
  Building a Web UI interface to a RESTCONF implementation is pure joy.
  This is a few tips that will server you well.
---

You can build powerful web interfaces that interface with RESTCONF APIs. While automation is primarily about APIs and not Web UIs, administration portals have proven invaluable in many conditions.  One advantage to serving your web interface with your application is that it is always up to date with the backend.

Interacting with a RESTCONF API requires no special libraries not even for subscribing to bi-directional events.

## Adjustable Data Granularity

Traditional REST APIs struggle with what level of information to return for each GET request and this is called [granularity](https://dzone.com/articles/restful-api-design-principle-deciding-levels-of-gr).  Return too little data and scripts will need to make constant trips for more.  Return too much data and majority of it will likely be unused.  Both situations cause delays and additional resource consumption. 

[RESTCONF APIs]({{< ref "interfacing-with-a-RESTCONF-API" >}}) do not have this issue.  Clients can precisely control the data they want from depth, to list pagination. From selecting fields to excluding fields.

With APIs developed with FreeCONF, each implemention has the control to only read the data that was requested. That is, granularity is not implemented as  a reponse filters like GraphQL.

## Model Driven

Unique to FreeCONF, clients can request any part of the YANG in JSON form and dynamically build user forms, navigation and report interfaces.

**Examples include:**

1. select list options from leaf enumerations
2. list of available columns in a table
3. tooltip descriptions of every object, list and field
4. data types and labels for any form field
5. client-side form validation thru string patterns and number ranges 
6. custom UI meta data thru YANG extensions such as
   1. marking password fields
   2. marking fields that require custom handlers
   3. fields that should be shown to advanced users
   4. many more..

The path to the meta definitions is just `/restconf/schema/{module}/`

## Form Processing/File Uploads

Unique to FreeCONF, you can upload files to RESTCONF APIs.  For following YANG

```
module school {
  rpc uploadReportCard {
    input {
        anydata reportCard;

        leaf studentName {
            type string;
        }
    }
  }
}
```

You can use the following Javascript

```javascript
  const reportCard = document.getElementById("my-file-upload-input").files[0];
  const form = new FormData();
  form.append("reportCard", reportCard);
  form.append("studentName", "joe");
  fetch("/restconf/data/school:uploadReportCard", {
    method: "POST",
    body: form
  });
```

The Node implementation will get `reportCard` as a stream reader in whatever format is uploaded.

## Serve Static Web Assets

FreeCONF will let you serve static assests like HTML/JS/CSS/images with your application.

## Register Custom Request Endpoints

Have REST methods that cannot be captured in RESTCONF? Just register custom web handlers to augment your RESTCONF API with straight REST or gRPC or whatever you like.

## Reactive Using notifications

Subscribing to `notifications` is [one line of javascript]({{< ref "interfacing-with-a-RESTCONF-API.md#subscribing-to-events-in-web-browser" >}}) with no additional javascript library required unlike websockets.  Notifications aren't just for alerts. One of the more useful notifications is for data has changed in back-end from possibly another user edit and front-end should reload data.

You can serve any number of modules in an application should you need to isolate your web-only functions.  For example `car` module and `car-web` module both from the same server.

You might have expected websockets for notifications.  Websockets would still require implement a publish and subscribe layer on top of the "socket".  HTTP/2 and SSE this is no longer neccessary resulting in faster and easier code with little risk of orphaned subscriptions.

## Generate REST DOCs

Generate REST API docs be reading every detail from the YANG file. Consumable by users that have never even heard of RESTCONF.