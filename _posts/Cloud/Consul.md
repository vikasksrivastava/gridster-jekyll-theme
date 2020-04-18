

# Gettign Started with Consul

**Challenges Faced Today**

- Configuring Software to Connect Services.
- Hardcoding IP Address.
- Hostnme into config files.
- - Trying to update thise
- With advent od docker , need to find critical services

**What you will Learn in this course**

- Consul is best tools to disocver and seervices directly
- Service Discover
- DNS and HTTP API
- Integrated health checking
- Seeting KV Store
- Break Down Systems in Reusable Components (Email Service , Mail Gun , Stripe )
- Let Somebody else be the expert of domain and we focus on value add we care about)

**Break Down Web Monolith into multiple services**

 -- Portal
 -- Customers
 -- Orders
 -- Inventory

**Example HA Proxy**

# Consul Value

`zookeeper` and `etcd` as Key Value Store and Store Services Information There . But then you have to enable your services and application to be able to read and wirte to key value stores.

`consul` brings the first class concept of service discovery using DNS.

# Consul Foundation

`Nodes` are this hosts where services reside.


# Consul Installation

**1. Download the binary (The installation process is dead simple)**

```sh
# Download consul
CONSUL_VERSION=0.6.4
curl https://releases.hashicorp.com/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_amd64.zip -o consul.zip

# Install consul
unzip consul.zip
sudo chmod +x consul
sudo mv consul /usr/bin/consul
```

Server Mode will be turned on , this is easy to get started

```sh
#This commands tells Cosul on which address to advertise
consul agent -dev -advertise 172.20.20.31

```
![](assets/markdown-img-paste-20191112041532756.png)

Notice above that we have the `Client Address` and `Cluster Address`.

`Client Address` : The services that conul provides on Client Address are meant to be consumed "locally" only.

> User Interface and HTTP API are different and does not depend on each other


**2. In this Step we consifgure a desktop to join the consul cluster created above and access the HTTP UI locally**

```sh
consul agent -config-file desky.consul.json
```

```sh
# desky.consul.json
{
  "ui": true,
  "retry_join": "172.20.20.31", # Join this Cluster Address
  "advertise_addr": "172.20.20.1", # Advertise this address for this host
  "data_dir": "/tmp/consul",
}
```

![](assets/markdown-img-paste-20191112042450206.png)

## Different ways to access Consul


#### API
```sh
http://localhost:85000/v1/catalog/nodes
```
![](assets/markdown-img-paste-20191112044948381.png)

#### DNS

```sh
dig @localhost -p 8600 <name.of.node>.node.consul
#Since the query type is node we are lookign under .node.consul
```

![](assets/markdown-img-paste-2019111204530844.png)

**This is the node where the service is running at**
```sh
dig @localhost -p 8600 <name.of.service>.service.consul
#Since the query type is node we are lookign under .service.consul
```

**We can also get the service record**
```sh
dig @localhost -p 8600 <name.of.service>.service.consul SRV
#Looking for the SRV record
```

![](assets/markdown-img-paste-20191112045712948.png)

#### RPC

We can also connect to the RPC Address
![](assets/markdown-img-paste-20191112050243117.png)


**Different Address Bindings which happen by default**

![](assets/markdown-img-paste-20191112050734813.png)
Notice the advertise flag is only for the services hosted on the cluster address.

**Running Consul Agent Service**
```sh
consul agent -advertise $ip --config-file /vagrant/common.json
```

```sh
#common.json
{
  "ui": false,
  "advertise_addr": "172.20.20.1",
}
```

**Consul Execurting Remote Commands**

![](assets/markdown-img-paste-20191112051729964.png)

![](assets/markdown-img-paste-20191112051852446.png)


**Differnet Signals**
`SIGHUP (Hangup)` : Leavbes the cluster gracefully
`SIGKILL` : Abrupt Kill


![](assets/markdown-img-paste-20191112052006119.png)

![](assets/markdown-img-paste-20191112052300610.png)

### Defining/Registering a Service In Consul

```json
{
  "service": {
    "name": "web",
    "port": "8080"
  }
}
```


![](assets/markdown-img-paste-20191112053146180.png)

A service registration is different from service maangement. So basically what it means is that when you register a service , irrespective of the actual service status this will show healthy. So even if there was no we server on port 8080 this will stay green.

### Addign Health Check to the service above

```json
{
  "service": {
    "name": "web",
    "port": "8080",
    "check": {
      "http": "http://localhost:8080",
      "interval": "10s"
    }
  }
}
```

> Serf : Is basically health check for the "node" itself.

**Running the Docker Container for Tests**

![](assets/markdown-img-paste-20191112054009153.png)

`Local Machnine Port` : `Container Port`
`Local File System` : `Container File System`
