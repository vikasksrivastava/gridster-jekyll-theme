---
layout: post
title: First 7 Days ACI
description: First 7 Days ACI
comments: true
---

# Notes on "Your First Seven Days Of ACI" - BRKACI-1001 Ciscolive! Session


---

# Day 1 - Why ACI
---

### ACI Advantages

 ![](/assets/markdown-img-paste-20200126092949307.png)

>**ACI solves the above challenges by providing a simple Leaf/Spine Topology, ECMP which removes the dependency on STP and so on an so forth .**

![](/assets/markdown-img-paste-20200126093011812.png)

---

### ACI Fabric Discovery Basics

![](/assets/markdown-img-paste-20200126093627913.png)

All the internal communication betweek the Spines,Leafs and the APIC happens on the Infra IP Address denoted by red T in the picture above. The reachability provided by this Infra Network is then used to deploy the required L2/L2 config wherever needed on the leafs.


![](/assets/markdown-img-paste-20200126094243775.png)

`ISIS` : This enables IP reachability between TEPs , the APIC assigns the TEP address. ISIS is automatically established and requires no configuration.

`MP-BGP` : This is L3 Out Configuration . The routes learned via the WAN (L3 Out in the above pic) needs to be "reflected" / "learnt" by other Leafs. Hence the MP-BGP config.  Note that the no manual config is required except for the the assignment of RR (Route Reflector)

Using the above two components we build the `Underlay Network` which builds as the foundation for the overlay network.


---

### APIC Controlled Basics

**Basic Details about the Ports on the APIC and the the Leafs/Spine**

![](/assets/markdown-img-paste-20200126094750556.png)

- The Blue cables connect to the Leafs
- The Red cables is the CIMC connection (MGMT)
- The Green Ones are the interfaces on which the Managment Interface of the APIC resides.

> **Irrespective of what APIC IP Address you connect to via HTTP you see the same data**

> Its a good idea to ensure all the controller status are in healthy status while troubleshooting an APIC issue ![](/assets/markdown-img-paste-20200203185456630.png)

---

# Day 2 - Infrastructure and Policies

---

### Managment Access to Switches

![](/assets/markdown-img-paste-20200126094915966.png)


![](/assets/markdown-img-paste-20200203185939462.png)

You can access the the Leafs and the Spines using their Console ports on via the APIC (which in turn connects via VTEP addreses). BUT it is advisable to have individual Management IP Addreses assigned to these devices directly (config done after discovery) , so that inc ase we need to access them.


**Each device needs their individual mgmt IP (Oulined above) rechability for AAA and NTP**

![](/assets/markdown-img-paste-20200126095656484.png)

---

### ACI Backups

**There are two ways to do backups**

- Full Backup
- Snapshots

**Capability to compare configuration (Snapshots)**

![](/assets/markdown-img-paste-20200126101615741.png)

![](/assets/markdown-img-paste-20200126101638584.png)

**Full Backup**

Note in the picture below that when , this setting is disabled during the backup ; the passwords stored for VMWare Integration or any third part integration are NOT exported. If its enabled , the password is exported in an encrypted fashion.

![](/assets/markdown-img-paste-20200203190922958.png)

---

### What's unders the Fabric Tab - This is where you build the UNDERLAY
##### Under Fabric we have Fabric Policies and Access Policies

- #### **Fabric Policies**

  Fabric policies govern the operation of internal fabric interfaces.
**<span style="color:blue"> Fabric policies configure interfaces that connect spine and leaf switches.</span>** Fabric policies can enable features such as monitoring (statistics collection and statistics export), troubleshooting (on-demand diagnostics and SPAN), or NTP.

  ![](/assets/markdown-img-paste-20200204132053437.png)


- #### **Access Policies**

  Access policies govern the operation of interfaces that provide external access to the fabric. **<span style="color:blue">Access policies configure external-facing interfaces that do not connect to a spine switch.</span>** External-facing interfaces connect to external devices such as virtual machine controllers and hypervisors, hosts, routers, or fabric extenders (FEX).

  ![](/assets/markdown-img-paste-20200204132444173.png)

---

### Whats unders the Tenants Tab - This is where you build the OVERLAY

- #### Tenant Policies
  `Tenant Policies` : Its is more about configuration related to EPG/BD/VRF.
  A tenant is a logical container or a folder for application policies. This container can represent an actual tenant, an organization, or a domain or can just be used for the convenience of organizing information.

  ![](/assets/markdown-img-paste-20200204133750270.png)

**We have three types of configuration in Tenant Policies**

![](/assets/markdown-img-paste-20200204134548149.png)


