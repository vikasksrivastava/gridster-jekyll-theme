---
layout: post
title: LISP Overview and a quick Demo
description: LISP Overview and a quick Demo
comments: true
---

<!-- TOC START min:1 max:5 link:true update:true -->
  - [Introduction](#introduction)
    - [xTR1](#xtr1)
    - [xTR-2](#xtr-2)
    - [MS-MR](#ms-mr)
    - [SP-CLOUD](#sp-cloud)
    - [Verification Commands](#verification-commands)

<!-- TOC END -->


## Introduction

Locator ID Separation Protocols m which sepearted the endpoint ID from its location.

LISP has 3 main components :

**The Egress and Ingress Tunnel Routers also known as `XTRs`** : Does the encapsulation and decapsulation of LISP packets.  Botht he functions can be muerged into one is called XTR .

**Proxy Engress/Ingress Tunnel Router or `PXTR`** : Acts as the border between LISP and Non LISP networks.

**MAP Server and MAP Resolver `MR` and `MS`** : The DNS type name resolution system for LISP.


![](/assets/markdown-img-paste-20180528105450485.png)

![](/assets/markdown-img-paste-20180527202335319.png)

### xTR1

```shell
!
!
hostname xTR-1
!

!
interface Loopback0
 no shutdown
 ip address 1.1.1.1 255.255.255.255
 ip ospf network point-to-point
!
interface LISP0
 no shutdown
!
interface GigabitEthernet1
 no shutdown
 ip address 172.16.100.1 255.255.255.0
 negotiation auto
 lisp mobility 172-16-100-0
 lisp mobility 172-16-120-0
 lisp mobility 172-16-130-0
!
interface GigabitEthernet2
 no shutdown
 ip address 10.10.10.2 255.255.255.0
 negotiation auto
!

!
router lisp
 locator-set site-1
  IPv4-interface Loopback0 priority 10 weight 10
  exit
 !
 dynamic-eid 172-16-100-0
  database-mapping 172.16.100.0/24 locator-set site-1
  exit
 !
 dynamic-eid 172-16-120-0
  database-mapping 172.16.120.0/24 locator-set site-1
  exit
 !
 dynamic-eid 172-16-130-0
  database-mapping 172.16.130.0/24 locator-set site-1
  exit
 !
 ipv4 itr map-resolver 3.3.3.3
 ipv4 itr
 ipv4 etr map-server 3.3.3.3 key cisco
 ipv4 etr
 exit
!
router ospf 10
 network 1.1.1.1 0.0.0.0 area 0
 network 10.0.0.0 0.255.255.255 area 0
!

```

### xTR-2

```shell


hostname xTR-2
!
interface Loopback0
 no shutdown
 ip address 2.2.2.2 255.255.255.255
 ip ospf network point-to-point
!
interface LISP0
 no shutdown
!
interface GigabitEthernet1
 no shutdown
 ip address 172.16.120.1 255.255.255.0
 negotiation auto
 lisp mobility 172-16-100-0
 lisp mobility 172-16-120-0
 lisp mobility 172-16-130-0
!
interface GigabitEthernet2
 no shutdown
 ip address 10.10.20.2 255.255.255.0
 negotiation auto
!
!
router lisp
 locator-set site-1
  IPv4-interface Loopback0 priority 10 weight 10
  exit
 !
 locator-set site-2
  IPv4-interface Loopback0 priority 10 weight 10
  exit
 !
 dynamic-eid 172-16-100-0
  database-mapping 172.16.100.0/24 locator-set site-1
  exit
 !
 dynamic-eid 172-16-120-0
  database-mapping 172.16.120.0/24 locator-set site-1
  exit
 !
 dynamic-eid 172-16-130-0
  database-mapping 172.16.130.0/24 locator-set site-1
  exit
 !
 ipv4 itr map-resolver 3.3.3.3
 ipv4 itr
 ipv4 etr map-server 3.3.3.3 key cisco
 ipv4 etr
 exit
!
router ospf 10
 network 2.2.2.2 0.0.0.0 area 0
 network 10.0.0.0 0.255.255.255 area 0
!

```



### MS-MR

```shell
!
hostname MR-MS
!
interface Loopback0
 no shutdown
 ip address 3.3.3.3 255.255.255.255
 ip ospf network point-to-point
!
interface GigabitEthernet1
 no shutdown
 ip address 10.10.30.2 255.255.255.0
 negotiation auto
!
!
router lisp
 site campus
  authentication-key cisco
  eid-prefix 172.16.100.0/24 accept-more-specifics
  eid-prefix 172.16.120.0/24 accept-more-specifics
  eid-prefix 172.16.130.0/24 accept-more-specifics
  eid-prefix 172.16.140.0/24 accept-more-specifics
  exit
 !
 ipv4 map-server
 ipv4 map-resolver
 exit
!
router ospf 10
 network 3.3.3.3 0.0.0.0 area 0
 network 10.0.0.0 0.255.255.255 area 0
!
```



### SP-CLOUD

```shell
!
hostname SP-CLOUD
!
!
interface Loopback0
 no shutdown
 ip address 6.6.6.6 255.255.255.255
 ip ospf network point-to-point
!
interface GigabitEthernet1
 no shutdown
 ip address 10.10.10.1 255.255.255.0
 negotiation auto
!
interface GigabitEthernet2
 no shutdown
 ip address 10.10.20.1 255.255.255.0
 negotiation auto
!
interface GigabitEthernet3
 no shutdown
 ip address 10.10.30.1 255.255.255.0
 negotiation auto
!
!
router ospf 10
 network 0.0.0.0 255.255.255.255 area 0
!
```



### Verification Commands

```shell
LISP Site Registration Information
* = Some locators are down or unreachable

Site Name      Last      Up   Who Last             Inst     EID Prefix
               Register       Registered           ID
campus         never     no   --                            172.16.100.0/24
               00:00:37  yes  10.10.10.2                    172.16.100.2/32
               00:00:16  yes  10.10.20.2                    172.16.100.3/32
               never     no   --                            172.16.120.0/24
               00:00:16  yes  10.10.20.2                    172.16.120.2/32
               00:00:24  yes  10.10.10.2                    172.16.120.3/32
               never     no   --                            172.16.130.0/24
               never     no   --                            172.16.140.0/24
MR-MS#
```


```shell
xTR-1#show ip cef 172.16.120.2/32 detail
172.16.120.2/32, epoch 2, flags [subtree context, check lisp eligibility]
  SC owned,sourced: LISP remote EID - locator status bits 0x00000001
  LISP remote EID: 19 packets 1862 bytes fwd action encap, dynamic EID need encap
  SC inherited: LISP cfg dyn-EID - LISP configured dynamic-EID
  LISP EID attributes: localEID No, c-dynEID Yes, d-dynEID No
  LISP source path list
    nexthop 2.2.2.2 LISP0
  2 IPL sources [no flags]
  nexthop 2.2.2.2 LISP0
xTR-1#

```



```shell
xTR-1#show ip lisp map-cache
LISP IPv4 Mapping Cache for EID-table default (IID 0), 6 entries

0.0.0.0/0, uptime: 01:15:20, expires: never, via static send map-request
  Negative cache entry, action: send-map-request
172.16.100.0/24, uptime: 01:15:20, expires: never, via dynamic-EID, send-map-request
  Negative cache entry, action: send-map-request
172.16.100.3/32, uptime: 00:58:41, expires: 23:01:18, via map-reply, complete
  Locator  Uptime    State      Pri/Wgt
  2.2.2.2  00:58:41  up          10/10
172.16.120.0/24, uptime: 01:15:20, expires: never, via dynamic-EID, send-map-request
  Negative cache entry, action: send-map-request
172.16.120.2/32, uptime: 01:08:00, expires: 22:52:00, via map-reply, complete
  Locator  Uptime    State      Pri/Wgt
  2.2.2.2  01:08:00  up          10/10
172.16.130.0/24, uptime: 01:15:20, expires: never, via dynamic-EID, send-map-request
  Negative cache entry, action: send-map-request
xTR-1#
```
