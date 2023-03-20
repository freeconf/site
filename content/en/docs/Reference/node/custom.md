---
title: Custom
weight: 1
description: >
  Writing plain old code to implement management API
---

This is the recommended approach for starting out. If your code becomes too repetative, then you can start to generate code to replace parts of your current code.

## Reflect

Use Reflect when your code mostly aligns with the fields in the YANG file.  Small variations can be managed by combining Reflection and Extend.

## Extend

Use Extend when you want to make small variations to any existing Node.

## Basic

Use Basic when you want no existing logic and implement everything yourself. In some languages this is called an "Abstract class".



