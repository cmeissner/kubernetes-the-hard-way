# Managing the Container Network Routes

Now that each worker node is online we need to add routes to make sure that Pods running on different machines can talk to each other. In this lab we are not going to provision any overlay networks and instead rely on Layer 3 networking. That means we need to add routes to our router. In GCP each network has a router that can be configured. If this was an on-prem datacenter then ideally you would need to add the routes to your local router.

## Container Subnets

The IP addresses for each pod will be allocated from the `podCIDR` range assigned to each Kubernetes worker through the node registration process. The `podCIDR` will be allocated from the cluster cidr range as configured on the Kubernetes Controller Manager with the following flag:

```
--cluster-cidr=10.200.0.0/16
```

Based on the above configuration each node will receive a `/24` subnet. For example:

```
10.200.0.0/24
10.200.1.0/24
10.200.2.0/24
...
``` 

## Get the Routing Table

The first thing we need to do is gather the information required to populate the router table. We need the Internal IP address and Pod Subnet for each of the worker nodes.

Use `kubectl` to print the `InternalIP` and `podCIDR` for each worker node:

```
kubectl get nodes \
  --output=jsonpath='{range .items[*]}{.status.addresses[?(@.type=="InternalIP")].address} {.spec.podCIDR} {"\n"}{end}'
```

Output:

```
10.1.232.8 10.200.1.0/24 
10.1.232.9 10.200.2.0/24 
10.1.232.10 10.200.0.0/24
```

## Create Routes

```
ip r a 10.200.1.0/24 via $(facter -p ipaddress)
```

```
ip r a 10.200.2.0/24 via $(facter -p ipaddress)
```

```
ip r a 10.200.0.0/24 via $(facter -p ipaddress)
```

To made these settings persistent you have to modify /etc/network/interfaces.d/eth0.conf to looks like this:

```
auto eth0
iface eth0 inet dhcp
  up   ip route add 10.200.1.0/24 via 10.1.232.8
  down ip route del 10.200.1.0/24 via 10.1.232.8
```
