
# Getting Started with Kubernetes

```sh
- https://linuxacademy.com/course/kubernetes-essentials/
- https://linuxacademy.com/course/cloud-native-certified-kubernetes-administrator-cka/
- https://linuxacademy.com/course/kubernetes-the-hard-way/
- https://medium.com/platformer-blog/how-i-passed-the-cka-certified-kubernetes-administrator-exam-8943aa24d71d
- Google Kubernetes Enginer Deep Dive
- Implementing a Full CI/CD Pipeline
- https://kubedex.com/7-5-tips-to-help-you-ace-the-certified-kubernetes-administrator-cka-exam/

- https://blog.shanelee.name/2018/10/17/how-i-aced-the-certified-kubernetes-administrator-cka-exam/
- https://www.katacoda.com/courses/kubernetes
- https://matthewpalmer.net/kubernetes-app-developer/
- http://devnetstack.com/certified-kubernetes-administrator-exam-study-guide/
- https://linux.xvx.cz/2018/04/cka-kubernetes-certified-administrator.html
Networking

https://itnext.io/benchmark-results-of-kubernetes-network-plugins-cni-over-10gbit-s-network-36475925a560

```

![](assets/markdown-img-paste-20191030180458986.png)

## Kubernetes Architecture

![](/assets/markdown-img-paste-2019102918553480.png)

**Kube Master has the following components**

  - `Docker` : Runtime environment
  - `kubeadm` : Additional Tool to setup kubernetes cluster. Automates large part of setting up Kubernetes cluster.

  - `kubelet` : Agent that manages the process of running containers. Its an essential components and needs to be running on all nodes. Serves as middleman betweek Kubernetes and Docker.

  - `kubectl` : command line tool to interact/administrative with cluster


  - control-plane : its the series of services that forms the runtime control plane.

Since we are using `kubeadm` , the control plane services will be running as containers.

---

## Setting up the Lab

> Pre-work
> 1. Set Hostname Correctly /etc/hostname
> 2. Set /etc/hosts file correct
> 3. Disable Swap in /etc/fstab
> vi /etc/fstab and # the swap line.


![](/assets/markdown-img-paste-20191029221252629.png)

**Step 1.** Install **Docker** (rkt,containerd are the alternatives) on all three servers.

```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update
#Install a sepcific version of Docker
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu
# Make the package version statick and does not auto upgrade.
sudo apt-mark hold docker-ce
```

**Step 2.** Install the **Kubernetes Components**

```sh

# This just sets the right gpg key so tht during install there is no complain about gpg key mismatch
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# Set the kubernets repository
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

#Install and update the Kubernetes
sudo apt-get update
sudo apt-get install -y kubelet=1.12.7-00 kubeadm=1.12.7-00 kubectl=1.12.7-00

```
<br>

> `sudo apt-mark hold packhage-name`
> This hold the package from being automatically upgraded.

**Step 3.** **Bootstraping the cluster**

```

root@kube-master:~# kubeadm init --pod-network-cidr=10.244.0.0/16

[preflight/images] Pulling images required for setting up a Kubernetes cluster
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
[controlplane] wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests"
[init] this might take a minute or longer if the control plane images have to be pulled
[apiclient] All control plane components are healthy after 20.508207 seconds
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.12" in namespace kube-system with the configuration for the kubelets in the cluster
[markmaster] Marking the node kube-master as master by adding the label "node-role.kubernetes.io/master=''"
[markmaster] Marking the node kube-master as master by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "kube-master" as an annotation
[bootstraptoken] using token: n72r7z.8gmlov9v8fe95ztd
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 150.1.7.54:6443 --token n72r7z.8gmlov9v8fe95ztd --discovery-token-ca-cert-hash sha256:fa5ff8c29297b2a6094c41be012e6de72fc7db50cdeebcee90faeadaee7180d2

```

**Step 4.**

When it is done, set up the local `kubeconfig` from above :
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

And use the config above for joinign the nodes

```sh
root@kube-node-1:~# swapoff -a
kubeadm join 150.1.7.54:6443 --token n72r7z.8gmlov9v8fe95ztd --discovery-token-ca-cert-hash sha256:fa5ff8c29297b2a6094c41be012e6de72fc7db50cdeebcee90faeadaee7180d2


This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
```

Run '`kubectl get nodes`' on the master to see this node join the cluster.

```sh
root@kube-master:~# kubectl get nodes
NAME          STATUS     ROLES    AGE     VERSION
kube-master   NotReady   master   6m45s   v1.12.7
kube-node-1   NotReady   <none>   4m      v1.12.7
kube-node-2   NotReady   <none>   16s     v1.12.7
root@kube-master:~#
```

> Notice the status above of NotReady , it is goign to staty in the NotReady status untill we setup networking . We will do that next.


# We will do the networking using Flannel

