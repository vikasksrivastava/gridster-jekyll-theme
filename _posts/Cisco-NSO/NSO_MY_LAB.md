### My NSO Lab ramblings as I learn.


<!-- TOC START min:1 max:5 link:true update:true -->
    - [My NSO Lab ramblings as I learn.](#my-nso-lab-ramblings-as-i-learn)
      - [Lets start with configuring the CE devices to be part of NSO](#lets-start-with-configuring-the-ce-devices-to-be-part-of-nso)
      - [EXERCISE 1. Now lets configure loopback address with a service !](#exercise-1-now-lets-configure-loopback-address-with-a-service-)
      - [Appendix of Exercise Above for Notes and detail annotation on code.](#appendix-of-exercise-above-for-notes-and-detail-annotation-on-code)
      - [EXERCISE 2. Now lets Deploy OSPF Configuration using Device Templates  !](#exercise-2-now-lets-deploy-ospf-configuration-using-device-templates--)
      - [EXERCISE 3. Deploy MPLS L2 VPN configuration with `device` , `interface`and `loopback` as selective option (No manual entry)!](#exercise-3-deploy-mpls-l2-vpn-configuration-with-device--interfaceand-loopback-as-selective-option-no-manual-entry)
      - [EXERCISE 4. Automatically assign PE Router Loopback Addresses](#exercise-4-automatically-assign-pe-router-loopback-addresses)
      - [EXERCISE 5. Configuring a CE device from NSO for L3MPLSVPN](#exercise-5-configuring-a-ce-device-from-nso-for-l3mplsvpn)
      - [EXERCISE 6. Adding a SUB template file than the default generated .](#exercise-6-adding-a-sub-template-file-than-the-default-generated-)
      - [How to Connect to MAAPI](#how-to-connect-to-maapi)
      - [Cisco NSO to add Python logic to a service](#cisco-nso-to-add-python-logic-to-a-service)
      - [* * * ON HOLD  * * * EXERCISE XXXX. Now Lets Automate the Loopback IP Address selection of the CE11 Routers based on the last octet of the address received on Fa0 (The loopback addressed are not required for the L3 xconnect but this is just for fun )](#---on-hold-----exercise-xxxx-now-lets-automate-the-loopback-ip-address-selection-of-the-ce11-routers-based-on-the-last-octet-of-the-address-received-on-fa0-the-loopback-addressed-are-not-required-for-the-l3-xconnect-but-this-is-just-for-fun-)

<!-- TOC END -->



![](/assets/markdown-img-paste-20180429205415788.png)

> Lab Details : PE CSR1000v and CEs does not matter for VPN Confgiration . We will only add the PEs to the

#### Lets start with configuring the CE devices to be part of NSO


```shell

  enable password cisco
  username vikassri pass cisco

  line vty 0 4
   password cisco
   login local
   transport input telnet
  !
```


#### EXERCISE 1. Now lets configure loopback address with a service !


Step 1. XML on the Actual CLI Configuration to be applied

```xml
admin@ncs(config)# commit dry-run outformat xml
result-xml {
    local-node {
        data <devices xmlns="http://tail-f.com/ns/ncs">
               <device>
                 <name>PE11</name>
                 <config>
                   <interface xmlns="urn:ios">
                     <Loopback>
                       <name>0</name>
                       <ip>
                         <address>
                           <primary>
                             <address>1.1.1.1</address>
                             <mask>255.255.255.0</mask>
                           </primary>
                         </address>
                       </ip>
                     </Loopback>
                   </interface>
                 </config>
               </device>
             </devices>
    }
}
```

The resulting XML file will be

```xml

<config-template xmlns="http://tail-f.com/ns/config/1.0"
                 servicepoint="loopback_deploy">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
      <!--
          Select the devices from some data structure in the service
          model. In this skeleton the devices are specified in a leaf-list.
          Select all devices in that leaf-list:
      -->
      <name>{/device}</name>
      <config>
        <!--
            Add device-specific parameters here.
            In this skeleton the service has a leaf "dummy"; use that
            to set something on the device e.g.:
            <ip-address-on-device>{/dummy}</ip-address-on-device>
        -->
        <!-- DEVICE1/IOS BELOW IS THE MAIL STUFF WE NEED TO ADD -->
        <interface xmlns="urn:ios">
          <Loopback>
            <name>0</name>
            <ip>
              <address>
                <primary>
                  <address>{/loopback-ip-address}</address>  <!-- Based on the variable from Yang file -->
                  <mask>255.255.255.0</mask>
                </primary>
              </address>
            </ip>
          </Loopback>
        </interface>



      </config>
    </device>
  </devices>
</config-template>


```

Step 2. Basic Yang Model

Here is the model for our loopback configuration looks like


```shell
module loopback_deploy {
  namespace "http://com/example/loopback_deploy";
  prefix loopback_deploy;

  import ietf-inet-types {
    prefix inet;
  }
  import tailf-ncs {
    prefix ncs;
  }

  augment "/ncs:services" {
    list loopback_deploy {
      key "name";
      ncs:servicepoint "loopback_deploy";
      uses ncs:service-data;

      leaf name {
        type string;
      }
      leaf-list device {
        type leafref {
          path "/ncs:devices/ncs:device/ncs:name";
        }
      }
      leaf loopback-ip-address {
        type inet:ipv4-address {

        }
      }
    }
  }
}

```

`services loopback_deploy PE11_Loopback device PE11 loopback-ip-address 1.1.1.1``

```shell
admin@ncs(config)# commit dry-run
cli {
    local-node {
        data  devices {
                  device PE11 {
                      config {
                          ios:interface {
             +                Loopback 0 {
             +                    ip {
             +                        address {
             +                            primary {
             +                                address 10.0.0.1;
             +                                mask 255.255.255.0;
             +                            }
             +                        }
             +                    }
             +                }
                          }
                      }
                  }
              }
              services {
             +    loopback_deploy PE11_Loopback {
             +        device [ PE11 ];
             +        loopback-ip-address 10.0.0.1;
             +    }
              }
    }
}
```

![](/assets/markdown-img-paste-2018042816445154.png)

#### Appendix of Exercise Above for Notes and detail annotation on code.

```shell
module loopback_deploy {
  namespace "http://com/example/loopback_deploy";
  prefix loopback_deploy;

  import ietf-inet-types {
    prefix inet;
  }
  import tailf-ncs {
    prefix ncs;
  }

  // Without AUGMENT the service name "loopback_deploy" will not be listed under availaible services.

  augment "/ncs:services" {

    list loopback_deploy {
      key "name";   # A key name is required in a list without it you will get the error yang/loopback_deploy.yang:13: error: the list needs at least one key, because it is used as config
      ncs:servicepoint "loopback_deploy";  # This tells NCS About the entrypoint of the service . Without this you will get  *** ALARM package-load-failure: [loopback_deploy-template.xml:2 Unknown servicepoint: loopback_deploy]
      uses ncs:service-data;  # Now since servicepoint is used above , service-data definition is necessary . yang/loopback_deploy.yang:16: error: A service must use the grouping 'ncs:service-data'.

      leaf name {  # Here we define what the key type is (string in this case)
        type string;
      }

      leaf-list device {   # Change this to just leaf (no list)
        type leafref {
          path "/ncs:devices/ncs:device/ncs:name";
        }
      }

      leaf loopback-ip-address {
        type inet:ipv4-address {
          pattern "10\\.0\\.0\\.[0-9]+";  # Note that the entire pattern line can be deleted. And NCS will still be able to enforce IP Address logic since its based on `ietf-inet-types`
        }
      }

    }
  }
}

```

Now you can bulk configure the Loopback IP Addresses on all the routers ! Next , Lets do something more interesting ...

---


#### EXERCISE 2. Now lets Deploy OSPF Configuration using Device Templates  !

1.Define the template

```sh
admin@ncs(config)# show full-configuration devices template OSPF\ Configuration
devices template "OSPF Configuration"
 config
  ios:router ospf 1
   non-vrf passive-interface interface Gi8  # Make Gi8 Passive so that no neighborshio happens on the home network .
   non-vrf network 0.0.0.0 255.255.255.255
    area 0
   !
  !
 !
!
```

 2.Apply the template to all devices

```sh
admin@ncs(config)# devices device-group All\ Routers apply-template template-name OSPF\ Configuration

```

#### EXERCISE 3. Deploy MPLS L2 VPN configuration with `device` , `interface`and `loopback` as selective option (No manual entry)!

1.  The YANG File

```shell
    module new_l2vpn {
      namespace "http://com/example/new_l2vpn";
      prefix new_l2vpn;

      import ietf-inet-types { prefix inet; }
      import tailf-ncs { prefix ncs; }
      import tailf-common { prefix tailf; }
      import tailf-ned-cisco-ios { prefix ios; }

      augment "/ncs:services" {
        list new_l2vpn {
          key "name";
          unique "pw-id";
          uses ncs:service-data;
          ncs:servicepoint "l2vpn";

          leaf name {
            tailf:info "Service Instance Name";
            type string;
          } //name

          leaf pw-id {
            tailf:info "Unique Pseudowire ID";
            mandatory true;
            type uint32 {
              range "1..4294967295";
            }
          } //pw-id

          leaf device1 {
            tailf:info "PE Router1";
            mandatory true;
            type leafref {
              path "/ncs:devices/ncs:device/ncs:name";
            }
          } //device1

          leaf intf-number1 {
            tailf:info "GigabitEthernet Interface ID";
            mandatory true;
            type leafref {
              path "/ncs:devices/ncs:device/ncs:config/ios:interface/ios:GigabitEthernet/ios:name";
            }
          } //int-number1

          leaf remote-ip1 {
            tailf:info "Loopback0 IP Address of Remote PE ";
            mandatory true;
            type leafref {
              path "/ncs:devices/ncs:device/ncs:config/ios:interface/ios:Loopback/ios:ip/ios:address/ios:primary/ios:address";
            }
          } //remote-ip1

          leaf device2 {
            tailf:info "PE Router2";
            mandatory true;
            type leafref {
              path "/ncs:devices/ncs:device/ncs:name";
            }
          } //device2

          leaf intf-number2 {
            tailf:info "GigabitEthernet Interface ID";
            mandatory true;
            type leafref {
              path "/ncs:devices/ncs:device/ncs:config/ios:interface/ios:GigabitEthernet/ios:name";
            }
          } //intf-number2

          leaf remote-ip2 {
            tailf:info "Loopback0 IP Address of Remote PE ";
            mandatory true;
            type leafref {
              path "/ncs:devices/ncs:device/ncs:config/ios:interface/ios:Loopback/ios:ip/ios:address/ios:primary/ios:address";
            }
          } //remote-ip2

        } //list_newl2vpn
      }
    }

```

 2. The XML file

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config-template xmlns="http://tail-f.com/ns/config/1.0"                 servicepoint="l2vpn">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <!-- DEVICE1 -->
    <device tags="nocreate">
      <name>{/device1}</name>
      <config tags="merge">

        <!-- DEVICE1/IOS -->
        <interface xmlns="urn:ios">
          <GigabitEthernet>
            <name>{/intf-number1}</name>
            <xconnect>
              <address>{/remote-ip1}</address>
              <vcid>{/pw-id}</vcid>
              <encapsulation>mpls</encapsulation>
            </xconnect>
          </GigabitEthernet>
        </interface>

      </config>
    </device>

    <!-- DEVICE2 -->
    <device tags="nocreate">
      <name>{/device2}</name>
      <config tags="merge">

        <!-- DEVICE1/IOS -->
        <interface xmlns="urn:ios">
          <GigabitEthernet>
            <name>{/intf-number2}</name>
            <xconnect>
              <address>{/remote-ip2}</address>
              <vcid>{/pw-id}</vcid>
              <encapsulation>mpls</encapsulation>
            </xconnect>
          </GigabitEthernet>
        </interface>

      </config>
    </device>
  </devices>
</config-template>

```
---

#### EXERCISE 4. Automatically assign PE Router Loopback Addresses

> Notice that you do not need to recompile the packagaes using `make` is the change is made in `xml` file. All you need to do is `packages reload` .

The purpose of this task is to unburden the operator from having to know and enter the loopback IP addresses that are used for control-plane mechanisms, such as BGP and LDP. All routers use Loopback0 for this purpose.

**The Main** thing to notice in this example is the use of `deref` directly in the XML which points to the loopback IP addresses .

1. The YANG File

```sh
module learning_deref2 {
  namespace "http://com/example/learning_deref2";
  prefix learning_deref2;

  import ietf-inet-types { prefix inet; }
  import tailf-ncs { prefix ncs; }
  import tailf-common { prefix tailf; }
  import tailf-ned-cisco-ios { prefix ios;}


  augment "/ncs:services"  // AUGMENT is used to add to another data model , OR augment it. So in the example we are adding/augumenting the learning_deref to it.
  {

    list learning_deref2
    {
      key "name";
      uses ncs:service-data;
      ncs:servicepoint "learning_deref2";

      leaf name
      {
        mandatory true;
        type string;
      }

      list link
      {

        min-elements 2;
        max-elements 2;
        key "var1_router_name";

            leaf var1_router_name
            {
              mandatory true;
              type leafref
              {
                path "/ncs:devices/ncs:device/ncs:name";  // This path points to a list of routers , not a sepcific router
              }
            } //routername

            container ios
            {

              //  name=current()/../var1_router_name = PE11
              //  ncs:name=current()/../var1_router_name  EXPANDS to ncs:name=/ncs:services/learning_deref/link/var1_router_name
              when "/ncs:devices/ncs:device[ncs:name=current()/../var1_router_name]/ncs:device-type/ncs:cli/ncs:ned-id='ios-id:cisco-ios'"
              {

                //tailf:dependency "../device";
                //tailf:dependency "/ncs:devices/ncs:device/ncs:device-type";
              }

              leaf intf-number
              {
                mandatory true;
                type leafref
                {
                  path "deref(../../var1_router_name)/../ncs:config/ios:interface/ios:GigabitEthernet/ios:name";
                }
              } //intf-number

            } //ios

      }
    }
  }
}

```

1. The XML File

```xml

<?xml version="1.0" encoding="UTF-8"?>
<config-template xmlns="http://tail-f.com/ns/config/1.0" servicepoint="learning_deref2">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device tags="nocreate">
      <name>{/link[1]/var1_router_name}</name>
      <config tags="merge">
        <interface xmlns="urn:ios" tags="nocreate">
          <GigabitEthernet>
            <name>{/link[1]/ios/intf-number}</name>
            <xconnect tags="merge">
              <!-- Notice Below the refrencing of Loopback0 directly in XML -->
              <address>
                {
                  deref(/link[1]/var1_router_name)/../config/interface/Loopback[name='0']/ip/address/primary/address |
                  deref(/link[1]/var1_router_name)/../config/interface/Loopback[id='0']/ipv4/address/ip
                }
              </address>
              <vcid>1166</vcid>
              <encapsulation>mpls</encapsulation>
            </xconnect>
          </GigabitEthernet>
        </interface>
      </config>
    </device>
    <device tags="nocreate">
      <name>{/link[2]/var1_router_name}</name>
      <config tags="merge">
        <interface xmlns="urn:ios" tags="nocreate">
          <GigabitEthernet>
            <name>{/link[2]/ios/intf-number}</name>
            <xconnect tags="merge">
              <address>
                {
                  deref(/link[2]/var1_router_name)/../config/interface/Loopback[name='0']/ip/address/primary/address |
                  deref(/link[2]/var1_router_name)/../config/interface/Loopback[id='0']/ipv4/address/ip
                }
              </address>
              <vcid>1166</vcid>
              <encapsulation>mpls</encapsulation>
            </xconnect>
          </GigabitEthernet>
        </interface>
      </config>
    </device>
  </devices>
</config-template>
```
---
#### EXERCISE 5. Configuring a CE device from NSO for L3MPLSVPN

YANG
```sh
module l3mplsvpn {
  namespace "http://com/example/l3mplsvpn";
  prefix l3mplsvpn;

  import ietf-inet-types { prefix inet; }
  import tailf-ncs { prefix ncs; }
  import tailf-common {prefix tailf;}

  augment "/ncs:services"
  {
    list l3mplsvpn-ce-config {
      tailf:info "Used to Configure the CE Side of the MPLSL3VPN";
      key "vpn-name";
      uses ncs:service-data;
      ncs:servicepoint "l3mplsvpn-ce-servicepoint";

      leaf vpn-name
      {
        tailf:info "Service Instance Name";
        type string;
      } // Key name

      leaf ce_router_name
      {
        tailf:info "CE Router";
        type leafref
        {
          path "/ncs:devices/ncs:device/ncs:name";
        }
      } // ce_router_name

      leaf ce_device_lo0_ipaddress
      {
        tailf:info "CE Router Loopback 0 IP Address";
        type inet:ipv4-address
        {
          //pattern "10\\.0\\.0\\.[0-9]+";
        }

      } //ce_device_lo0_ipaddress

      leaf ce_to_pe_facing_interface_ipaddress
      {
        tailf:info "CE to PE Facing IP Address";
        type inet:ipv4-address
        {
          //pattern "10\\.0\\.0\\.[0-9]+";
        }

      } //ce_device_lo0_ipaddress


      leaf ce_to_pe_facing_interface_ipaddress2
      {
        tailf:info "CE to PE Facing IP Address";
        type inet:ipv4-address
        {
          //pattern "10\\.0\\.0\\.[0-9]+";
        }

      } //ce_device_lo0_ipaddress


  }
}
}




/*


augment "/ncs:services" {
  list l3mplsvpn {
    tailf:info "Layer-3 MPLS VPN Service";
    key "vpn-name";
    uses ncs:service-data;
    ncs:servicepoint "l3mplsvpn-servicepoint";
    leaf vpn-name {
      tailf:info "Service Instance Name";
      type string;
    }
    leaf vpn-id {
      tailf:info "Service Instance ID (1 to 65535)";
      type uint32;
    }
    leaf link-id-cnt {
      tailf:info "Provides a unique 32 bit numbers used as a site identifier";
      default "1";
      type uint32;
    }
    leaf customer {
      tailf:info "VPN Customer";
      type leafref {
        path "/ncs:customers/ncs:customer/ncs:id";
      }
    }
    list link {
      tailf:info "PE-CE Attachment Point";
      key "link-name";
      leaf link-name {
        tailf:info "Link Name";
        type string;
      }
      leaf link-id {
        tailf:info "Link ID (1 to 65535)";
        type uint32;
      }
      leaf device {
        tailf:info "PE Router";
        type leafref {
          path "/ncs:devices/ncs:device/ncs:name";
        }
      }
      leaf pe-ip {
        tailf:info "PE-CE Link IP Address";
        type string;
      }
      leaf ce-ip {
        tailf:info "CE Neighbor IP Address";
        type string;
      }
      leaf interface {
        tailf:info "Customer Facing Interface";
        type string;
      }
      leaf routing-protocol {
        tailf:info "Routing option for the PE-CE link";
        type enumeration {
          enum "bgp";
          enum "rip";
        }
      }
    }
  }
}

*/


```

XML
```xml

<?xml version="1.0" encoding="UTF-8"?>
<config-template xmlns="http://tail-f.com/ns/config/1.0" servicepoint="l3mplsvpn-ce-servicepoint">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
      <!--          Select the devices from some data structure in the service          model. In this skeleton the devices are specified in a leaf-list.          Select all devices in that leaf-list:      -->
      <name>{/ce_router_name}</name>
      <config>
        <interface xmlns="urn:ios">
          <Loopback>
            <name>0</name>
            <ip>
              <address>
                <primary>
                  <address>{/ce_device_lo0_ipaddress}</address>
                  <mask>255.255.255.0</mask>
                </primary>
              </address>
            </ip>
          </Loopback>
          <GigabitEthernet>
            <name>1</name>
            <ip>
              <no-address>

              </no-address>
              <address>
                <primary>
                  <address>{/ce_to_pe_facing_interface_ipaddress}</address>
                  <mask>255.255.255.0</mask>
                </primary>
              </address>
            </ip>

          </GigabitEthernet>
        </interface>
        <router xmlns="urn:ios">
          <eigrp>
            <as-no>100</as-no>
            <network-ip>
              <network>
                <ip>{/ce_device_lo0_ipaddress}</ip>
              </network>
              <network>
                <ip>{/ce_to_pe_facing_interface_ipaddress}</ip>
              </network>
            </network-ip>
          </eigrp>
        </router>
      </config>
    </device>
  </devices>
</config-template>
```

---

#### EXERCISE 6. Adding a SUB template file than the default generated .

For this to happen you can create a new template file and make sure that the servicepoint
is configured and pointing to the entry point in the YANG file. See the example below.

```xml
<config-template xmlns="http://tail-f.com/ns/config/1.0" servicepoint="l3mplsvpn-ce-servicepoint">

```

---


services l3mplsvpn-ce-config HQ1 ce_device_lo0_ipaddress_side_a 9.9.9.9 ce_router_name_side_a HQ1 ce_to_pe_facing_interface_side_a 1 ce_to_pe_facing_interface_ipaddress_side_a 192.168.100.1 pe_router_name_side_a SP1 Route_Distinguiser 1 Route_Target 100 pe_to_ce_facing_interface_side_a 4 pe_to_ce_facing_interface_ipaddress_side_a 192.168.100.4 side_b_loopback0_ip_address 2.2.2.2 vrf_Customer_name NEW_CUSTY


<br><br> <br><br> <br><br> <br><br> <br><br> <br><br> <br><br> <br><br> <br><br> <br>
---
#### How to Connect to MAAPI

> **Make sure you start the NCS Service**

```python
import ncs

with ncs.maapi.Maapi() as m:
    with ncs.maapi.Session(m, 'admin', 'python'):
        # The first transaction
        with m.start_read_trans() as t:
            address = t.get_elem('/ncs:devices/device{PE11}/address')
            # Example to access a value with "ncs:"
            # address = t.get_elem('/ncs:devices/device{PE11}/ncs:authgroup')
            # example ('ncs:devices/device{PE11}/ncs:config/ios:interface/ios:GigabitEthernet{1}/ios:name')
            print("First read: Address = %s" % address)
```
---



#### Cisco NSO to add Python logic to a service

> `https://www.youtube.com/watch?v=4hCLAYbxjwE&t=817s`

Step 1. Make a service which has a python file to allow premod and post modifications.

```shell
ncs-make-package --service-skeleton `python-and-template` vikassri_auto_lo0_deploy`
```

Step 2. Define the Variable in your XML which needs to updated or changes as per the Python Logic . Lets says its `VARIABLE_IP_Address`

Step 3. Perform logic in the code below and when done add the final variable to for the XML file to read using `vars.add` . This the variable which will be refrenced in the XML file , e.g  {/VARIABLE_IP_Address}

```python
# ------------------------
# SERVICE CALLBACK EXAMPLE
# ------------------------
class ServiceCallbacks(Service):

    # The create() callback is invoked inside NCS FASTMAP and
    # must always exist.
    @Service.create
    def cb_create(self, tctx, root, service, proplist):
        self.log.info('Service create(service=', service._path, ')')

        ###  MYNOTE :  PUT YOUR LOGIC HERE ....
        modified_ip_address = regex (some_IP_Address)

        vars = ncs.template.Variables()

        ###  MYNOTE: Now add the value generated above to the your logic .
        vars.add('VARIABLE_IP_Address', modified_ip_address)

        template = ncs.template.Template(service)
        template.apply('vikassri_auto_lo0_deploy-template', vars)

```

<br><br><br><br><br><br><br><br><br><br><br><br>



























<br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br>


![](/assets/markdown-img-paste-20180430040349433.png)


![](/assets/markdown-img-paste-20180429002142529.png)





```sh
module loopback_deploy {
  namespace "http://com/example/loopback_deploy";
  prefix loopback_deploy;

  import ietf-inet-types {
    prefix inet;
  }
  import tailf-ncs {
    prefix ncs;
  }

  augment "/ncs:services" {
    list loopback_deploy {
      key "name";
      ncs:servicepoint "loopback_deploy";
      uses ncs:service-data;

      leaf name {
        type string;
      }
      leaf-list device {
        type leafref {
          path "/ncs:devices/ncs:device/ncs:name";
        }
      }

      leaf management_ip_address {
        mandatory true;
        type leafref {
           path "/ncs:devices/ncs:device[ncs:name=current()/../device]/ncs:name/address";
         }
      }

      leaf loopback-ip-address {
        mandatory true;
        type inet:ipv4-address {


        }


      }
    }
  }
}

```

```shell
typedef my-base-int32-type {
  type int32 {
    range "1..4 | 10..20"  # 1 to 4 and 10 to 20
  }
}

typedef derived-int32 {
  type my-base-int32-type {
    range "11..max";   # Derived from the typedef above but is only limited to 11 to 20 .
  }
}

```





```shell
module loopback_deploy {
  namespace "http://com/example/loopback_deploy";
  prefix loopback_deploy;

  import ietf-inet-types {
    prefix inet;
  }
  import tailf-ncs {
    prefix ncs;
  }

  augment "/ncs:services" {
    list loopback_deploy {
      key "name";
      ncs:servicepoint "loopback_deploy";
      uses ncs:service-data;

      leaf name {
        type string;
      }
      leaf-list device {
        type leafref {
          path "/ncs:devices/ncs:device/ncs:name";
        }
      }

      leaf management_ip_address {
        type leafref {
           path "/ncs:devices/ncs:device/ncs:name/address";
         }
      }

      leaf loopback-ip-address {
        type inet:ipv4-address {
          "1.1.1"+management_ip_address

        }
      }
    }
  }
}
```






#### * * * ON HOLD  * * * EXERCISE XXXX. Now Lets Automate the Loopback IP Address selection of the CE11 Routers based on the last octet of the address received on Fa0 (The loopback addressed are not required for the L3 xconnect but this is just for fun )

Lets Start with Skeleton creation

`ncs-make-package --service-skeleton template CE_loopback_deploy`

The XML Template will pretty much remain the same as the above configuration , though the login the YANG file needs to change


```xml

<config-template xmlns="http://tail-f.com/ns/config/1.0"
                 servicepoint="loopback_deploy">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
      <!--
          Select the devices from some data structure in the service
          model. In this skeleton the devices are specified in a leaf-list.
          Select all devices in that leaf-list:
      -->
      <name>{/device}</name>
      <config>
        <!--
            Add device-specific parameters here.
            In this skeleton the service has a leaf "dummy"; use that
            to set something on the device e.g.:
            <ip-address-on-device>{/dummy}</ip-address-on-device>
        -->
        <!-- DEVICE1/IOS BELOW IS THE MAIL STUFF WE NEED TO ADD -->
        <interface xmlns="urn:ios">
          <Loopback>
            <name>0</name>
            <ip>
              <address>
                <primary>
                  <address>{/loopback-ip-address}</address>  <!-- Based on the variable from Yang file -->
                  <mask>255.255.255.0</mask>
                </primary>
              </address>
            </ip>
          </Loopback>
        </interface>



      </config>
    </device>
  </devices>
</config-template>
```

Now lets define the YANG

Ok , remember that we need to based the last octed of the loopback based on the last octet of the Fa0 interface .

This is how you find the XPATH

`show running-config devices device PE11 config ios:interface GigabitEthernet | display xpath`

```
      show running-config devices device CE21 address | display xpath

      /devices/device[name='CE21']/address
```


This is how you refrence DEREF()

`path "deref(../../device)/../ncs:config/ios:interface/ios:GigabitEthernet/ios:name";`

Ok