- **Static Binding**

   ![](/assets/markdown-img-paste-20200204135424679.png)


   ![](/assets/markdown-img-paste-2020020413575866.png)
   `Policy Control Enfrocement` defines if the any contracts policies will be enforced or not. `Policy Control Direction` defines which direction it is applied.

   **An Example of a Static Path Binding Config**
   ![](/assets/markdown-img-paste-20200204154603949.png)
   With the above binding configured , you can extend your legacy VLAN configuration to ACI and vice versa

   ![](/assets/markdown-img-paste-20200204154924457.png)
   *In Picture above VLAN from N7K is extended to ACI*

   <br><br>

- **Cisco ACI VMWare Integration**

   ![](/assets/markdown-img-paste-20200204175320396.png)
   - **Step 1.**  Define the vCenter Domain We are going to talk to .
   - **Step 2.**  ACI and VMWare handshake (the communication between APIC and VMWare happens on the Out of Band Network NOT on the INFRA network; We can use the Inband network but is NOT recommened.)
   - **Step 3.** vCenter goes ahead and created a vDS on the VMWare
   - **Step 4.** VMWare Admins Associated the ESXi to the vDS created above
   - **Step 5.** Between the vDS and ACI ; `LLDP` happens , this will tell what VMWare Blade is on what Leaf.
   - **Step 6.** Associate the EPGs to the VMMDomain
   - **Step 7. and Step 8** The EPGs are auto mapped to VMWare Port Groups and are created in VMWare
   - **Step 9.** Now since ACI knows about the Server location based the information learned from LLDP , ACI and now push the policy on the exact switch where needed (Instead of pushing it everywhere).
   - **So the network path is established above for the VM to ACI**


---

# Day 3 - Forwarding Overview

---

## Endpoint

**What is an endpoint ?**

It's a combination of MAC address and IP Address.

We can see it on the APIC
![](/assets/markdown-img-paste-2020012611072006.png)

We can find more details about the same on the specific Leaf


![](/assets/markdown-img-paste-20200126110818755.png)


MAC and /32 IP Address are stored in the `Endpoint Table`
Exception is L3 Out , If we use the same mechanism of learnign all the IP Address , on the MAC address on Nexus router we would have thousand of /32.  Other IP Information is used int he Ip Table just as the normal routing. That is the reason why we use arp table for L3 out.

**ACI Leafs Can Learn via ARP via 2 Methods**

ACI Learns the :

- Source MAC and Source IP during ARP
- A routed frame triggers a Source IP and MAC Address Learning


**Pervasive Gateway**

Pervasive gateway means a local gateway residing on every switch for each subnet on that switch.

![](/assets/markdown-img-paste-20200126174516134.png)

**Proxy Routing**

Every endpoint learn by a Leaf is informed to every spine using multicast. This way every SPINE knows about every endpoint in the network.

When Leafs do not know a path to a remote endpoint , they can query the Spine for the same.

![](/assets/markdown-img-paste-20200126174645703.png)

---







---

# Day 4 - Network Centric Migrations

---


---

# Day 5 - Multi Location Deployments

---

---

# Day 6 - Troubleshooting Tools

---


---

# Day 7 - Additional Resources

---

### Digital Learning Topologies

**Topology**

![](/assets/markdown-img-paste-20200205185831244.png)

**Out of Band Management Access**

![](/assets/markdown-img-paste-20200205185901826.png)

---

# Lab 1. Explore the Cisco ACI Fabric Inventory
##### Digital Learning
---



![](/assets/markdown-img-paste-20200207071921760.png)

`Fabric`: Cisco ACI inventory and configuration point for intra-fabric and access policies

`Virtual Networking`: Configuration menu for VM Manager interoperability, such as vCenter, Hyper-V, or KVM

`L4-L7 Services`: Package repository for upper-layer service elements, such as firewalls or load balancers, that can be inserted into the fabric

`Admin`: Menu for controlling the operation, administration, and maintenance (OAM) aspects

`Operations`: Menu for visibility, troubleshooting, and capacity profiling

`Apps`: App center used for deploying applications in the Cisco ACI

> The Cisco ACI solution uses an overlay, based on VXLAN, to virtualize the physical infrastructure. This overlay, like most overlays, requires the data path at the edge of the network to map from the tenant end-point address in the packet, also known as its identifier, to the location of the endpoint, also known as its locator. This mapping occurs in a function called a tunnel endpoint (TEP), also known as VXLAN tunnel end point (VTEP). The VTEP addresses are displayed in the INFRASTRUCTURE IP column. The TEP address pool 10.0.0.0/16 has been configured on the Cisco APIC using the initial setup dialog. The APIC assigns the TEP addresses to the fabric switches via DHCP, so the infrastructure IP addresses in your fabric will be different from the figure.