Kubernetes can do multiple other networkign plugins , but in this lessons we will use the Flannel.

**Step 1.** First thing which we need to do is to do this on all the nodes :

```sh
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

**Step 2.** next thing we do is setup the Flannel based on the YAML values provided by coreOS here :

https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

net.bridge.bridge-nf-call-iptables = 1
annel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml/fl
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created

```

> If you are curious the YAML file has content like this (networking parameters)

```yaml
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```

Now all our nodes are up

```sh
root@kube-master:~# kubectl get nodes
NAME          STATUS   ROLES    AGE   VERSION
kube-master   Ready    master   20m   v1.12.7
kube-node-1   Ready    <none>   17m   v1.12.7
kube-node-2   Ready    <none>   13m   v1.12.7
```

To verify if our Flannel services are up , we can look into the "system" pod which is the internal pod for the kubernetes system and has the flannel services running.


```sh
root@kube-master:~# kubectl get pods -n kube-system
NAME                                  READY   STATUS    RESTARTS   AGE
coredns-bb49df795-hhzfj               1/1     Running   0          20m
coredns-bb49df795-zh972               1/1     Running   0          20m
etcd-kube-master                      1/1     Running   0          20m
kube-apiserver-kube-master            1/1     Running   0          19m
kube-controller-manager-kube-master   1/1     Running   0          19m
kube-flannel-ds-amd64-4jf9p           1/1     Running   0          107s
kube-flannel-ds-amd64-gvnzp           1/1     Running   0          107s
kube-flannel-ds-amd64-jvtdn           1/1     Running   0          107s
kube-proxy-2mxbl                      1/1     Running   0          20m
kube-proxy-hkczm                      1/1     Running   0          14m
kube-proxy-nqtfd                      1/1     Running   0          18m
kube-scheduler-kube-mast
```

---
<p align="center">
  <img width="50" height="100" src="/assets/markdown-img-paste-20191029211927809.png">
</p>

---

# Containers and Pods

## How to create a pod
`Pods` are the smallest and most basic building block of the Kubernetes model.

![](assets/markdown-img-paste-20191030170225644.png)

A `pod` consists of one or more `containers`, `storage resources` and a `unique ip address`

Kubernetes shcedules pods to be created on the nodes. These pods are then used to create the containers inside.

Example of a basic pod creation
```sh
root@kube-master:~# kubectl create -f basic.yml
pod/nginx created
```

```yaml
#basic.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```
## How to see pod details
```sh
root@kube-master:~# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          85s
```
```sh
root@kube-master:~# kubectl describe pod nginx
Name:               nginx
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               kube-node-1/150.1.7.52
Start Time:         Wed, 30 Oct 2019 23:07:18 +0200
Labels:             <none>
Annotations:        <none>
Status:             Running
IP:                 10.244.1.2
Containers:
  nginx:
    Container ID:   docker://927944a0f4224d2c4b32b35003897ed17c522e11884cd316db4dfb2ac6ca21ea
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:922c815aa4df050d4df476e92daed4231f466acc8ee90e0e774951b0fd7195a4
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 30 Oct 2019 23:08:01 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-ftzdf (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-ftzdf:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-ftzdf
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From                  Message
  ----    ------     ----   ----                  -------
  Normal  Scheduled  2m28s  default-scheduler     Successfully assigned default/nginx to kube-node-1
  Normal  Pulling    2m27s  kubelet, kube-node-1  pulling image "nginx"
  Normal  Pulled     105s   kubelet, kube-node-1  Successfully pulled image "nginx"
  Normal  Created    105s   kubelet, kube-node-1  Created container
  Normal  Started    105s   kubelet, kube-node-1  Started container
```
## How to check the node details

