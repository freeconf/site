---
title: "Why a standard?"
weight: 3
description: >
  Standards are the key.
---

Standards obviate the need for integration because both sides can communicate using a predetermined protocol.  You don't need to build your own adapter if you can plug right in!

Standards like TCP/IP allowed computers from different networks to communicate without having support for each other's specific networking stack.  SMTP did the same for electronic messages amd HTML/HTTP did it for information.

Are RESTCONF, YANG and gNMI the next standard for application management? They might be, but whatever it is, with a library like FreeCONF, you don't need to care. As new standards emerge the FreeCONF community will support it and your code doesn't need to change.

## Aren't defacto standards good enough?

> *"Tool X is the defacto standard for doing this in the industry so I don't see the need for an official standard"*

Every standard, defacto or official has a life span. Defacto ones might be a few years, official ones can last decades. When a standard lasts long enough for an application to ship with support built in, it changes the game.  An application with built in support does not rely on fragmented set of communities to fill the gap sometime later.  When support is implemented by the application developers it tends to be complete because they know the application the best.

## Additional reading

To get an understanding of the genesis and impact of standards you can read the book [Where the Wizards Stay Up Late](https://www.amazon.com/Where-Wizards-Stay-Up-Late-audiobook/dp/B00AQU7OFS/ref=sr_1_1) to appreciate the before and after of each standard.
