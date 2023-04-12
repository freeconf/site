---
title: Why the project?
weight: 4
description: >
  Why the project and how it is different
---

## Current systems

![Today](/arch-current.png)

### Terms

Let's settle on the following definitions:

* **Management** - Configuration, Metrics, Alerts, RPCs, Security<br>
  *Example RPCs: Backup, drain queue, build report, take snapshot*
* **Applications** - Running software critical to your business<br>
  *Examples: Mysql, HAProxy, Kubernetes, Custom Applications*
* **Tools** - Software used by DevOps to manage your applications<br>
  *Examples: Ansible, Grafana, Kubernetes, Terraform*
* **Integration** - Custom scripts or software written by DevOps to allow tools to manage applications<br>
  *Examples: Ansible playbooks, Terraform plugins, Bash, Python, Admin Utilities*
* **Automation** - High-level data and orchestration. The "Fun Stuff!"<br>
    *Examples: KPI, Application Lifecycle handling, A.I., auto-scaling, auto-repair*

### Obstacles for DevOps

1. DevOps has to build the *"Tool/App Integration"* part only so they can get to the fun *"Automation"* part.
2. Every line of code in *"Tool/App Integration"* locks DevOps into each tool.
3. Applications upgrades or rollbacks often require changes to *"Tool/App Integration"* which require testing and change management control.
4. Each application is different and code reuse can be hit or miss.
5. Copying code from internet to jump start integration often finds incomplete, buggy unmaintained solutions.  Anyway you look at it, you now own it.
6. When you discover a new config, metrics or alert you want to capture, it means redeploying code *"Tool/App Integration"* which requires testing and change management control.
7. Configuration often requires creating a template even when you want to change a setting or two. This template is constantly changing with each application version which means deploying a new template which means more testing and change management control.
8. Each and every management change requires application developers to communicate the change before support is needed and enough time to accomodate the change.
1.  Often you need to support multiple versions of an application which means support both versions of an application in case of rollback.  It also means knowing when to safely remove legacy versions.

### Obstacles for Application Developers

1. Creating new configuration means relying on DevOps to support it first
2. Watching for health related events often means documenting precise logging conventions and then coordinating with DevOps. Once defined the structure is difficult to change and prone to siliently breaking on subtle application changes.  Because these are alerts, you don't find out they are broken until after your customers do.
3. Like health events, creating new application metrics often requires DevOps to write structured log parsers and possibly new ingestion scripts.  DevOp will prioritize metrics that indicate stability over simply informative metrics and could push back in implementing at all.
4. If any of the above is slowing you down and you want to help, you need to learn a lot of tools, how to use them safely, securely and in a maintainable manor.
5. Any time you spend in DevOps is time you are not writing application code
6. For streaming metrics, you need to integrate client libraries into your application code locking you into that tool.  This also means when developing code locally, you need to be able to disable metrics.  This also means issues with metrics streaming won't be discovered until in production.

### Obstacles for Software Vendors

1. When you have multiple services, customers often struggle to get everything perfectly configured together.  You end up building custom tools to help them.  These tools are expensive to build and sometimes conflict with tools customers are already using.
2. Producing helper scripts for each and every management tool is a lot of work and will never meet every customer's needs.
3. Applications metrics are often off the table because it means coupling your software to every conceivable metrics system available.  Only option is logging and hope your customers implement log parsing.  It also means never changing your logging strategy.

## The ideal system!

![Today](/arch-ideal.png)

Here all the applications and tools agree to a predetermined, industry wide, management standard protocol. Tools download the applications published management manifests and operate on behalf of the DevOps teams for automation.  Manifests list every configuration, metrics, alert and RPC.  Once the manifest is exchanged, then tools know how to fullfill automation tasks from the tool on behalf of the DevOps engineer.

Aside from removing every obtacle list above, some more reliefs come forward:

### Reliefs for DevOps

1. Entire configuration, metric, alert, RPC available across every appliction, infrastructure, tool and OS available ready for automation with zero effort.
2. Precise documentation on every management facet.
3. Freedom to change tools or use multiple tools as they see fit without having to change application or integration code.
4. Configuration is validated before being applied. If configuration is wrong, it will not apply.

### Reliefs for Application Developers

1. Empowered to connect all application management operations without requiring any tasks from DevOps.
2. Freedom to use different sets of tool for different environments. Use Prometheus in local development, log to postgres in end to end testing and use AWS Active monitoring for production.  All work on the same application binaries unaltered.
3. Generate all documentation. Documentation is still important so DevOps can decide which options they need to automate.
4. Your applications get deployed faster because integration scripts do not need to be updated or tested. 
5. Safely do rollbacks and again because of no integration code to break

### Reliefs for Software Vendors

1. All your software is compatible with every customers mananagement tools.

## The catch

1. Everyone would have to agree on a standard.
2. Standard would have to be capable enough to match everyone's needs.
3. After agreeing on a standard, all applications and tools would then have to implement the standard.

Let's tackle each one.

### Agreeing on a standard

Every tool and application uses a library to implement the standard.  If there were multiple standards, then the libraries can abstract out the difference by supporting them all.  FreeCONF supports both RESTCONF and gNMI wire protocols.  Any new emerging standards would gladly be added to FreeCONF.

### Capable enough standards

YANG, RESTCONF and gNMI are all very capable. Networks have had years of experiences with these standards and their capabilities are well documented but you should judge for yourself of course.  

### Implementing the standard

1. **Challenge:** I need a library that implements the standard.<br>
   **Answer:** FreeCONF and [others]({{< relref "../reference/resources" >}})
1. **Challenge:** Awareness the standard exists:<br>
   **Answer:** You're here aren't you? Spread the word, share this site.
2. **Question:** Time to implement the standard:<br>
   **Answer:** Take it slow. See ["What can it do"]({{< relref "what-can-it-do" >}}) for strategies to get there in parallel to what you are doing now.