Clickign on the Node provides information on the neigbor and port connectivity


![](/assets/markdown-img-paste-20200205182333520.png)

***The Link Layer Discovery Protocol (LLDP) is responsible for discovering directly adjacent neighbors. When run between the Cisco APIC and a leaf switch, it precedes three other processes: Tunnel endpoint (TEP) IP address assignment, node software upgrade (if necessary), and the intra-fabric messaging (IFM) process, which is used by the Cisco APIC to push policy to the leaves.***


**This is where you can see the switch level details**
![](/assets/markdown-img-paste-20200205182608720.png)

**Interface Level Details**

![](/assets/markdown-img-paste-20200205182734834.png)



![](/assets/markdown-img-paste-20200205182757602.png)


**ACI Diag is the Diagnostic Command for the APIC**

```sh
apic1# acidiag -h
usage: acidiag [-h] [-v]
{avread,fnvread,fnvreadex,rvread,rvreadle,crashsuspecttracker,bootother,bootcurr,journal,logs,telemetry,hwcheck,dbgtoken,validateimage,validateng inxconf,version,preservelogs,platform,verifyapic,bond0test,linkflap,touch,run,installer,start,stop,restart,dmestack,dmecore,reboot}
               ...
positional arguments:
{avread,fnvread,fnvreadex,rvread,rvreadle,crashsuspecttracker,bootother,bootcurr,journal,logs,telemetry,hwcheck,dbgtoken,validateimage,validatenginxconf,version,preservelogs,platform,verifyapic,bond0test,linkflap,touch,run,installer,start          ,stop,restart,dmestack,dmecore,reboot}
                        sub-command help
    avread              read appliance vector
    fnvread             read fabric node vector
    fnvreadex           read fabric node vector (extended mode)
    rvread              read replica vector
    rvreadle            read replica leader summary
    crashsuspecttracker
                        read crash suspect tracker state
    bootother           on next boot, boot other Linux Partition, and display
                        updated /etc/grub.conf
    bootcurr            on next boot, boot current Linux Partition, and
                        display updated /etc/grub.conf
    journal             Contents of journal logs
    logs                show log history
    telemetry           enable/disable telemetry
    hwcheck             Quick check of APIC Hardware
    dbgtoken            show debug token
    validateimage       validate image
    validatenginxconf   validate nginx conf
    version             show ISO version
    preservelogs        stash away logs in preparation for hard reboot
    platform            show platform
    verifyapic          run apic installation verify command
    bond0test           ==SUPPRESS==
    linkflap            flap a link
    touch               touch special files
    run                 run specific commands and capture output
    installer           installer
    start               start a service
    stop                stop a service
    restart             restart a service
    reboot              reboot

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         verbose

```

```

apic1# acidiag fnvread
ID   Pod ID   Name     Serial Number  IP Address      Role   State   LastUpdMsgId
----------------------------------------------------------------------------------
101    1      leaf-a   FDO21351F7L   10.0.128.66/32   leaf   active   0
102    1      leaf-b   FDO21351F9A   10.0.128.64/32   leaf   active   0
201    1      spine    FDO214111Q5   10.0.128.65/32   spine  active   0

Total 3 nodes

```

 Cisco APIC Version (3.1) allows you to configure the system and view the configuration through the CLI.

```
apic1# conf t
apic1(config)# sh run
# Command: show running-config
  aaa banner 'Application Policy Infrastructure Controller'
  aaa authentication login console
    exit
<... output omitted ...>
```


**Static Out of Band management Address**

*Notice that you assign a subnet , and IP Addresses are choosen from that block**

![](/assets/markdown-img-paste-20200205183537995.png)

Result of Above configuration


![](/assets/markdown-img-paste-20200205183647111.png)


To assign to the ONLY ONE  devide put the same number twice


![](/assets/markdown-img-paste-20200205183752216.png)


```
leaf-a# show lldp neighbors
Capability codes:
  (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
  (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID            Local Intf      Hold-time  Capability  Port ID
3560-x.dc.local       Eth1/1          120        BR          Gi1/0/3
apic1                 Eth1/2          120                    eth2-1
spine                 Eth1/49         120        BR          Eth1/1
Total entries displayed: 3
```

**Removing OOB Management Address Does not affect the capabilit to SSH from APIC to Leaf as this happens with the inra network which ACI had seatup**

![](/assets/markdown-img-paste-20200205184419109.png)



![](/assets/markdown-img-paste-20200205184714373.png)


