---
title: "How Does it Work"
weight: 2
description: >
  Very High-level Description of How FreeCONF Works
---

Software applications commonly employ a library, such as FreeCONF, which facilitates the exporting of their configuration, metrics, alerts, and functions as a RESTCONF compatible interface. Developers then define these elements using a specialized syntax known as YANG. Tools may optionally read and interpret the "YANG files" to gain an understanding of the application's management functions and subsequently communicate with the system as appropriate.

To illustrate this concept, consider the hypothetical scenario where an individual has developed a novel toaster service and seeks to imbue it with manageability.

**Steps as a developer:**

1. Describe the management capabilities of the toaster in a [YANG file like this one.](https://github.com/YangModels/yang/blob/master/experimental/odp/toaster.yang).
2. Write and/or generate code to connect toaster software definitions described in the YANG file.

**Steps as a operator:**

1. Start toaster service within your IT infrastruture *(doesn't matter how : docker container, bare metal or physical device)*.
2. Any tool that can utilize REST APIs can perform access management operations. Tools that integrate RESTCONF-client support would likely make things easier.
      
**For Example:**

* An alert tool can read toaster's YANG file and discover there are two events exported by the toaster service: `toasterOutOfBread` and `toasterRestocked`.  Operator can configure alert tool to notify operator on Slack when toaster is out of bread.
* A configuration can get the current toaster configuration and then send an updated copy relying on the configuration being rejected if invalid for any reason.
* A metrics tool can pull toast making metrics in JSON format and at any frequency they like.  In addition, the metrics tool can parse the YANG and pass along the field descriptions to graphing system such as Grafana.  Pushing metrics is possible but requires special server library support.
* A toaster, web admin portal can do any of the above and also call RPC functions like toasting bread by just POSTing JSON data as input and parsing the JSON output response
