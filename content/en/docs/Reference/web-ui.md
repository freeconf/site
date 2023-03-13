---
title: "Web UIs"
weight: 1000
---

You can build powerful web interfaces that interface with RESTCONF APIs. While automation is primarily about APIs and not Web UIs, administration portals have proven invaluable in many conditions.  You can deliver your web app with FreeCONF library or obviously serve it elsewhere.  Serving with your application means it is always up to date with backend.

Below are some very powerful constructs you can use when building your web applications.

Interacting with a RESTCONF API requires no special libraries not even for real-time event messaging as details below.

## Adjustable Data Granularity

Traditional REST APIs struggle with what level of information to return for each GET request and this is called [granularity](https://dzone.com/articles/restful-api-design-principle-deciding-levels-of-gr).  Return too little and scripts will need to make constant trips for more data.  Return too much and data is thrown away.

[RESTCONF APIs]({{< ref "interfacing-with-a-RESTCONF-API" >}}) do not have this issue.  Clients can control precisely the data they want from depth, to list pagination. From including select field to excluding select fields.

With APIs developed with FreeCONF, each implemention has the control to only the data that was requested. It is not implemented as a reponse filters.

## Model Driven

Unique to FreeCONF, UIs can request any part of the YANG in JSON form and dynamically building user interfaces.  Just a few examples include:
1. options in a select list from leaf enumerations
2. list of columns in a table
3. descriptions of every object, list and field
4. data types for any field.
5. client-side form validation thru leaf patterns and ranges 
6. devise your UI related meta information like which fields are password or which require custom handlers.

The path to the meta definitions is just `/restconf/schema/{module}/` . 

Access to YANG files in JSON resolved form


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

## Custom Request/Response Handling

FreeCONF will let you serve static assests like HTML/JS/CSS/images as well as register custom web handlers to augment your RESTCONF API with straight REST or whatever you like.

## Reactive Using notifications

Subscribing to `notifications` is [one line of javascript]({{< ref "interfacing-with-a-RESTCONF-API.md#subscribing-to-events-in-web-browser" >}}) with no additional javascript library required.  Notifications aren't just for alerts. One of the more useful notifications is for data has changed in back-end from possibly another user edit and front-end should reload data.

You can serve any number of modules in an application should you need to isolate your web-only functions.

You might have expected websockets for notifications.  Websockets would still require implement a publish and subscribe layer on top of the "socket".  HTTP/2 and SSE this is no longer neccessary resulting in faster and easier code with little risk of orphaned subscriptions.