This is where you can validate the OOB Management address of a specific Leaf/Spine

![](/assets/markdown-img-paste-2020020518531889.png)

---

# Lab 2. Configuring Port Channel
---


In this Lab2 we wil configure the following scenario


![](/assets/markdown-img-paste-20200207055238481.png)

The following components are to be configured (show in <span style="color:purple">purple</span>)


![](/assets/markdown-img-paste-20200207055302373.png)

**High Level Steps**


![](/assets/markdown-img-paste-20200207065148867.png)


- ##### Configuring the Port Character

  1. Enable the CDP in a CDP interface policy that is named `CDP-Enabled`.
  2. Configure an interface policy (`LLDP-Disabled`), with LLDP **disabled**.
  3. Configure a **port channel policy**, named `PC-Policy`, with a static port channel mode. Ensure that you have `Static Channel—Mode On`

   > Notice above that you have separate config item named "Port Channel Policy" in ACI

  4. **Configure vPC Interface Policy Group** - An interface policy group gathers multiple interface policies into one set. **A vPC interface policy group gathers the policies with the purpose of activating them on a vPC interface bundle.**  In your topology, the hypervisor is connected to the leaves in a redundant fashion that allows a vPC deployment. You will configure a vPC policy group for your hypervisor. **Name is `IPG-VPC-ESX`** and add everything you created above within it.

  > What's happenign int he above step is that you are defining the characteristics of what a vPC member port would look like

  5. Create a Leaf Interface Profile name `InterProf-ESX`
  6. Select the Interfaces as `1/3`
  7. Select the `Interface Policy Group` created above int he `Port Charachter` section : `IPG-VPC-ESX`

- ##### Configuring the Switch Character


  1. Create a Leaf Profile - `LeafProfile-ESX` , add both-leaves 101 and 102
  2. Associate the Interface Selector `Interface-Profile-ESX`
  This is the actual step where the `Interface Profile` and `Switch Profile` connect.

  ![](/assets/markdown-img-paste-20200207065238851.png)

- ##### Finally Configure vPC between Leafs

  1. Review/Look at the VPC Default Domain
  `Fabric > Access Policies > Switch Policies > Policies > VPC Domain.`
  2. Add both leaf switches to the vPC security policy
  `Fabric > Access Policies > Switch Policies > Policies > Virtual Port Channel default.`
  3. Configure a VPC Protection Group table with these settings and click Submit.

    ```
      Name: ACI
      ID: 100
      vPC domain policy: default
      Switch 1: leaf-a (switch ID 101)
      Switch 2: leaf-b (switch ID 102)
    ```


- #### Verification

After the above configuration you will see

**show vpc status show the Peer Link Connectivt and ALso list the Port Channels that are configured on the Leafs**

```
leaf-a# show vpc
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 100
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : Disabled
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : primary
Number of vPCs configured         : 1
Peer Gateway                      : Disabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Enabled (timeout = 240 seconds)
Operational Layer3 Peer           : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id   Port   Status Active vlans
--   ----   ------ --------------------------------------------------
1           up     -

vPC status
----------------------------------------------------------------------
id   Port   Status Consistency Reason                     Active vlans
--   ----   ------ ----------- ------                     ------------
343  Po5    up     success     success
```

```
leaf-a# show vpc role

vPC Role status
----------------------------------------------------
vPC role                        : primary
Dual Active Detection Status    : 0
vPC system-mac                  : 00:23:04:ee:be:64
vPC system-priority             : 32667
vPC local system-mac            : 70:7d:b9:f3:f1:c5
vPC local role-priority         : 101
```


**Notice that in the above configuration the Port Channel Number is NOT the same , one 56 and other is 58 for the SAME port channel configured towards a single end host**

![](/assets/markdown-img-paste-20200207070854660.png)

**Note once the vPC config above is complete you can verify your config in the UI**


![](/assets/markdown-img-paste-20200207071531977.png)

---

# Lab 3. Configure Cisco ACI Logical Constructs
---



![](/assets/markdown-img-paste-20200208144443276.png)


**High Level Steps**


  1. Create a Tenant `Sales` - skip the step to create the VRF.
  2. Create a VRF named `Pre-Sales` under `Sales` tenant - Without creatign a `Bridge Domain` in the same step.
**Each tenant can have one or more VRFs, or share one default VRF with other tenants when there is no overlapping IP addressing being used in the ACI fabric.**
  3. Create a bridge domain, named `Presales-BD`, associated with the Presales VRF inside the `Sales` tenant.

---

