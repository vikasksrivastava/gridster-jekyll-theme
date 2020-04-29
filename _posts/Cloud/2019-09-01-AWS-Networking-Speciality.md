---
layout: post
title: AWS Networking Speciality
description: My Notes on AWS Network Speciality
comments: true
featimg: AWSNetworkingSpeciality.png
tags: [AWS, Cloud, Networking]
category: [Cloud]
sidebar_toc:	true
---



### To be done

> - Labbing VGW with Ubiquiti
> - Labbing NAT-G vs IGW (@ host in the same subnet with one host havin a Floating IP so that we can SSH in and then ssh to the other host to test.) . TO check if a Host can access internet without NAT-G

<!-- TOC -->

- [To be done](#to-be-done)
- [Foundational Concepts](#foundational-concepts)
  - [IPv4 Subnetting](#ipv4-subnetting)
    - [How to Calculate Binary (Divide Method)](#how-to-calculate-binary-divide-method)
- [Memorize the following](#memorize-the-following)
- [Practice Examples](#practice-examples)
  - [DNS Basics](#dns-basics)
  - [DNS History](#dns-history)
  - [The Domain Name System](#the-domain-name-system)
  - [Domains, Delegation and Zones](#domains-delegation-and-zones)
    - [Domains](#domains)
  - [Delegation](#delegation)
  - [Internationalised Domain Names](#internationalised-domain-names)
    - [Punnycode](#punnycode)
  - [Name Servers , Zones and Authority](#name-servers--zones-and-authority)
    - [Name Server](#name-server)
  - [DNS Query and Responses](#dns-query-and-responses)
  - [Authoritative , Recursive and Caching Name Servers](#authoritative--recursive-and-caching-name-servers)
  - [Serial Numbers and Zone Data Transfer](#serial-numbers-and-zone-data-transfer)
  - [Forwarding and Forwarders](#forwarding-and-forwarders)
    - [Resource Records](#resource-records)
    - [The A Record and the AAAA Record](#the-a-record-and-the-aaaa-record)
    - [SRV Record (Service Record)](#srv-record-service-record)
    - [TXT Record](#txt-record)
    - [Infrastructure Record](#infrastructure-record)
- [IPSEC VPN (from CCIE Notes)](#ipsec-vpn-from-ccie-notes)
  - [MED Attribute (from BGP Technotes )](#med-attribute-from-bgp-technotes-)
  - [Physical Organisation](#physical-organisation)
  - [AWS Region](#aws-region)

<!-- /TOC -->

## Foundational Concepts

**This is a seven-layer model, which we show in Figure 1.14, along with the `approximate mapping` to the Internet protocol suite.**

![](/assets/markdown-img-paste-20190727135403543.png)

### IPv4 Subnetting

#### How to Calculate Binary (Divide Method)

**Convert 75 to binary**

> Keep Dividing 75 by 2 and Write the number in Reverse
  75
  37 **1**
  18 **1**
  9  **0**
  4  **1**
  2  **0**
  **1**  **0**
Answer : **1001011**


Helpful Series of Numbers for Subnetting
> **128 64 32 16 8 4 2 1**

## Memorize the following
**Powers of 2**

![](/assets/markdown-img-paste-20190727165622171.png)
--
![](/assets/markdown-img-paste-20190727173837434.png)
--
## Practice Examples



----
**EXAMPLE 1** **Subnet: 255.255.255.128**

**Step 1.** Based on the Sequence `128 64 32 16 8 4 2 1` , the Subnet bit is `1` and there are `7` bits for host.

> Magic Number here is `128`

- **Network Bits** = $2^1$ = 2
- **Host Bits** = $2^7$ = 128 (-2 for network and broadcast)
- **Valid Subnet Ranges** = 256-128 = 128 `(0,128,256)`


----
**EXAMPLE 2** **Subnet: 255.255.255.192 (/26)**

> Magic Number here is `64`

**Step 1.** Based on the Sequence `128 64 32 16 8 4 2 1` , the Subnet bit is `2` and there are `6` bits for host.

- **Network Bits** = $2^2$ = 4
- **Host Bits** = $2^6$ = 64 (-2 for network and broadcast)
- **Valid Subnet Ranges** = 256-192 = 64 `(0,64,128,192....)`

----
**EXAMPLE 3** **Subnet: 255.255.255.248 (/29)**

> Magic Number here is `8`

**Step 1.** Based on the Sequence `128 64 32 16 8 4 2 1` , the Subnet bit is `5` and there are `3` bits for host.

- **Network Bits** = $2^5$ = 32
- **Host Bits** = $2^3$ = 8 (-2 for network and broadcast)
- **Valid Subnet Ranges** = 256-248 = 8 `(0,8,16,24,32,40....)`



> **Notes**
![](/assets/markdown-img-paste-20190727173103222.png)



![](/assets/markdown-img-paste-20190727183626821.png)


![](/assets/markdown-img-paste-20190727203734908.png)


### DNS Basics


![](/assets/markdown-img-paste-20190818161605696.png)

---

![](/assets/markdown-img-paste-20190818161623299.png)

> **The 13 Anycast IP Address are operated by 12 Organisations**

> **Why there are only 13 root servers?**
At the time the DNS was designed, the IP address in use was IPv4, which contains 32 bits. For efficient networking and better performance, these IP addresses should fit into a single packet (using UDP, the DNS’s default protocol). Using IPv4, the DNS data that can fit into a single packet is limited to 512 bytes. **As each IPv4 address requires 32 bytes, having 13 servers uses 416 bytes, leaving up to 96 bytes for the remaining protocol information.**

As an IPv6 address is not limited by these constraints, it is possible new root servers may be established in the future.

----

### DNS History

Before DNS , there was a large HOSTS.TXT which has a mapping of all the ARPANETs hosts.

This file was managed and distrubuted by a single organisation (SRI-NIC) and hence the host name `sri-nic`

Here's what it looked like :


![](/assets/markdown-img-paste-20190820111620251.png)

> <span style="color:red"> It was extremely difficult to manage and update hosts with this approach. Lookup on the file was seqeuntial , updates were slow and new host addition tool a lot of time.

### The Domain Name System

![](/assets/markdown-img-paste-20190823163342645.png)

### Domains, Delegation and Zones

#### Domains

Domains are group of related nodes , thy are subtree of the the namespace they are in.


---

### Delegation

Delegation is a process allocation Sub-Domains to different companies for their management , MACD and upkeep.

<br><br>

![](/assets/markdown-img-paste-20190823181703527.png)

### Internationalised Domain Names

Look at the example below , it shows how .gove is writtent in chinese , Google in Japanese and site in Arabic .


> ![](/assets/markdown-img-paste-2019082318205192.png)

Since DNS Native canot support such characters , enter the world of Punnycodes .

#### Punnycode

A punnycode is a mapping of the foreign language character to the DNS naming system. A punny code always starts with xn-- and the mapping is locally done from foreign language to punnycode in the browser before sending the query out to the DNS server.

> ![](/assets/markdown-img-paste-20190823182124419.png)

---

###  Name Servers , Zones and Authority

#### Name Server
A `name server` has the database of the names (DNS to IP Mappings).
Name servers load data in `zones` , the administrative units in DNS.

A name server may load thousands of zones , but anever loads the  entire namespace.

**<span style="color:blue">A name server that loads an `entire zone` is AUTHORITATIVE for that zone; which means this name server can answer any quesry about any domain name in the zone**

Se below on how a typical DNS query looks like .

1. Recursive query is sent by client for `linux1.cisco.com`
2. The Name server in the middle (the companies local name server) does not know the answer and reaches out to the `root` name server
3. The `root` name servers send a `REFFERAL` to the `com` server since it does not know the answer
4. The `com` server sends a `REFERRAL` to the `cisco` zone since it does not know the answer
5. `cisco` replies back with the answer.

Notice above the amound of the work the server in the middle has to do with processing the `REFFERALS` and **recursing** through the DNS tree. It means this server has **recursion enabled**.

The name servers in the middle accepted a `recursive` query from the resolver. Accepting a recursive query obliges the name server to do all the work and retun the name.

Most resolvers send recursive name queries as they are not smart engough to follow refferals.

Most name servers are smart enough to follow refferals and send non recursive queries by default. It gives the name server and option to reply back with refferal. Its beneficial to receive refferals as it contains valuable information.


![](/assets/markdown-img-paste-20190823200617518.png)


### DNS Query and Responses

DNS used UDP , the QUERY and RESPONSE are the same format.

![](/assets/markdown-img-paste-20190823201624968.png)

The HEADER and the QUESTION section are always present! Even on the RESPONSE packet.

Whether the other sections above are present are not are dependedent on the type of response , depending on whenter its and ANSWER or REFFERAL.

**HEADER contains**

- **OPCODE** : `QUERY` (its the same for QUERY and RESPONSE)
- **qr bit** : `0` for query ; `1` response
- **rd bit** : `1` for recursive and `0` non-recursive

**QUESTION contains**

 - **Domain Name** : Domain which we are interested in .
 - **Class** : `IN` (Almost always IN for internet)
 - **TYPE** : Type of data record the queries is interested in , like `MX` .

**ANSWER , AUTHORITY and ADITIONAL**

Just to carry data in form of Resorce Records

> **`TTL` is how long the record is cached in the name server before it needs to be asked again**

> **Server Selection algorithm** is how the name server to be queries is choosen. BIND ( a variant of name server) chooses based on the response time.

### Authoritative , Recursive and Caching Name Servers

As we understood each concepts above about Authority , Recursiveness and Caching ; lets look at the servers of these types.

**Authoritative Server** : A name server whose primary function is to answer **non-recursive queries for data in its authoritative zones**.

**Recursive Server** : A recursive name servers is the one who is willing to do all the work to resolve the  hostname doing multiple level of recursion. A recursive name server also caches the responses.

Authoritative name servers **come into two categories** :

- **Primary** : A primary reads the data from the local "Zone Data file" hosted on the server.
- **Secondary** : The Secondary takes the zone data information from the primary. This process is known as the "Zone Transfer"


![](/assets/markdown-img-paste-20190906062727758.png)

### Serial Numbers and Zone Data Transfer


![](/assets/markdown-img-paste-20190906064539206.png)

1. When the `SECONDARY` is first setup , it sends an AXFR request to the primary to copy the zone data file.
2. The PRIMARY sends the zone information to the secondary.
3. The Secondary preriodically sends the SOA query to the primary the response of which also contains the serial number for the zone data file.
4. If the serial number on the Primary is higher than the serial number on secondary , a zone transfer is intiated again with step 5. and 6.


### Forwarding and Forwarders

A forwarder does not forward queries.
A forwarder receives queries that are FORWARDED to it by other name server.

Its a way to build a rich in house name server. All the name servers internally in a company point to the Forwarder which in turn goes to the internet for queries. This also simplifies firewall tasks.

In the example below the Resolver sends the recursive query to the "name server" which forwards the query to the "forwarder" ; which in turn does the recursive name lookup.

![](/assets/markdown-img-paste-20190906065502266.png)

#### Resource Records

These are the units of data stored in the DNS namespace.


> ![](/assets/markdown-img-paste-2019090607014381.png)

- `TTL` : How long this entry should be cached
- `IN` : Internet
- `TYPE of record` : `A` or `MX` etc
- `RDATA` : The actual data for the entry , also the type of this data (IPv4 Address or Name) is dependent on the `TYPE`. For example for an `A` record it would be an IP Address.

#### The A Record and the AAAA Record

These are the IPv4 Address records.
IPv6 A record is called `AAAA` Record. It has 4 A's since its 4 times the 8bits IPv4 Address.

> If both A and AAAA records are available for a Host, some clients will prefer the IPv6 over IPv4 record ; and some will connect to both AND use the one which is faster. This algorithm is call the HAPPY EYEBALLS.

 #### PTR Records

 This is used for Reverse Lookups.

 Why do we need another record for IP to Name lookup , why cant we use the A / AAAA record for the same. Well we could but it does not allow exhaustive tiered lookup for the IP Address.

192.168.1.3 becomes 3.1.168.192.cisco.com

IPv4 and IPV6 PTR example

![](/assets/markdown-img-paste-20190906071553752.png)

 #### PTR Records

A CNAME Record is an alias in DNS. When a DNS Server does not have a A record for the query , it responds back with the CNAME record if it has it to the querier. THe queries then restarts the lookup process with this new name provide to with the CNAME field.

 #### MX Records

For email records.


![](/assets/markdown-img-paste-20190906073450522.png)

#### SRV Record (Service Record)

SRV, or service record. It's like a general purpose MX record that can work with any service. The syntax of the SRV record is a little more complicated than the MX record.


![](/assets/markdown-img-paste-20190906073720952.png)

- First field is the name of the service and the protocol it runs over.

#### TXT Record

The TXT, or text record, can be used to attach comments or almost anything else to domain names in the name space.


![](/assets/markdown-img-paste-20190906074017497.png)

#### Infrastructure Record

 **GOOD LINKS**
 - [X] https://engineering.purdue.edu/kak/compsec/NewLectures/Lecture17.pdf
 - [X] http://wiki.netkit.org/netkit-labs/netkit-labs_application-level/netkit-lab_dns/netkit-lab_dns.pdf

 - [X] https://seedsecuritylabs.org/Labs_16.04/PDF/DNS_Remote.pdf
 - [X] http://www.cis.syr.edu/~wedu/seed/Labs_12.04/Networking/DNS_Remote/DNS_Remote.pdf


# IPSEC VPN (from CCIE Notes)

> **Diffie Hellman** is the algorith that generates a `KEY` . Lifetime of a DH key is 3600 secs (1hr).

There are two tunnels :
1. `PHASE 1` The **first tunnel** is to exchange the KEY . `ISAKMP` Internet Security Association and Key Managment Protocol is used here .
2. `PHASE 2` The **second tunnel** is for Data transfer. `ESP` Encapsulation Security Payload is used in this phase.

<img src="/assets/markdown-img-paste-20180619143018242.png" alt="Drawing" style="width: 600px;"/>

> Though its not recommened , you can manually setup the `Phase 2` tunnel to use a manual key skipping the `Phase 1` negotiation (without using `ISAKMP`).


### MED Attribute (from BGP Technotes )

- `MED` (Metric) is advertised to your neighbor on how they should enter your AS.

- `Local Preference` is used for **outbound** traffic , `MED` is used for **inbound** traffic.

- MED is exchnaged between autonomous systems
- Lowest MED is the preffered.
- MED is propagated to all routers in the neighbor AS , but is not passed out to other AS


![](/assets/markdown-img-paste-2019081716470814.png)

```sh
!Set the MED
route-map MED permit 10
set metric 100
exit

!Configure it in BGP
router bgp 1
 neighbor 192.168.12.2 route-map MED out
```

# Physical Organisation

At a very high level AWS can be broken down into two main blocks

- **`AWS Regions`** : These are grouping of independenly separated datacenters in a specific geographic regions know as `Availabilty Zones`.

- **`AWS Edge Location`** : It is a Datacenter which does not contain any AWS Services ; Instead it is used to deliver contents to parts of the world. `CloudFront`

> Not All AWS Services are availaible globally , one of the example of a Global Service is the `IAM`



An `AWS Region` has multiple `Availabilty Zones`



Now within each `Availabilty Zone` there can be multiple `Datacenters`

`Availabilty Zones` are **phyically sperated** to each other but **have high speed connection between them** to provide **Fault Tolerance**

> So as an example `S3` is replicated accross all `Availabilty Zones` and all `Datacenters` for **reliability and high availaibility**.


**AWS has 19 Regions and 57 AZs within those 19 regions.**

![](/assets/markdown-img-paste-20190725061715368.png)


![](/assets/markdown-img-paste-20190725071537652.png)

-------

## AWS Region

>**AWS re:Invent 2014: AWS Innovation at Scale with James Hamilton**
>https://www.youtube.com/watch?v=JIQETrFC_SQ

Let look into one of the Regions: US East
Each region has at least 2 availaibility zones. Instead of a DC being a region, AWS has an AZ as a Region. There are two transit centers which connect to customers.

The AZs are quite apart (few miles). The speed between AZs are extremely high. 25Tbps.

> **Transit points allow conectivity with other Regions and Customer Peering Points.**

![](/assets/markdown-img-paste-20190725072213950.png)

![](/assets/markdown-img-paste-20190725073401480.png)

- If an AZ you can commit synchronously to your Data with a 1ms delay rather than 74ms delay (above) . This is empowering and great for the applications.
- Failover between AZs is **SUPERFAST** , compare this to a Intra Region failover which would be much slower.


Going further , lets look into Availability Zones

![](/assets/markdown-img-paste-20190725090416853.png)
![](/assets/markdown-img-paste-20190725090810986.png)
![](/assets/markdown-img-paste-20190725090954823.png)

---


![](/assets/markdown-img-paste-20190725094118435.png)


![](/assets/markdown-img-paste-20190725094202242.png)

> https://daviseford.com/blog/2018/12/21/aws-advanced-networking-specialty-exam-tips.html


Now as we further move ahead , the `VPC` :


![](/assets/markdown-img-paste-20190725103828635.png)

>  - **The VPC is a Logical Construct with little significance to actual networking.**
> - **A VPC is specific to a Region**
>  - **A Large IP Space is define while creating a VPC , but eventually smaller subnets are to be created specific to the AZs.**
>  - **A subnet spans only the AZ**
> - **When an Instance is created with a Subnet Specified -- The instance gets created in the same AZ as the subnet (Obviously :))**
> - **Dual Interfaces can reside in a single subnet only** (As per the lab)


> **Security Groups are State Full , once you allow a traffic inbount , there is no need to create an outbout rule for the same.**
> **NACLs are not statefull , both INBOUND and OUTBOUND rules need to be created**




<img src="/assets/markdown-img-paste-20180317161347849.png" alt="Drawing" style="width: 300px;"/>







<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
