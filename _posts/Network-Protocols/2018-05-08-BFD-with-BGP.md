---
layout: post
title: BGP with BFD on EVE-NG with CSR1000v
description: BGP with BFD on EVE-NG with CSR1000v
comments: true
featimg: NSO.png
tags: [nso, automation, yang]
category: [network-automation]
sidebar_toc:	true
---

## BGP with BFD on EVE-NG with CSR1000v

BFD (Bi-Directional Forwarding Detection) is a superfast protocol to detect link failures withing milliseconds or microseconds!

Routing protocols have mechanism to detect link failures as well with `HELLO` like mechanism but is not good enough for convergence where technologies like VOIP is involved. We can tune the routing protocol for exmaple OSPF to use a dead interval of one second. (Which for VOIP is too high)


> BFD runs independent of the routing protocol.

Once BFD is UP you can use other routing protocols to leverage is for fast convergence.

> BFD is not up until you use the same in a protocol.

Once BFD stops receiving packets from the peer , it notifies the protocol using it about the same. (Which is faster than the protocols native dead timer expiry)

### BFD had two modes of operation
- `Asynchronous Mode` : The Asynchronous mode is like the Hello and Holddown timer , when BFD does not receive some of them the session is teared.
- `Demand Mode` : In this once BFD has found a neighbour it wont continuously send packets to neighbour but uses only a polling mechanism. For example noticing the rx/tx of the interface. As of now neither Cisco nor any other vendor supports this mode.

Both modes support `echo` mode , in which a device sends a echo packet and other device reponds withour processing it. If the sender does not get the response the neighborship is teared.

BFD is basically a keepalive system whic utilizes UDP and CEF.
Orginally BFD was designed for directly connected peers.

BFD can utilize two type of packets:
1. ECHO (Echo Packets came later , these are not received by CPU)
2. CONTROL (Processed by CPU)

> ECHO mode is on by default.
> ECHO packets are encapuslated un UDP .

Echo packets go between the interfaces , the loss of the Echo packet triggers the notification to the upper level process.

Echo packets are not processed by the CPU but Control packets are .

So how does BFD do the ECHO without involving the CPU ?

ECHO Packet consists of the `senders` IP and MAC in both source and destination!
To elaborate it , the BFD echo packets sent by X will have X's MAC and IP in Source and Destination!

Now when the packet is sent to the directly connected device (which has CEF enabled) look at the destination and forwards it to it without being processed by the CPU !

You can turn off echo mode (Generallt you would not) .

So if you rely on Control Packet , you basically are taxing CPU.


<img src="/assets/bfd.png" alt="" style="width: 800px;"/>

Intial BFD is configured on the interface :

`bfd interval 100 min_rx 200 multiplier 3`

First Value : I would like to transmit BFD echo packets every 100msec.
Second Value : The fastest I can process incoming BFD ECHO packets is every 200msec. Dont send any faster than that .

### From Cisco Documentation
`[no] bfd interval <50-999> min_rx <1-999> multiplier <3-50>`
- `interval`: Determines how frequently (in milliseconds) BFD packets will be sent to BFD peers.
- `min_rx`: Determines how frequently (in milliseconds) BFD packets will be expected to be received from BFD peers
- `multiplier`: The number of consecutive BFD packets which must be missed from a BFD peer before declaring that peer unavailable, and informing the higher-layer protocols of the failure.



| R1 | R2
| ------  | ------ |
| `bfd interval 100 min_rx 200 multiplier 3`| `bfd interval 300 min_rx 75 multiplier 4` |

1. In the above example `R1` send the min_rx as 200 , `R2` see that and says "I was planing to send it every 300ms but since you can process faster , I will send faster (i.e every 200ms)" .

And becuase it has a multiplier of `4` , If it does not receive it within 4x200 (800ms) it will tear the relationship.

2. In the above example `R2` sends the min_rx  as 75 , `R1` see that and says "I was planing to send it every 100ms but since you can process faster , I will send faster (i.e every 75ms)" .

And becuase it has a multiplier of `3` , If it does not receive it within 3x75 ms it will tear the relationship.

> So basically your min_rx is the speed your neighbor send the packets to you .



```sh
CSR3#show running-config int gi1
Building configuration...

Current configuration : 141 bytes
!
interface GigabitEthernet1
 ip address 192.168.12.1 255.255.255.0
 bfd interval 50 min_rx 50 multiplier 3
end

CSR3#show running-config | sec bgp
router bgp 1
 timers bgp 10 11 10
 neighbor 192.168.12.2 remote-as 2
 neighbor 192.168.12.2 fall-over bfd
```

```sh
CSR4#show running-config int gi1
Building configuration...

Current configuration : 131 bytes
!
interface GigabitEthernet1
 ip address 192.168.12.2 255.255.255.0
 negotiation auto
 bfd interval 50 min_rx 50 multiplier 3
end

CSR4#show running-config | sec bgp
router bgp 2
 timers bgp 10 11 10
 neighbor 192.168.12.1 remote-as 1
 neighbor 192.168.12.1 fall-over bfd

CSR4(config-router)#
*Mar 30 21:19:31.028: %BFD-6-BFD_SESS_CREATED: BFD-SYSLOG: bfd_session_created, neigh 192.168.12.1 proc:BGP, idb:GigabitEthernet1 handle:1 act
*Mar 30 21:19:31.147: %BFDFSM-6-BFD_SESS_UP: BFD-SYSLOG: BFD session ld:4097 handle:1 is going UP
*Mar 30 21:19:41.187: %SYS-5-CONFIG_I: Configured from console by console
*Mar 30 21:19:46.680: %BFDFSM-6-BFD_SESS_DOWN: BFD-SYSLOG: BFD session ld:4097 handle:1,is going Down Reason: ECHO FAILURE
*Mar 30 21:19:46.682: %BGP-5-NBR_RESET: Neighbor 192.168.12.1 reset (BFD adjacency down)
*Mar 30 21:19:46.682: %BGP-5-ADJCHANGE: neighbor 192.168.12.1 Down BFD adjacency down
*Mar 30 21:19:46.682: %BGP_SESSION-5-ADJCHANGE: neighbor 192.168.12.1 IPv4 Unicast topology base removed from session  BFD adjacency down
*Mar 30 21:19:46.683: %BFD-6-BFD_SESS_DESTROYED: BFD-SYSLOG: bfd_session_destroyed,  ld:4097 neigh proc:BGP, handle:1 act

```



> For example if you have configured the BFD timers as 100msecs  with multiplier as 3 so that BFD session will go down if the device did not get response within 300 msecs
For the Voice network we need to have less 200 ms delay in the network otherwise the voice communication will break. Please consult with your ISP while setting up BFD with them.
