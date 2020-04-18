---
layout: post
title: Cisco ISE Demo Script
description: My Notes preparing for CCIE Security v5
comments: true
---

# Cisco ISE Demo Script

The purpose of this blog entry is to to detail in the simplest wat key ISE functionalities and lay them out so that it could be easily reviewed/demoed.

We will cover the following in the order below. Please note that the some sections could have configuration dependencies from the sections before it.

> At some point I will upload the configs based on each sections. Please feel free to reach out to me meanwhile.


- ##### Profiling
- ##### Basic Wired DOT1.X Authentication (Allow access to deny completely)
- ##### Basic Wired DOT1.X Authentication with Change of VLAN/DACL
- ##### Posture


## Basic Discovery and Profiling Example with Switch sending Data to ISE

```sh
snmp-server community snmp_ro RO
snmp-server trap-source Vlan1
snmp-server source-interface informs vlan1
snmp-server enable traps snmp authentication linkdown linkup coldstart warmstart
snmp-server enable traps
snmp-server host 150.1.7.212 version 2c snmp_ro
```

> **NOTE** that 802.1X Works at the Layer 2 level and there is no IP communication at this level.
Good review of the process here : https://en.wikipedia.org/wiki/IEEE_802.1X


## Basic Wired DOT1.X  Authentication (Allow access to network or deny completely)

In this lab setup the switchport either authorsises the devices if the right credential is provided or completely denies access and the devices gets APIPA IP .


The `dot1.x` configuration flow consists of configuring three main sections.

1. The **Switch** to which the endpoint is connected : AAA and DOT1X related config.
2. The **ISE Server** with the details of the Switch and the end user
3. The **End Point** itself for `dot1.x` authentication


> In the topology below we will configure the **Switch** , **ISE** and the **Win** devices. Very basic connectivity is already setup as show int the topology.
![](/assets/markdown-img-paste-2018110420170354.png)

# Lets start with the Switch
---

**Different types of Host Authentication Modes**

**single-host**  - Exactly one MAC Address

**multi-host** -   One MAC Address opens the door , and rest (other VMs on the Host)  can get in easily without authentication.

**multi-domain** -  Has nothing to do with AD Domain , its about multiple VLANs like voice and data vlan.

**multi-auth**  -  Every single MAC Address has to be authenticated . Even the VMs has to be authenticated .



**The code below is commented and sequenced accordign the steps required.**


```sh
!Make sure a enable password is set
enable secret 5 $1$AGpH$kIw79LdzMFQ395d/


!Enable AAA system
aaa new-model


!Point to ISE
aaa group server radius ISE-group
 server name ISE
!
radius server ISE
 address ipv4 192.168.1.101 auth-port 1812 acct-port 1813
 key **sharedsecret_with_ISE**

!Configure shell login to use enable secret details
aaa authentication login default enable


! Use the Radius Authentication for dot1x
aaa authentication dot1x default group radius

!Authorization is for Dynamic VLANs and ACLs to be assigned
aaa authorization network default group radius

!Default method for account is  RADIUS
aaa accounting dot1x default start-stop group radius


!Include IP Address of the supplicant  IP Address of the suplicant as a part of the request.
radius-server attribute 8 include-in-access-req


!Globally enabling Dot1X Authentication
dot1x system-auth-control


!Default the port on which the endpoint is connected to reset config
default interface GigabitEthernet1/2


!Switchport configuration details
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
! Open mode for testing
 authentication open
! Authentication mode , see above for what each mode means
 authentication host-mode multi-auth
 authentication port-control auto
! Recurring authentication
 authentication periodic
! Let server decide on how often to re-athenticate
 authentication timer reauthenticate server
! Set port access entity as the  autheticator
 dot1x pae authenticator
! Supplicant retry timeout
 dot1x timeout tx-period 10

!Ensure the following for reachability from switch to ISE
interface Vlan1
 ip address 192.168.1.102 255.255.255.0
!

! Some default configs
aaa session-id common
!
```

**Copy Paste Snippet (Modify Here)**