```sh
root@kube-master:~# kubectl describe node kube-node-1
Name:               kube-node-1
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=kube-node-1
Annotations:        flannel.alpha.coreos.com/backend-data: {"VtepMAC":"32:ae:45:85:75:0c"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 150.1.7.52
                    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 30 Oct 2019 02:57:17 +0200
Taints:             <none>
Unschedulable:      false
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  OutOfDisk        False   Wed, 30 Oct 2019 23:14:47 +0200   Wed, 30 Oct 2019 04:07:42 +0200   KubeletHasSufficientDisk     kubelet has sufficient disk space available
  MemoryPressure   False   Wed, 30 Oct 2019 23:14:47 +0200   Wed, 30 Oct 2019 04:07:42 +0200   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Wed, 30 Oct 2019 23:14:47 +0200   Wed, 30 Oct 2019 04:07:42 +0200   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Wed, 30 Oct 2019 23:14:47 +0200   Wed, 30 Oct 2019 02:57:17 +0200   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Wed, 30 Oct 2019 23:14:47 +0200   Wed, 30 Oct 2019 04:07:42 +0200   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  150.1.7.52
  Hostname:    kube-node-1
Capacity:
 cpu:                4
 ephemeral-storage:  19027240Ki
 hugepages-2Mi:      0
 memory:             8076512Ki
 pods:               110
Allocatable:
 cpu:                4
 ephemeral-storage:  17535504355
 hugepages-2Mi:      0
 memory:             7974112Ki
 pods:               110
System Info:
 Machine ID:                 ee5322b8d5ec03b7129cec5b5ac40a31
 System UUID:                F871B357-D927-45A8-B163-042F5A3607D5
 Boot ID:                    f5796017-10ec-48a4-8421-f194bbe0fc22
 Kernel Version:             4.4.0-116-generic
 OS Image:                   Ubuntu 16.04.4 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.6.1
 Kubelet Version:            v1.12.7
 Kube-Proxy Version:         v1.12.7
PodCIDR:                     10.244.1.0/24
Non-terminated Pods:         (3 in total)
  Namespace                  Name                           CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                           ------------  ----------  ---------------  -------------
  default                    nginx                          0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-flannel-ds-amd64-jvtdn    100m (2%)     100m (2%)   50Mi (0%)        50Mi (0%)
  kube-system                kube-proxy-nqtfd               0 (0%)        0 (0%)      0 (0%)           0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource  Requests   Limits
  --------  --------   ------
  cpu       100m (2%)  100m (2%)
  memory    50Mi (0%)  50Mi (0%)
Events:
  Type     Reason                   Age                From                     Message
  ----     ------                   ----               ----                     -------
  Normal   Starting                 20h                kubelet, kube-node-1     Starting kubelet.
  Normal   NodeHasSufficientDisk    20h                kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasSufficientDisk
  Normal   NodeHasSufficientMemory  20h                kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    20h                kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     20h                kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  20h                kubelet, kube-node-1     Updated Node Allocatable limit across pods
  Normal   Starting                 20h                kube-proxy, kube-node-1  Starting kube-proxy.
  Normal   NodeReady                20h                kubelet, kube-node-1     Node kube-node-1 status is now: NodeReady
  Normal   NodeReady                19h                kubelet, kube-node-1     Node kube-node-1 status is now: NodeReady
  Normal   NodeHasSufficientDisk    19h (x2 over 19h)  kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasSufficientDisk
  Normal   NodeHasSufficientMemory  19h (x2 over 19h)  kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    19h (x2 over 19h)  kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     19h                kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasSufficientPID
  Normal   Starting                 19h                kubelet, kube-node-1     Starting kubelet.
  Normal   NodeNotReady             19h                kubelet, kube-node-1     Node kube-node-1 status is now: NodeNotReady
  Normal   NodeAllocatableEnforced  19h                kubelet, kube-node-1     Updated Node Allocatable limit across pods
  Warning  Rebooted                 19h                kubelet, kube-node-1     Node kube-node-1 has been rebooted, boot id: db07bcb4-6975-4211-a112-7381340be70d
  Normal   Starting                 19h                kube-proxy, kube-node-1  Starting kube-proxy.
  Normal   Starting                 13m                kubelet, kube-node-1     Starting kubelet.
  Normal   NodeHasSufficientDisk    13m (x2 over 13m)  kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasSufficientDisk
  Normal   NodeHasSufficientMemory  13m (x2 over 13m)  kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    13m (x2 over 13m)  kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     13m (x2 over 13m)  kubelet, kube-node-1     Node kube-node-1 status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  13m                kubelet, kube-node-1     Updated Node Allocatable limit across pods
  Warning  Rebooted                 13m                kubelet, kube-node-1     Node kube-node-1 has been rebooted, boot id: f5796017-10ec-48a4-8421-f194bbe0fc22
  Normal   Starting                 13m                kube-proxy, kube-node-1  Starting kube-proxy.
```

# Kubernetes Architecture and Components

There are multiple components which comprise the Kubernetes system.

The following commands basically lists all the components responsible for running the kubernetes-system.

```sh
root@kube-master:~# kubectl get pods -n kube-system
NAME                                  READY   STATUS    RESTARTS   AGE
coredns-bb49df795-hhzfj               1/1     Running   4          33h
coredns-bb49df795-zh972               1/1     Running   4          33h
etcd-kube-master                      1/1     Running   4          33h
kube-apiserver-kube-master            1/1     Running   4          33h
kube-controller-manager-kube-master   1/1     Running   4          33h
kube-flannel-ds-amd64-4jf9p           1/1     Running   3          32h
kube-flannel-ds-amd64-gvnzp           1/1     Running   4          32h
kube-flannel-ds-amd64-jvtdn           1/1     Running   3          32h
kube-proxy-2mxbl                      1/1     Running   4          33h
kube-proxy-hkczm                      1/1     Running   3          33h
kube-proxy-nqtfd                      1/1     Running   3          33h
kube-scheduler-kube-master            1/1     Running   4          33h
```

