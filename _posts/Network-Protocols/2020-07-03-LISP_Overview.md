---
layout: post
title: LISP Overview
description: Basic Introduction to LISP 
comments: true
featimg: Lisp-logo-450x253.jpg
tags: [DNA, Protocol]
category: [SDAccess]
sidebar_toc:	true
---

This is a post to get y ou started with the LISP protocol quick, This is a good primer for SD Access.

*If you would like to follow along this exercise on EVE-NG , download the file from here. This also has the base configs needed for bringign the routign up.*
<br>
<a href="#link" class="btn btn-info" role="button">Download </a>


# Introduction

LISP is an Overlay Routing Architecture , which separates the endpoint identity from its location.

LISP is very analogous to a DNS system. 

![](/assets/LISPDNS.png)


Locator ID Separation Protocols  which separated the endpoint ID from its location.

## LISP Components 
*LISP has 3 main components* :

- **The Egress and Ingress Tunnel Routers also known as `ETR or ITR`** : Does the encapsulation and decapsulation of LISP packets. There are Egress and Ingress Tunnel Routesr (ETR and ITR). When both the functions can be merged into one is called **`XTR`**.


- **MAP Server and MAP Resolver `MR` and `MS`** : The DNS type name resolution system for LISP.
  - The **MAP Resolver (MR)** receivers MAP-REQUESTS from ITRs and forwards them to the mapping system. 

  - The **MAP Server (MS)** Advertises EID (Endpoint ID) inot the LISP mapping system for ETRs that register to it. 

- **Proxy Engress/Ingress Tunnel Router or `PXTR`** : Acts as the border between LISP and Non LISP networks. Not used in the example here though. 

![](/assets/LISPNet.png)

# Configuration Example

-  In this config below , all the basic connectivity between sites and the service provider is configured. 
-  OSPF is being used as the routing protocol. 
-  Next we will see the LISP related config only. 



### Step 1. Start the Config with MAP Server/ Resolver `MS-MR` 

This is the heart of the system ; which is has the entire MAPPING table of the network. 



```shell
hostname xTR-1

router lisp
 # We define sites and the prefixes 
 # which will be reachable via that site.
 site campus
  authentication-key cisco
  # You can create multiple sites for each 
  # prefixes but here we will us just one campus.
  eid-prefix 172.16.100.0/24 accept-more-specifics
  eid-prefix 172.16.120.0/24 accept-more-specifics
  eid-prefix 172.16.130.0/24 accept-more-specifics
  eid-prefix 172.16.140.0/24 accept-more-specifics
  exit
 !
 # Finally enable the services
 ipv4 map-server
 ipv4 map-resolver
 exit
!
```


### Step 2. Next we configure one of the site routers . Configure the xTR-1

```shell
!
router lisp
 # Used when there are muliple ETRs at a site and this can be used for grouping or redundancy puposes.
 locator-set site-1
  # define the source interface that will be used for LISP 
  # The weights and priorotu are again used when there are multiple routers.
  ipv4-interface Loopback0 priority 10 weight 10
  exit
 !
 # Now we define the EID blocks that will be availiable from this site (xTR1)
 # Since we are showing LISP Mobility we will use dynamic . If it was static we could have used static. 
 # In this example all subnets are availaible on all sites
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
 # Enable and Point to MS-MR
 ipv4 itr map-resolver 3.3.3.3
 ipv4 itr
 ipv4 etr map-server 3.3.3.3 key cisco
 ipv4 etr
 exit
!
```

```shell
!
# Enable the mobility under the needed 
# interface for the defined networks
interface GigabitEthernet1
 no shutdown
 ip address 172.16.100.1 255.255.255.0
 negotiation auto
 lisp mobility 172-16-100-0
 lisp mobility 172-16-120-0
 lisp mobility 172-16-130-0
!
```

### Step 3. Similarly we configure xTR-2


```shell


hostname xTR-2
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

!
interface GigabitEthernet1
 no shutdown
 ip address 172.16.120.1 255.255.255.0
 negotiation auto
 lisp mobility 172-16-100-0
 lisp mobility 172-16-120-0
 lisp mobility 172-16-130-0
!

```



## Verification Commands

*This command below show the registration status of various sites in the LISP domain*
```shell
MR-MS>en
MR-MS#show lisp site
LISP Site Registration Information
* = Some locators are down or unreachable
# = Some registrations are sourced by reliable transport

Site Name      Last      Up     Who Last             Inst     EID Prefix
               Register         Registered           ID
campus         never     no     --                            172.16.100.0/24
               00:22:31  yes#   1.1.1.1                       172.16.100.2/32
               00:00:48  yes#   2.2.2.2                       172.16.100.3/32
               never     no     --                            172.16.120.0/24
               00:26:51  yes#   2.2.2.2                       172.16.120.2/32
               00:00:14  yes#   1.1.1.1                       172.16.120.3/32
               never     no     --                            172.16.130.0/24
               never     no     --                            172.16.140.0/24
MR-MS#
```

*Notice that on the xTR router below , we do not have a routing table entry for the 172.16.100.2 host* **But we are still able to reach it**

```shell
xTR-2#sh ip route 172.16.100.2
% Subnet not in table 
```
**Becuase its is being reached out on the LISP interface** **This helps save the resources in the xTRs as well**



```shell
xTR-2#sh ip cef  172.16.100.2
172.16.100.2/32
  nexthop 1.1.1.1 LISP0
xTR-2#

xTR-2#sh ip cef  172.16.100.2 detail
172.16.100.2/32, epoch 0, flags [subtree context, check lisp eligibility]
  SC owned,sourced: LISP remote EID - locator status bits 0x00000001
  LISP remote EID: 20 packets 1680 bytes fwd action encap, cfg as EID space, dynamic EID need encap
  SC inherited: LISP cfg dyn-EID - LISP configured dynamic-EID
  LISP EID attributes: localEID No, c-dynEID Yes, d-dynEID No
  LISP source path list
    nexthop 1.1.1.1 LISP0
  2 IPL sources [no flags]
  nexthop 1.1.1.1 LISP0



```


*The command below show the cache entries on the XTR device*

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

*A sample packet capture done on the MAP Servers for the request coming in and the responses going out*

![](/assets/LISPPACKETCAPTURE.png)