```sh
hostname SW1
enable secret cisco123!
aaa new-model
!
aaa group server radius ISE-group
 server name ISE
!
radius server ISE
 address ipv4 150.1.7.212 auth-port 1812 acct-port 1813
 key cisco123!
!
aaa authentication login default enable
aaa authentication dot1x default group radius
aaa authorization network default group radius
aaa accounting dot1x default start-stop group radius
!
radius-server attribute 8 include-in-access-req
!
dot1x system-auth-control
!
default interface GigabitEthernetX/X
!
! # In this example we will notice , that if authentication does not
! # success , the port is not able to get into VLAN1 and hence no
! # IP Address is received .
! # show authentication port ..... ... ...
! #
! # Once the right password is supplied , the device gets into VLAN1
! # gets the  IP Address
! #
interface GigabitEthernetX/X
 switchport mode access
 switchport access vlan 1
 spanning-tree portfast
 authentication open
 authentication host-mode multi-auth
 authentication port-control auto
 authentication periodic
 authentication timer reauthenticate server
 dot1x pae authenticator
 dot1x timeout tx-period 10

! Make sure connectivity to ISE
interface Vlan1
 ip address 150.1.7.222 255.255.255.0
 no shut
!
!
aaa session-id common
!
```

**Finally**

```sh
! # Here we actually lock the port down. with "no auth open"
interface GigabitEthernet0/0
 no authentication open
 authentication host-mode single-host
```

# Lets configure ISE now
---

**OPTIONAL Set the ISE Password to be less restrictive**
![](/assets/markdown-img-paste-20181104205150657.png)

**Add the User "bob" with password "cisco123!"**
![](/assets/markdown-img-paste-20181104205619430.png)

**Add the Switch SW1 with RADIUS "cisco123!"**
This basically lest the switch to communication via RADIUS to ISE
![](/assets/markdown-img-paste-20181104205545490.png)

**Test Authentication**

```sh
debug aaa
test aaa group ISE-group bob cisco123! new-code
```

# Finally lets enable the PC to do DOT1.X
---

![](/assets/markdown-img-paste-20181104220926256.png)
![](/assets/markdown-img-paste-20181104221039835.png)

# **Troubleshooting Commands**

**PC Troubleshooting**

`services.msc --> Wired Auto Config --> Start`

**Switch Troubleshooting**
```sh
do debug radius authentication
show dot1x all

debug aaa
test aaa group ISE-group bob cisco123! new-code

show authentication sessions interface gi1/2

```


## Basic Wired DOT1.X  Authentication with Change of VLAN

> **No Additional switch configuration is required for the change of VLAN/DACL**


This builds upon the lab above. Instead of blatenlty denying network access to the user , we put in on a separate VLAN.

1. **Make sure the user is moved under a user group**
![](/assets/markdown-img-paste-20181105230403658.png)


2. **Create and Authorization Profile to set the VLAN to 30 or whatever**
![](/assets/markdown-img-paste-20181105230500743.png)

3. **Set the Policy set (Authorization Policy)**
![](/assets/markdown-img-paste-2018110523072321.png)


### **And that is it!**
Now let the user `bob` login and the switchport vlan should change to the desired VLAN.

## Configuring DACL is pretty much the same as the VLAN change above.

1. **Make the DACL**
![](/assets/markdown-img-paste-20181105232047938.png)
2. **Add the DACL to the Authorization to the Profiles**
![](/assets/markdown-img-paste-20181105231114865.png)
3. **Now re-authenticate the PC and you should see new ACLs on the switch (show ip access-list)**
> Notice the deny to 8.8.8.8
![](/assets/markdown-img-paste-20181105232236124.png)
```sh
Switch# sh authentication sessions interface gigabitEthernet 2/0/1 details

            Interface:  GigabitEthernet2/0/1
          MAC Address:  9457.a5b0.0ade
         IPv6 Address:  Unknown
         IPv4 Address:  Unknown
            User-Name:  94-57-A5-B0-0A-DE
               Status:  Authorized
               Domain:  DATA
       Oper host mode:  multi-auth
     Oper control dir:  both
      Session timeout:  N/A
    Common Session ID:  0A0000640000012B75977C34
      Acct Session ID:  0x00000039
               Handle:  0xE7000033
       Current Policy:  POLICY_Gi2/0/1

Local Policies:

        Service Template: DEFAULT_LINKSEC_POLICY_SHOULD_SECURE (priority 150)

Server Policies:

        ACS ACL:  xACSACLx-IP-PERMIT_ALL_TRAFFIC-57452910

Method status list:

       Method           State
       dot1x            Stopped
       mab              Authc Success

```

