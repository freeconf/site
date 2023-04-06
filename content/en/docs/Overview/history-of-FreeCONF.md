---
title: "Why the project?"
weight: 4
description: >
  Brief history around why the project was created.
---

*By Douglas Hubler*

In 2001, I was first introduced to management software for a telephony system, which marked the beginning of my involvement in this field. Together with my colleague, Damian Kremenski, we developed a basic meta language to assist with management duties, which proved to be crucial in our work. Over the course of the next decade, we dedicated ourselves to developing plugin APIs to allow third-party developers to add services to the telephony system, with our modeling language serving as a key component.

While we were proud of the management system we had created, we were also aware of the challenges involved in accessing it due to its proprietary meta language and reliance on plugins.

In 2014, I transitioned to a new role in the networking field, where I became familiar with two management standards, NETCONF and YANG. YANG is a modeling language, while NETCONF is a protocol that leverages said management model. Given my experience in developing a management meta language previously, I quickly recognized the value of YANG. While NETCONF was powerful, I found its newer REST-based counterpart, RESTCONF, to be more exciting as it was expected to increase adoption rates. 

As a user of these standards, I was impressed with how they addressed interoperability, which led me to envision the potential for an open-source implementation of these standards that could be used in general-purpose application development. This solution would break free from the limitations of proprietary management tools and the plugin-oriented systems that I had contributed to in the past.

Between 2015 and the present day, I have been actively implementing the YANG and RESTCONF standards. Throughout the development process, my main objectives were to ensure that the library did not interfere with developers' work and to make complex tasks possible while keeping simpler tasks easy.

In 2023, ground broke on supporting Python.  After many attempts to port FreeCONF to other languages, we finally found a way that is scalable, maintainable and familiar to Python developers.  This same strategy can be used to support every language. This is still a work in progress but well past proof of concept.

Also in 2023 support was added gNMI; a wire protocol that can work in replace of RESTCONF.  Not only did adding gNMI bring a greater set of DevOps tool support it remains completely abstract and optional for all applications that use FreeCONF.

I cannot see DevOps in the same way now.  Ansible playbooks look like homework ripe with fleeting assumptions. Terraform plugins look endless. JuJu looks sexy but too few applications. Datadog integration looks like a decision that cannot be easily undone.  All of these tools are amazing and necessary but if each added support for these management standards then they would remove the burdon put on their community to build elaborate bridges just to use them.
