---
layout: post
title: Offensive Security Preparation
description: This is a detailed tutorial for making your SEO compatible. It covers every thing that you need to do - step by step!
comments: true
image:
---









<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
======================

## BGP with BFD

Routing protocols have mechanism to detect link failures as well with `HELLO` like mechanism but is not good enough for c

> BFD runs independent of the routing protocol.

### BFD had two modes of operation
- `Asynchronous Mode` : The Asynchronous mode is like the Hello and Holddown timer , when BFD does not receive some of them the session is teared.

> ECHO mode is on by default.
> ECHO packets are encapuslated un UDP .