**A bridge domain is a unique Layer 2 forwarding domain that contains one or more subnets. Each bridge domain must be linked to a VRF.**
**By default, unicast routing is enabled. ARP flooding is disabled so that unicast routing will be performed on the target IP address. Endpoint dataplane learning controls whether the remote leaf switch should update the IP-to-VTEP information with the source VTEP of traffic coming from this bridge domain.**

**Note: First-hop security and other policies are enabled on a per tenant bridge domain basis. As the bridge domain may be deployed on a single or across multiple leaf switches, the first-hop security threat control and mitigation mechanisms cater to a single switch and multiple switch scenarios.** ![](/assets/markdown-img-paste-2020020815035462.png)

---


  4. Configure four subnets for the Presales-BD with these default gateways: `10.0.1.254/24`, `10.0.2.254/24`, `10.0.3.254/24,` and `10.0.4.254/24`.![](/assets/aci-advertise-external-internal.gif)
  5. Create Application Profile `Tiered-App` and create EPGs `Web` , `App` , `DB`


  6. Create another filter named `HTTP` add `HTTP` under it.
  6. Configre a filter names `Basic-Ping-SSH` and under it create `ICMP` , `SSH` ![](/assets/markdown-img-paste-20200208163905538.png)


  7. Create a Contract `Web-Access` and include the filter `Basic-Ping-SSH` and `HTTP`



![](/assets/markdown-img-paste-20200208155300693.png)



![](/assets/markdown-img-paste-20200208161253674.png)


  8. Finally do the following



![](/assets/markdown-img-paste-20200208164245784.png)



---

# Lab 4. Configure VMM Domain Integration
---

#### PICTURE DRAWING HERE


**Cisco ACI supports three integration methods with VMware vCenter:**

- Distributed Virtual Switch (DVS)

- Cisco Application Virtual Switch (Cisco AVS)

- Cisco ACI Virtual Edge (AVE)


1. Create a `vCenter VMM Domain` in ACI and also configure the Dynamic VLAN Range .

![](/assets/markdown-img-paste-20200208194224127.png)

2. Provide the vCenter Credentials and IP Address of vCenter
3. The steps provisions VMWare DVS named `Sales-vCetner` in the VMWare environment

![](/assets/markdown-img-paste-20200208194932523.png)

4. Create an `AAEP`  name `vCenter AAEP` and join the `vCenter VMM Domain` and the `ESX` Policy Group created in `Lab 2`

5. Now go to the `Interface Policy Group` `ESX` and attach the `vCetner AAEP` to it as well.

6. Add  ESXi Host 192.168.10.62 to the DVS Switchs and assign the uplinks

```
For this lab

vmnic2 --- uplink1
vmnic3 --- uplink2

```


7. Ensure CDP is on in vCetner side ![](/assets/markdown-img-paste-20200208200432132.png)


8. Verify the Leaf is discovered ![](/assets/markdown-img-paste-20200208200603747.png)


9. Now go the APIC and look for the Uplink `vmnics` there ans see the discovered peers (leafs) ![](/assets/markdown-img-paste-20200208200746845.png)

10. Now associate the `Application Profile`s `Domain (VMs and BareMetals)` to the VMM Domain created.


**This will create the EPGs under this Application Profile as port groups in VMWare!** ![](/assets/markdown-img-paste-20200208201143960.png)

**Now these Port Groups can be assigned to different VMs which in ACI map to different EPGs.**

---

# Lab 5. Deploy Cisco ACI Virtual Edge and Microsegmentation
---



---

# Lab 6. Integrate Cisco ASAv with Cisco APIC
---

Cisco Application-Centric Infrastructure (ACI) **provides the capability to insert Layer 4 through Layer 7 functions using an approach that is called a service graph. It can be considered as a superset of service insertion.** Meaningful Layer 4 through Layer 7 services can include firewalls, load balancing, SSL offloading, and application acceleration.


In this activity you will deploy ASAv as a  service in the Cisco ACI. The traffic between the DB and BACKUP EPGs will go through the ASAv. You will implement two types of operations:

-  managed and
-  unmanaged mode

and identify the differences between the two.

**High Level Steps**


![](/assets/markdown-img-paste-20200211195737484.png)

- **Step 1.** **LOAD the device package** A device package contains the mappings and abstrations of the object model of the device that is being controlled and the scripts to configure the device.
**`L4–L7 Services > Packages > Quick Start > Import Device Package` (asa-device-pkg-1.3.10.24.zip)**![](/assets/markdown-img-paste-20200211194054185.png)


- **Step 2.** **REVIEW the details of the package** You should see several web policy profiles, for **routed and transparent mode**. All of them initially permit web traffic, with different combinations of IP version and NAT.![](/assets/markdown-img-paste-20200211195534165.png)