`etcd` : Distibuted synchornised datastore , all information about Pod , Nodes and al data storage .

`kube-api` : It serves the Kubernetes API , which every system uses.

`kube-controller-manager` : Does all the behind the scenes work.

`kube-scheduler` : Is behind the scenes responsible for scheduling pods.

`kubelet` service which runs as a system service , is the middleman between Container runtime and Kubernetes.

`kube-proxy`kube-proxy handles network communication between different nodes.

# Kubernetes Deployments

`Deployments` are a type of object in kubernetes. So far we have been running individiual pods. But when we need to run it production we need something better.

`Scaling` : In Deployment we defien the desired state. Here we can define the number of replicas we need in our pod, Deployment will ensure that those many replicas stay in the system.

`Rolling Update` Deployments enables rolling updates as new version of services are being deployed.

**Deployment Example**

Following is the Deployment file we are goign to use.
The deployment is named `nginx-deployment` , having `2` replicas.

```yaml
#nginxDeployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
        ports:
        - containerPort: 80
```

Following command executed the above deployment file
```sh
root@kube-master:~# kubectl create -f nginxDeployment.yaml
deployment.apps/nginx-deployment created
```

We can check the status of the pods created by the deployment below.

```sh
root@kube-master:~# kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
nginx                             1/1     Running   1          13h
nginx-deployment-d55b94fd-72pj4   1/1     Running   0          107s
nginx-deployment-d55b94fd-n8fjm   1/1     Running   0          107s
```

In this example , we will delete a pod which was created by the deloyment and notice that the Deployment spins up a new Pod to replinish the same

```sh
root@kube-master:~# kubectl delete pod nginx-deployment-d55b94fd-72pj4
pod "nginx-deployment-d55b94fd-72pj4" deleted
```

The following command show the deployments and you can use the `describe` to get more details.

```sh
root@kube-master:~# kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2         2         2            2           88s
```

```sh
root@kube-master:~# kubectl describe deployments nginx-deployment
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.15.4
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-d55b94fd (2/2 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m43s  deployment-controller  Scaled up replica set nginx-deployment-d55b94fd to 2
root@kube-master:~# kubectl describe deployments nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 31 Oct 2019 12:11:56 +0200
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               2 desired | 2 updated | 2 total | 1 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.15.4
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      False   MinimumReplicasUnavailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-d55b94fd (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  4m7s  deployment-controller  Scaled up replica set nginx-deployment-d55b94fd to 2
```

# Kubernetes Services

A Kubernetes Service allows the access to the rapidly changing pods and their location.

Services allows load-balanced service from one of the pods.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

```sh
root@kube-master:~# kubectl create -f nginxService.yaml
service/nginx-service created
```
The service allows the nginx containers now to be accessible on the node with the port 30080

```sh
root@kube-master:~# kubectl get service
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        33h
nginx-service   NodePort    10.105.91.20   <none>        80:30080/TCP   25s
```

```sh
kubectl proxy --address='0.0.0.0' --accept-hosts='^*$'
```

# Kubernetes Microservices

Microservices are a collection of services which forms the full service

Example here is the **Robot Shop Microservice**

```sh
git clone https://github.com/linuxacademy/robot-shop.git

kubectl create namespace robot-shop
kubectl -n robot-shop create -f ~/robot-shop/K8s/descriptors/

kubectl get pods -n robot-shop -w

http://$kube_server_public_ip:30080
```


# Kubernetes Troubleshooting

**Bash Completions**
```sh
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples


source <(kubectl completion bash)
```

**Services Status Check**

```sh
root@kube-master:~# swapoff --a
root@kube-master:~# systemctl start kubelet

root@kube-master:~# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded /lib/systemd/system/kubelet.service; enabled; vendor preset:
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf

```

```sh
root@kube-master:~# kubectl get nodes
NAME          STATUS   ROLES    AGE   VERSION
kube-master   Ready    master   20h   v1.12.7
kube-node-1   Ready    <none>   20h   v1.12.7
kube-node-2   Ready    <none>   20h   v1.12.7
```

```sh
#This command provides container listings with their IP Addresses and Node Information.
root@kube-master:~# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE
nginx   1/1     Running   1          12h   10.244.1.4   kube-node-1   <none>
```

```sh
#This command lets you execute your command inside the container.
#Here you are running a command 'curl' in the busybox container.
kubectl exec busybox -- curl nginx_pod_ip
```

# Kubernetes GUI Access

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml
kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml

kubectl describe sa admin-user -n kubernetes-dashboard
kubectl describe secret admin-user1-token-8wdg9 -n kubernetes-dashboard
```

```sh
# User Creation

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
