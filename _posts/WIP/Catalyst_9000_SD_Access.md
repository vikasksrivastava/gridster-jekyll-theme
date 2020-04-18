## Catalayst 9000 and SD Access

### LISP is the Control Plane and Data Plane is VXLAN .

The controllers foes support YANG model , but as of today it supports CLI model .

![](/assets/markdown-img-paste-20180328061804850.png)

On the top of the underlay we have the overlay network.

The `Overlay Control Plane` also knows as `Host Tracking Database` .

### The Key Concepts

![](/assets/markdown-img-paste-20180328062152822.png)

#### Control Plane

In simple terms its a very simple key value pair and does the resolution of lookup.
Once a packet comes on the network , the Control Nodes where the device is (just like the behaviour of a VTEP)
and the packet is sent to that VTEP.


### Edge Node

- Responsible for Indentifying and Authenticating Endpoints (802.1X / AD)
- Registers the specific endpoint to the the `Control Plane`
- The `Edge Node` also becomes the Anycast Gateway for any device connected to it. (Anycast gateway is like the HSRP VIP on any switch! just like the BGP-EVPN Contol Plane)
- `Edge Node` also performs the encap/decap of VXLAN.

### Border Nodes

- `Fabric Border` Connects to Known networks. This is where we know these are our networks and we want it to be routed and import that routing table in your network.
- `Default Border` Used for unknown routes outside your company. This is like where Internet where you do not want to learn all the internet routes and set a default route to the internet. Think of the default border as the one which acts as a default route out for the internet.


### Sounds Like ACI

- The Fabric Does look similar though this is for Access Layer .
- Wireless
- It supports any topology unlike ACI which is strictly a leaf spine architecture.
- Underlay can support any protocol .

### Fabric Border Nodes

- Connects to any `Known` IP subnets attached to the the outside network.
- The Virtual Network is mapped to the VRF .

### Fabric Enable Wireless LAN Controller.


![](/assets/markdown-img-paste-20180328063908782.png)

The managment is still centralised as today , but the datapath becomes much more scalablae and high bandwidth capable. Notice in the picture above the Datapath between the WLCs.

The `Fabric Enabled WLC` registers Wireless Clients with the Control Plane (the LISP DB).

### Virtual Network

`Virtual Network` maintains a separate Routing and Switching instance for each VN.

The `Virtual Network` is based on VRFs. Now why do we not call it VRF is it is based on it ?

In the picture below only the Fabric Border and the Edge nodes have the VRFs. Anything else between these devices is a VXLAN Forwarder.

![](/assets/markdown-img-paste-20180328064342979.png)

### Scalable Group Tag

![](/assets/markdown-img-paste-20180328064612236.png)

SGT is carried in the VXLAN and now can tranverse devices which do not have SGT Capability. Unlike hostorically where SGT was a hop by hop protocol.

`Subtended Node` Should be connected to only one physical edge (No Redundancy ?)

### A very important and good slide summarising a lot of info :

![](/assets/markdown-img-paste-20180328071006218.png)

### A light and scalable Control Plane based on LISP

![](/assets/markdown-img-paste-20180328072634856.png)

Notice the Table Sizes on the routers have narrowed down and is offloaded to the Mapping Database.

### LISP Lookup Protocol

![](/assets/markdown-img-paste-2018032807290141.png)

![](/assets/markdown-img-paste-20180328072929191.png)

### The VXLAN Header

![](/assets/markdown-img-paste-20180328073258309.png)

VRF is carried in the VNI and SGT is carried in the Se

### Cisco Trustsec

![](/assets/markdown-img-paste-20180328073504455.png)


#### Cisco Trustsec TAG Assignment

![](/assets/markdown-img-paste-20180328073634552.png)



![](/assets/markdown-img-paste-20180328073904227.png)


`Grapevine`  is the Assurance Part , which is a container infrastructructure whic is moving to Maglab.

APIC-EM Services has the Inventory and other servies.

Then using the north bound APIS the GUI is created.


![](/assets/markdown-img-paste-20180328074403482.png)


### APIC EM Cluster Architecture

![](/assets/markdown-img-paste-20180328074752927.png)

- Switch Support Matrix for SD Access wired and wireless
- Campus Fabric
- What Exactly is an Overlay 
  - Takes the abstraction details of  out of the network
  - Simple to Onboard Users
  - THis is what provides your services , firewall
- What is SGT and Trustsec Primer
- DNA Structure
- Anatomay of Attck
- Trustsec Segmentation
- DNA Device Discovery Integration
- ISE Integration Point
- Authentication and Authorisation ISE Policy



-
