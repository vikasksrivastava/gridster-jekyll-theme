
# Kubernetes Cluster Architecture

https://docs.bitnami.com/kubernetes/how-to/create-your-first-helm-chart/
https://www.youtube.com/watch?list=PLGgVZHi3XQNn8uIb3wy3u-Eq0xL-BU2Nk&time_continue=222&v=TJ9hPLn0oAs

https://learn.hashicorp.com/consul/getting-started-k8s/minikube?utm_source=consul.io&utm_medium=docs
https://learn.hashicorp.com/consul/getting-started-k8s/helm-deploy


**The Control Plane**

 - `API Server` is the communication hub for all cluster components. It exposes the kubernetes API
 - `Scheduler` : Assigns and maintains your apps to worker nodes.
 - `Controller Manager` : Maintains the cluster , handles failures , replicating components and maintainign correct amount of pods.
 - `etcd` : Its the data store the stores cluster configuration .


**Worker Node**

- kubelet : runs and manages the container on the node and talks to the API server .
- kubeproxy : Loadbalances traffic between application components
- container runtime : docker , rtk etc.


![](assets/markdown-img-paste-20191102195938465.png)

```sh
root@kube-master:~# kubectl get namespace
NAME                   STATUS   AGE
default                Active   3d23h
kube-public            Active   3d23h
kube-system            Active   3d23h
kubernetes-dashboard   Active   2d11h
```

![](assets/markdown-img-paste-20191102200253768.png)

**Checking Cluster Service Status**

```sh
root@kube-master:~# kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health": "true"}
```

**Gettign the YAML Details of a Deployment**

```sh
root@kube-master:~# kubectl get deployments nginx-deployment -o yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: 2019-10-31T10:11:56Z
  generation: 1
  labels:
    app: nginx
  name: nginx-deployment
  namespace: default
  resourceVersion: "48935"
  selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/nginx-deployment
  uid: de5bdfcd-fbc6-11e9-b52e-500000010000
spec:
```

**Selector Fields**

```sh
root@kube-master:~# kubectl get pods --field-selector status.phase=Running
NAME                              READY   STATUS    RESTARTS   AGE
nginx                             1/1     Running   3          3d3h
nginx-deployment-d55b94fd-6pccr   1/1     Running   2          2d14h
nginx-deployment-d55b94fd-n8fjm   1/1     Running   2          2d14h
```

# Service and Network Primitives

A kubernetes service allows you to access the services in your pod externally


![](assets/markdown-img-paste-2019110220573617.png)

Mode detailed look inside Serivces

![](assets/markdown-img-paste-20191102210112987.png)


**How to create a busybox (testing) pod**

```sh
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    args:
    - sleep
    - "1000"
EOF
```

# Installation, Configuration, and Validation (12%)