> **EXAMINE** **WebPolicyForRoutedMode**, the first profile in the list. This section is the configuration placeholder for a Cisco ASA deployment in the Layer 3 mode. Among others, you will see the configuration of an ACL and the external and internal interfaces.

> **NOTE: If you do not customize these settings**, the firewall will be configured with a single ACL (access-list-inbound) to permit HTTP and HTTPS.

- **Step 3.** **Create TENANT Networking**
  - Create `Backup-BD` with the `Presales` VRF**
  - Create a **new EPG** (`Backup`), associate it with the new **bridge domain** (`Backup-BD`) and check the option `Associate to VM Domain Profiles` AND Associate it with your `VMM domain (Sales-vCenter)`

- **Step 4.** **Reuse the TRANSACT VM as a BACKUP VM** Delete the `10.0.4.254/24` subnet from the `Presales-BD` bridge domain and configure it for the `Backup-BD` bridge domain. assign the `TRANSACT VM` to the `Backup port group`.

- **Step 6.**  **Verify OOB Management to ASAv** All communication between the Cisco APIC and the Cisco ASAv occurs via HTTPS. The Cisco ASAv has been pre-provisioned with the necessary commands to enable HTTPS/SSH management access.

- **Step 7.** **Notice** all the ports of the ASAv in VSphere are assigned to the default `VM Network` . Once L4-L7 is complete you will see the ports change.

- **Step 8.** **Login to ASAv and verify basic configs AND HTTPs access**

  ```sh

  interface Management0/0
   nameif management
   security-level 100
   ip address 192.168.10.71 255.255.255.0

   route management 0.0.0.0 0.0.0.0 192.168.10.254 1

   aaa authentication http console LOCAL
   aaa authentication ssh console LOCAL
   aaa authentication enable console LOCAL
   aaa authorization exec LOCAL auto-enable

   http server enable
   http 0.0.0.0 0.0.0.0 management
   ssh 0.0.0.0 0.0.0.0 management

   username admin password CsI1KX6iq7UBl3KK privilege 15 -- Important

  ```

- **Step 9.** **Create a L4–L7 Device for Cisco ASAv**

  ![](/assets/markdown-img-paste-20200211202214483.png)

  ![](/assets/markdown-img-paste-20200211202253156.png)


- **Step 10.** **Create and Apply Service Graph**

  ![](/assets/markdown-img-paste-20200211202446497.png)

  ![](/assets/markdown-img-paste-2020021120250574.png)

  **Check the Above Config Here**
  ![](/assets/markdown-img-paste-20200211202603880.png)


> **NOTICE** That nothing has yet been deployed on the ASAv

- **Step 9.** **Apply the service graph template to the DB-to-Backup traffic**


  ![](/assets/markdown-img-paste-20200211202825309.png)

  ![](/assets/markdown-img-paste-20200211202843192.png)

  ![](/assets/markdown-img-paste-20200211204719420.png)


  ![](/assets/markdown-img-paste-20200211204817979.png)

  **Double-click the network_ip_address field of the network object web_server and enter the IP address of the TRANSACT VM (10.0.4.1/32).**
  ![](/assets/markdown-img-paste-2020021120484281.png)



- **Step 9.** **Check ASA Config**

  ```sh
  ciscoasa# show running-config access-list
  access-list access-list-inbound extended permit tcp any object web_server eq www
  access-list access-list-inbound extended permit tcp any object web_server eq https

  ciscoasa# show running-config access-group
  access-group access-list-inbound in interface externalIf

  ciscoasa# show running-config object network
  object network web_server
  subnet 10.0.4.1 255.255.255.255

  ciscoasa# show running-config interface GigabitEthernet 0/0
  !
  interface GigabitEthernet0/0
   nameif internalIf
   security-level 100
   no ip address

  ciscoasa# show running-config interface GigabitEthernet 0/1
  !
  interface GigabitEthernet0/1
   nameif externalIf
   security-level 50
   no ip address

  ```

- **Step 9.** **Verify Contract is applied**

  ![](/assets/markdown-img-paste-20200211205152138.png)

  ![](/assets/markdown-img-paste-20200211205218220.png)

**Also lok at the Subject**

  ![](/assets/markdown-img-paste-20200211205253392.png)

- **Step 9.** **Look at the Topology**

  ![](/assets/markdown-img-paste-2020021120532213.png)

- **Step 9.** **Examine the L4–L7 service parameters of the provider EPG (Backup).**

  ![](/assets/markdown-img-paste-20200211205356406.png)