4. **Now check the ping front the PC and you should not be able to ping 8.8.8.8 anymore**
![](/assets/markdown-img-paste-20181105232306747.png)


# Lets enable SSH Auth via ISE for a IOS Router

```sh
ip domain-name cisco.com
crypto key generate rsa modulus 1024

aaa new-model
!
aaa authentication login NOAUTH  none
!
line con 0
 login authentication NOAUTH

!
radius server lab_ise
 address ipv4 X.X.X.X auth-port 1645 acct-port 1646
 key cisco123!
!
aaa group server radius ISE
 server name lab_ise
!
aaa authentication login FOR_SSH group ISE
aaa authorization exec FOR_SSH group ISE if-authenticated
!
line vty 0 4
 authorization exec FOR_SSH
 login authentication FOR_SSH
 transport input ssh
 session-timeout 1440
 exec-timeout 0
```


![](/assets/markdown-img-paste-20191010045756905.png)


![](/assets/markdown-img-paste-20191010045918416.png)

# POSTURE **

In this lab we will have 3 different compliance status configured on ISE

- Unknown
- Non-Compliant
- Compliant

In relation to above , we will have three DACLs

- DACL_UNKNOWN_POSTURE : DACL when the posture is not known.
- DACL_INTERNET_ONLY_NON_COMPLIANT_POSTURE : DACL when the end user denies to to meet the posture requirements and do the actions necesary for the same.
- DACL_COMPLIANT_POSTURE : This is when the the User is deemed compliant by ISE.

And corresponding three Authorization Profiles respective to above DALCs

- AUTH_PROF_UNKNOWN_POSTURE : DACL when the posture is not known.
- AUTH_PROF_INTERNET_ONLY_NON_COMPLIANT_POSTURE : DACL when the end user denies to to meet the posture requirements and do the actions necesary for the same.
- AUTH_PROF_COMPLIANT : This is when the the User is deemed compliant by ISE.


**We are going to use the Workcenter for Posture**


![](/assets/markdown-img-paste-20191014064401886.png)

**The workflow is to move from Left to Right**

![](/assets/markdown-img-paste-20191014064835940.png)

**As a Next Step we ensure that the Dot1X Configuration is done on the switch and is added to the ISE**


![](/assets/markdown-img-paste-2019101406504572.png)

For the client provisioning the workflow looks like this :


![](/assets/markdown-img-paste-20191014071001603.png)

**Next we create a Posture Profile (XML) file to be used by the Anyconnect client**

![](/assets/markdown-img-paste-20191014065600965.png)

In this section we define the characteristics of the client on what it should be doign based on the XML file:


![](/assets/markdown-img-paste-20191014065812760.png)

Server Name Rule is Mandatory (We keep it as * (All))

![](/assets/markdown-img-paste-2019101406590417.png)

Ensure that the compliance Module is present

![](/assets/markdown-img-paste-20191014070347884.png)

Now that the profile is configured and we configure the the actual anyconnect software package.

![](/assets/markdown-img-paste-20191014070113731.png)

![](/assets/markdown-img-paste-20191014070129570.png)


![](/assets/markdown-img-paste-20191014070738493.png)


![](/assets/markdown-img-paste-2019101407162502.png)

We leave the "Client Provisioning Portal" as default (Its a straight forward config)


![](/assets/markdown-img-paste-20191014071829379.png)

**FOR NOW WER ARE NOT DOING ANY POLICY ELEMENT /POSTURE CHECK AND ENSURE THAT THE CLIENT IS ABLE TO DOWNLOAD THE ANYCONNECT MODULE**

**NOW Leats create the Different DACLs we discussed above**

- **DACL_UNKNOWN_POSTURE**


