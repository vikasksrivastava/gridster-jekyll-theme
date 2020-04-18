---
layout: post
title: NSO Common Error Messages and their resolution
description: NSO Common Error Messages and their resolution
comments: true
---

---

#### Should be of type path-arg

In this case make sure your `XPATH` does not have a hardcoded variable.

```sh
yang/new_l2vpn.yang:60: error: bad argument value "/ncs:devices/ncs:device{PE11}/ncs:config/ios:interface/ios:GigabitEthernet{1}/ios:name", should be of type path-arg
```

---

#### error: the leafref refers to non-leaf and non-leaf-list node


```sh
yang/new_l2vpn.yang:38: error: the leafref refers to non-leaf and non-leaf-list node 'FastEthernet' in module 'tailf-ned-cisco-ios' at /opt/ncs/packages/neds/cisco-ios/src/ncsc-out/modules/yang/tailf-ned-cisco-ios.yang:34121
```

In this case make sure your `XPATH` points to a `list` , in this case it was `ios:name`

```sh
path "/ncs:devices/ncs:device/ncs:config/ios:interface/ios:GigabitEthernet/ios:name";`
```

The error occurs if you leave the path untill here , expecting to list GigabitEthernet interfaces :

```sh
path "/ncs:devices/ncs:device/ncs:config/ios:interface/ios:GigabitEthernet";
```
Basically the key is , things like `name` , `negotiation` , `descripton` which is a list .

![](/assets/markdown-img-paste-20180502203026575.png)

---

#### XPath error: Invalid namespace prefix: ios

```sh
yang/circuit.yang:57: error: XPath error: Invalid namespace prefix: ios
make: *** [../load-dir/circuit.fxs] Error 1
```

**Solution :**

Import the required module:
```sh
import tailf-ned-cisco-ios {
   prefix ios;
}
```

And add the yang path below in Makefile.

```sh
## Uncomment and patch the line below if you have a dependency to a NED
## or to other YANG files
# YANGPATH += ../../<ned-name>/src/ncsc-out/modules/yang \
#       ../../<pkt-name>/src/yang

YANGPATH += ../../cisco-ios/src/ncsc-out/modules/yang
```
---

#### Aborted: no registration found for callpoint learning_defref/service_create of type=external

This message occurs when the you perform a commit or commit-dry-run . At this stage your YANG is merged with the corresponding XML Definition


```sh
admin@ncs(config-link-PE21)# commit dry-run
Aborted: no registration found for callpoint learning_defref/service_create of type=external
admin@ncs(config-link-PE21)# exit
```

**Solution :**

Make sure that in the corresponding XML code , the `servicepoint` name is correct.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config-template xmlns="http://tail-f.com/ns/config/1.0" servicepoint="learning_defref">
```

---

#### info [l2vpn-template.xml:2 Unknown servicepoint: l2vpn]

This message occurs when you have not compile the src folder with the `make` command.
Since make isnt issued , the XML template cannot find the entrypoint.


```sh
reload-result {
    package l2vpn
    result false
    info [l2vpn-template.xml:2 Unknown servicepoint: l2vpn]
}
```

**Solution :**

Run the `make` command in the src folder.














---

### Another Set of error Messages

```sh
/opt/ncs/packages/neds/cisco-iosxr/src/ncsc-out/modules/yang/tailf-ned-cisco-ios-xr.yang:17988: warning: Given dependencies are not equal to calculated: ../end-marker, ../start-marker. Consider removing tailf:dependency statements.
/opt/ncs/packages/neds/cisco-iosxr/src/ncsc-out/modules/yang/tailf-ned-cisco-ios-xr.yang:26306: warning: when tailf:cli-drop-node-name is given, it is recommended that tailf:cli-suppress-mode is used in combination. using tailf:cli-drop-nodename in a list child without using tailf:cli-suppress-mode on the list, might lead to confusing behaviour, where the user enters the submode without being able to give further configuration.
yang/learning_defref.yang:55: warning: Given dependencies are not equal to calculated: ../router_name, /ncs:devices/ncs:device/ncs:name, /ncs:devices/ncs:device/ncs:device-type/ncs:cli/ncs:ned-id. Consider removing tailf:dependency statements.
yang/learning_defref.yang:68: warning: Given dependencies are not equal to calculated: ../router_name, /ncs:devices/ncs:device/ncs:name, /ncs:devices/ncs:device/ncs:device-type/ncs:cli/ncs:ned-id. Consider removing tailf:dependency statements.
```

Start with the learning_defref lab. And analyse the error.



---
You can delete an entire package our from the NSO by deleting the package from the `packages` dir followed by a `packages reload` . This needs to be forced for the deletion .


```sh
admin@ncs# packages reload
Error: The following namespaces will be deleted by upgrade:
l2vpn: http://com/example/l2vpn
If this is intended, proceed with 'force' parameter.
admin@ncs# packages reload force

>>> System upgrade is starting.
>>> Sessions in configure mode must exit to operational mode.
>>> No configuration changes can be performed until upgrade has completed.
>>> System upgrade has completed successfully.
reload-result {
    package cisco-ios
    result true
}
reload-result {
    package cisco-iosxr
    result true
}
```

---
`ERROR`: Aborted: no registration found for callpoint l2vpn/service_create of type=external

This error points to the the `ncs-run/packages/service-name/template/service_name-template.xml` . Check if the file exists and if any error exists.

```sh
admin@ncs(config-l2vpn-CE11-CE21)# commit dry-run
Aborted: no registration found for callpoint l2vpn/service_create of type=external

* make sure you perform `package reload`
```
---
