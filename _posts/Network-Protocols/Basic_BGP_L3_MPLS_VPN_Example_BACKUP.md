


![](/assets/markdown-img-paste-20180421171152449.png)

Step1. Configure OSPF between the SP Routers
Step2. Setup MPLS neighborship between the SP routers.
Step3. Create VRF on the PE routers for the CUSTOMER as there can be many customers connected to a PE Router.
Step4. Configure EIGRP between PE and Customer Routers and ensure that the neighborship (EIGRP) is formed.
Step5. Configure MP-BGP between the PE Routers (vpnv4) . There is no need to configure this on the SP2 router.
Step6. Redistribute the Routes from EIGRP to BGP and vice-versa


```sh


hostname SP1
!

ip vrf CUSTOMER
 rd 100:1
 route-target export 1:100
 route-target import 1:100
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.0
 ip ospf network point-to-point
!
interface Ethernet1/0
 ip vrf forwarding CUSTOMER
 ip address 192.168.12.2 255.255.255.0
 duplex full
!
interface Ethernet1/1
 ip address 192.168.23.2 255.255.255.0
 duplex full
 mpls ip
!
!
router eigrp 100
 !
 address-family ipv4 vrf CUSTOMER
  redistribute bgp 1 metric 1500 4000 200 10 1500
  network 192.168.12.0
  autonomous-system 100
 exit-address-family
!
router ospf 1
 network 2.2.2.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
!
router bgp 1
 bgp log-neighbor-changes
 neighbor 4.4.4.4 remote-as 1
 neighbor 4.4.4.4 update-source Loopback0
 !
 address-family vpnv4
  neighbor 4.4.4.4 activate
  neighbor 4.4.4.4 send-community both
 exit-address-family
 !
 address-family ipv4 vrf CUSTOMER
  redistribute eigrp 100
 exit-address-family
!
!
mpls ldp router-id Loopback0
!
end
```

```sh

hostname SP2
!
interface Loopback0
 ip address 3.3.3.3 255.255.255.0
 ip ospf network point-to-point
!
interface Ethernet1/1
 ip address 192.168.23.3 255.255.255.0
 duplex full
 mpls ip
!
interface Ethernet1/2
 ip address 192.168.34.3 255.255.255.0
 duplex full
 mpls ip
!
router ospf 1
 network 3.3.3.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
 network 192.168.34.0 0.0.0.255 area 0
!
!
mpls ldp router-id Loopback0
!
!
end

```

```sh
hostname HQ
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.0
 ip ospf network point-to-point
!
!
interface Ethernet1/0
 ip address 192.168.12.1 255.255.255.0
 duplex full
!
!
router eigrp 100
 network 1.0.0.0
 network 192.168.12.0
!
!
end

```



```sh
hostname BRANCH
!
!
interface Loopback0
 ip address 5.5.5.5 255.255.255.0
 ip ospf network point-to-point
!
!
interface Ethernet1/0
 ip address 192.168.45.5 255.255.255.0
 duplex full
!
!
router eigrp 100
 network 5.0.0.0
 network 192.168.45.0
!
end

```


```sh
!
hostname SP3
!
ip vrf CUSTOMER
 rd 100:1
 route-target export 1:100
 route-target import 1:100
!
interface Loopback0
 ip address 4.4.4.4 255.255.255.0
 ip ospf network point-to-point
!
interface Ethernet1/0
 ip vrf forwarding CUSTOMER
 ip address 192.168.45.4 255.255.255.0
 duplex full
!
interface Ethernet1/2
 ip address 192.168.34.4 255.255.255.0
 duplex full
 mpls ip
!
router eigrp 100
 !
 address-family ipv4 vrf CUSTOMER
  redistribute bgp 1 metric 1500 400 20 20 1500
  network 192.168.45.0
  autonomous-system 100
 exit-address-family
!
router ospf 1
 network 4.4.4.0 0.0.0.255 area 0
 network 192.168.34.0 0.0.0.255 area 0
!
router bgp 1
 bgp log-neighbor-changes
 neighbor 2.2.2.2 remote-as 1
 neighbor 2.2.2.2 update-source Loopback0
 !
 address-family vpnv4
  neighbor 2.2.2.2 activate
  neighbor 2.2.2.2 send-community both
 exit-address-family
 !
 address-family ipv4 vrf CUSTOMER
  redistribute eigrp 100
 exit-address-family
!
!
mpls ldp router-id Loopback0
!
end

```