```
!DHCP/DNS
permit udp any eq bootpc any eq bootps
permit udp any any eq 53
! The Only IP our client needs to talk to PSNs
permit ip any host ISE_SERVER_IP_ADDRESS_1
permit ip any host ISE_SERVER_IP_ADDRESS_2
deny ip any any
```

![](/assets/markdown-img-paste-20191014072535301.png)


- **DACL_INTERNET_ONLY_NON_COMPLIANT_POSTURE**

Denying all traffic to RFC 1918 Local address and only allowinf internet traffic.

```
!DHCP/DNS
permit udp any eq bootpc any eq bootps
permit udp any any eq 53
! The Only IP our client needs to talk to PSNs
permit ip any host ISE_SERVER_IP_ADDRESS_1
permit ip any host ISE_SERVER_IP_ADDRESS_2
! Deny Traffic to all local RFC1918 Networks (Allowing Internet Only)
deny ip any any 10.0.0.0 0.255.255.255
deny ip any 172.16.0.0 0.255.255.255
deny ip any 192.168.0.0 0.0.255.255
permit ip any any
```

![](/assets/markdown-img-paste-20191014073235916.png)


- DACL_COMPLIANT

Allow all traffic

```
permit udp any eq bootpc any eq bootps
permit udp any any eq 53
permit ip any host 150.1.7.212
permit ip any any
permit tcp any any
permit icmp any any
```

![](/assets/markdown-img-paste-20191014073526505.png)

**AN ACL On the switch is required which will redirect the web traffic to the posture page**

```sh
!ACL_REDIRECT
! # Will no redirect DNS/DHCP or ISE Connections
deny udp any eq bootpc and eq bootps
deny udp any any eq domain
deny ip any host 150.1.7.212
! # But will redirect HTTP/HTTP Traffic
permit tcp any any eq www
permit tcp any any eq 443
```

**Now Create the Authorization Profiles**

- AUTH_PROF_UNKNOWN_POSTURE


![](/assets/markdown-img-paste-20191014075624886.png)
![](/assets/markdown-img-paste-20191014075637414.png)

- AUTH_PROF_INTERNET_ONLY_NON_COMPLIANT_POSTURE

![](/assets/markdown-img-paste-20191014075807246.png)

- AUTH_PROF_COMPLIANT


![](/assets/markdown-img-paste-20191014075904713.png)


**FOR NOW WE WILL SKIPP TO THE POSTURE POLICY AND MOVE TO POLICY SETS TO ENURE OUR CONFIG SO FAR IS WORKKIGN**


![](/assets/markdown-img-paste-20191014080337817.png)


**Now Create the Respective Authorization Rules**

- AUTHZ_RULE_USER_UNKNOWN

![](/assets/markdown-img-paste-20191014081607240.png)

- AUTHZ_RULE_INTERNET_ONLY
- AUTHZ_RULE_COMPLIANT


![](/assets/markdown-img-paste-20191014082200779.png)


1.
ip access-list extended ISE-POSTURE-REDIRECT
 deny   icmp any any
 deny   udp any any eq domain
 deny   udp any eq bootpc any eq bootps
 remark ISE
 deny   ip any host 10.104.99.103
 deny   ip any host 10.103.2.105
 deny tcp any any eq 8905
 remark PROXY
 deny    ip any host 10.104.164.235
  deny    ip any host 10.104.164.236
 permit tcp any any eq 443
permit tcp any any eq 8080
permit tcp any any eq www
 deny ip any any

DACL
permit ip any host 10.104.99.103
permit ip any host 10.104.99.204
permit ip any host 10.104.164.203
permit ip any host 10.104.164.102
permit ip any host 10.103.2.105
permit icmp any any
permit udp any any eq domain
deny ip any any


ip http port 8080

·         ip port-map http port 8080




ip access-list extended ACL-POSTURE-REDIRECT
 deny   ip any host 10.24.218.254
 deny   ip any 10.75.250.228 0.0.0.3
 deny   ip any host 194.3.27.88
 deny   ip any host 13.107.4.52
 deny   ip any host 104.86.189.82
 deny   ip any host 88.221.113.75
 deny   ip any host 23.55.38.81
 deny   ip any host 23.206.242.17
 permit tcp any any eq www
 permit tcp any any eq 443