- **Step 10.** **Examine the deployed graph instance and device.**


  ![](/assets/markdown-img-paste-20200211214532111.png)




- **Step 9.** **Go to Services > L4–L7, expand Deployed Devices, and examine the ASA-Presales device. You should see the VLAN encapsulations for the cluster devices.** ![](/assets/markdown-img-paste-20200211214701944.png)

- **Step 9.** **Notice the VM Settings**

  ![](/assets/markdown-img-paste-20200211214758213.png)


- **Step 9.** **Notice the Roll Back Options** ![](/assets/markdown-img-paste-20200211214928217.png)

- **Step 9.** **Customize the Cisco ASAv Configuration** You can assign ASAv interface IP Addresses right from the ACI.


  ![](/assets/markdown-img-paste-20200211215255628.png)

  ![](/assets/markdown-img-paste-20200211215316110.png)

  ![](/assets/markdown-img-paste-20200211215335358.png)

  ```sh
  ciscoasa# show interface ip brief
  Interface                  IP-Address      OK? Method Status                Protocol
  GigabitEthernet0/0         10.0.4.253      YES manual up                    up
  GigabitEthernet0/1         10.0.3.253      YES manual up
  ```

**You can verify the connectivity in the ASA**

  ```sh

  ciscoasa# show access-list
  access-list cached ACL log flows: total 0, denied 0 (deny-flow-max 4096)
              alert-interval 300
  access-list access-list-inbound; 2 elements; name hash: 0xcb5bd6c7
  access-list access-list-inbound line 1 extended permit tcp any object web_server eq www (hitcnt=3) 0x1560fdca
    access-list access-list-inbound line 1 extended permit tcp any host 10.0.4.1 eq www (hitcnt=3) 0x1560fdca
  access-list access-list-inbound line 2 extended permit tcp any object web_server eq https (hitcnt=0) 0x122c48e4
    access-list access-list-inbound line 2 extended permit tcp any host 10.0.4.1 eq https (hitcnt=0) 0x122c48e4

  ```



---

# Use L4–L7 Device in Unmanaged Mode

- **Step 1.** **Exami**

- **Step 2.** **Exami**

- **Step 3.** **Exami**

- **Step 4.** **Exami**

- **Step 5.** **Exami**

- **Step 6.** **Exami**


---




**ACI Operations and Troubleshooting (CI-ACIOPS)**

