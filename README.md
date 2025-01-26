# MetalLB-K8s

## How i setup my MetalLB on K8s

### Requirements in infrastructure

First you obviously need a K8s Single Node or Cluster.
And because i use BGP (Border Gateway Protocol) for advertising some of my services,
you will need a router that supports BGP in my case this was OPNsense.

So in short:
```
- K8s (Single Node or Cluster ready for Deployment)
- BGP enabled Router (e.g. OPNsense)
```

### Requirements in services

First you should install MetalLB this can be done by executing this command:
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```

After that make really really sure (in case of single Node use) that your node has no label named:
```
node.kubernetes.io/exclude-from-external-load-
```
You can remove it by executing:
```
kubectl label node <node-name> node.kubernetes.io/exclude-from-external-load-balancers-
```
When your done with that it doesn´t get any more complicated.

Now open the WebUI of your OPNsense and go to "Sytem > Firmware > Plugins", if there are update finish them before hand,

make sure os-frr is installed. If it is go to "Routing > General" and activate check enabled than save.

With that done go to "Routing > BGP" there you configure in General:
```
enable        = checked
BGP AS Number = 64513
```
Then in Neigbors add an Entry and configure it like (repeat for every Node):
```
Enabled                 = checked
Description             = <as you wish>
Peer-IP                 = <ip of your k8s node>
BGP AS Number           = 64514
Update-Source Interface = The interface facing your nodes 
```
After all this, simply execute this command for deploying the yml file in this repo:
```
kubectl apply -f metallb.yml
```

An example k8s svc for using such a loadbalancer ip could be
```
apiVersion: v1
kind: Service
metadata:
  name: your-app
  namespace: your-app-namespace
  annotations:
    metallb.universe.tf/loadBalancerIP: 192.168.1.20
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: https
  selector:
    name: your-app
```

### Customization

But you can change some things in the YAML if you want, for example addresses.
Just enter your wanted Addresspool, for example:
```
192.168.1.0/24 
```
or like:
```
192.168.1.20-192.168.1.30
```

You could set other AS Numbers but please search for possible Numbers. Because there are certain privat usable ranges.

Obviously changeable is the name metadata choose as you like. From all I found they dont need to be the same through the three instances.

For any other configuration that is possible please refer to the MetalLB Docs found here:
https://metallb.io/configuration/_advanced_bgp_configuration/