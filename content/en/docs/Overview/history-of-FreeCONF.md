---
title: "Why the Project"
weight: 4
---

*By Douglas Hubler*

In 2001, I was first introduced to management software for a telephony system, which marked the beginning of my involvement in this field. Together with my colleague, Damian Kremenski, we developed a basic meta language to assist with management duties, which proved to be crucial in our work. Over the course of the next decade, we dedicated ourselves to developing plugin APIs to allow third-party developers to add services to the telephony system, with our modeling language serving as a key component.

While we were proud of the management system we had created, we were also aware of the challenges involved in accessing it due to its proprietary meta language and reliance on plugins.

In 2014, I transitioned to a new role in the networking field, where I became familiar with two management standards, NETCONF and YANG. YANG is a modeling language, while NETCONF is a protocol that leverages said management model. Given my experience in developing a management meta language previously, I quickly recognized the value of YANG. While NETCONF was powerful, I found its newer REST-based counterpart, RESTCONF, to be more exciting as it was expected to increase adoption rates. 

As a user of these standards, I was impressed with how they addressed interoperability, which led me to envision the potential for an open-source implementation of these standards that could be used in general-purpose application development. This solution would break free from the limitations of proprietary management tools and the plugin-oriented systems that I had contributed to in the past.

Between 2015 and the present day, I have been actively implementing the YANG and RESTCONF standards. Although I initially spent two months working on the project alone, the rest of the time was spent developing and supporting the library in a production environment, ensuring its stability and ease of use.

Throughout the development process, my main objectives were to ensure that the library did not interfere with developers' work and to make complex tasks possible while keeping simpler tasks easy.