```
Outline: ACI Operations and Troubleshooting (ACIOPS)
Course Introduction

Overview
Course Goal and Objectives
Prerequisites
Course Outline
Module 1: Cisco ACI Component Review

Lesson 1: Cisco ACI Architecture and Network Review
ACI and APIC Review
Tenant, Management Tenant and Context Review
VLAN and VRF Review
Bridge Domain Review
Application Profile Review
End Point Group Review
Lesson 2: Cisco ACI Policy Review
ACI Contract Review
ACI Filter Review
ACI Subject Review
Lesson 3: Cisco ACI Unified Fabric Review
Cisco ACI Server Connectivity Review
Cisco ACI Network Connectivity Review
Cisco ACI Switch Review
Cisco ACI Routing Review
Lesson 4: Cisco ACI External Connectivity Review
Layer 2 External Switching Review
Layer 3 External Routing Protocol Review
Lesson 5: Customer Setup Review
Scaling Cisco ACI
Migration Paths to Cisco ACI
Module 1 Lab Exercises
Lab 0 – Lab Topology and Access
Lab 1 - Reviewing the Cisco ACI Environment
Lab 2 – Tenant and Context
Lab 3 - VLAN and VRFs
Lab 4 – Bridge Domains
Lab 5 – Application Profiles
Lab 6 - End Point Groups
Lab 7 – External Bridged Networks
Lab 8 – External Routed Networks
Lab 9 – Physical and VMM Domains
Module 2: Cisco ACI Operations

Lesson 1: The Cisco ACI Control Plane and Packet Flow
Determine ACI Packet Flow
Single DB/Single EPG with Two Endpoints on the Same Leaf
Single BD/Single EPG with Two Endpoints on Different Leafs
Single BD/Two EPGs with One Endpoint in EPG on the Same Leaf
Two BDs/Two EPGs with One Endpoint in Each EPG on the Same Leaf (Routed Packet)
Lesson 2: Cisco ACI Router Baseline
Route Profiles Overview
Configurating a Route-Profile
Applying a ‘Default’ Route-Profile
Applying a Route-Profile at the Bridge Domain
Applying a Route-Profile at the Bridge Domain Subnet
Applying a Route-Profile at the External EPG and External EPG Subnet
Applying a Route-Profile at the L3out as an Interleak Policy
Lesson 3: Configure and Verify Changes in the Network
Configure and Verify Interface Profiles
Configure and Verify Static EPGs
Verify Interfaces and Encapsulation
Lesson 4: Cisco ACI Logging and DHCP Relay
Logging data and data retention
Activity and audit logs
DHCP Relay service
Infra/Tenant
L3 Out
L2 Out
Module 2 Lab Exercises
Lab 10 – ACI Packet Flows
Lab 11 - Verify Routing Information
Module 3: Cisco ACI Troubleshooting

Lesson 1: Cisco ACI Troubleshooting Tools
Atomic Counters and Network Time Protocol
Cisco ACI Endpoint Tracker Application
End to End Testing
Endpoint to Endpoint Testing
Endpoint Group to Endpoint Group Testing
Discovering Paths and Testing Connectivity with Traceroute
Lesson 2: On-Demand Diagnostics and Statistics
Gathering On-Demand Collection and Statistics
Cisco Switched Port Analyzer (SPAN)
Access SPAN (or infrastructure SPAN)
Fabric SPAN
Tenant SPAN
ACI Monitoring and Diagnostics
Fabric Troubleshooting Diagnostics
Lesson 3: Cisco Nexus Data Broker Integration with ACI
The Cisco Nexus Data Broker
Test Access Points (TAPs)
Configure SPAN Sessions
Synchronize SPAN Sessions
Data Broker Use Case
Lesson 4: Cisco ACI Troubleshooting Methodology and Scenarios
Techsupport Files
Atomic Counters
Faults and Health Scores
Troubleshooting Scenarios
Troubleshoot Fabric Initialization
Troubleshoot APIC High Availability and Clustering
Troubleshoot Firmware and Image Management
Troubleshoot Rest Interface
Troubleshoot Management Tenant
Troubleshoot Common Network Services
Troubleshoot Unicast Data Plane Forwarding and Reachability
Troubleshoot Policies and Contracts
Troubleshoot Bridged Connectivity to External Networks
Troubleshoot Routed Connectivity to External Networks
Troubleshoot Virtual Machine Manager Insertion
Troubleshoot Layer 4 Through 7 Services Insertion
Troubleshoot ACI Fabric Node Process Crash Troubleshooting
Troubleshoot an APIC Process Crash
Troubleshoot Spanning Tree Loops​
Module 3 Lab Exercises
Lab 12 – ACI Monitoring Tool
Lab 13 – Using Statistics
Lab 14 – Configure Syslog Monitoring
Lab 15 – Configure ERSPAN
Lab 16 – Create and Test an EPG-to-EPG Atomic Counter
Lab 17 – Using the Moquery Command
Lab 18 – Configure and Verify Contracts
Lab 19 – Examine and Enhance Fabric Access Policies
Module 4: Cisco ACI Automation

Lesson 1: The REST API
The REST API Structure
The Six Constraints of REST
REST Client–Server Constraint
REST Stateless Constraint
REST Cacheable Constraint
REST Layered Constraint
REST Code-On-Demand Constraint
REST Uniform Interface Constraint
Lesson 2: JSON and XML
JSON API Characteristics
XML API Characteristics
JSON and XML Advantages, Similarities, and Differences
Lesson 3: Cisco ACI Data Model
The Cisco ACI Programmable Network Infrastructure
The APIC Data Model
The APIC Policy Model
The Cisco ACI Object-Oriented Data Model
Module 4 Lab Exercise
Lab 21 – Configure the APIC Using the REST API
Module 5: Cisco ACI Backups, Snapshots, Rollbacks and Upgrades

Lesson 1: Cisco ACI Backups, Snapshots and Rollbacks
Backing up Configurations
Snapshots
Rollbacks
Lesson 2: Cisco ACI Upgrades
Firmware Repository
Software Image
Switch Image
Catalog Image
Module 5 Lab Exercises
Lab 22 - Backups and Snapshots
Lab 23 - Create and Use Configuration Backups
Appendix (Optional Labs)

Lab 23 – Explore Spanning Tree Protocol and Loop Prevention
Lab 24 – APIC Cluster Operations and Troubleshooting
Lab 25 – Nexus Spine and Leaf Switch Operations
Lab 26 – Upgrade APIC and Switch Firmware

```

**Implementing Cisco Application Centric Infrastructure – Advanced**

https://www.cisco.com/c/dam/en_us/training-events/le31/le46/cln/marketing/exam-topics/600-660-DCACIA.pdf


![](/assets/markdown-img-paste-20200210114008920.png)
