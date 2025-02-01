Kube-OVN in Kubernetes

# kubernetes user kube-ovn cni plugin to provide all reach feaures of SDN. On a Installed kubernetes cluster OVN create two subnets

1. ovn-default > all pods connect to this by default
2. join > all node connected to this using ovn0 interface .

With one ovn-default subnet pod on same node communicate with each other using this switch as all these are on same node. However if pod connected to this subnet are on differnet node then communication/packet is routed using join switch > ovn0 port > geneve tunnel interface.

On each Node ovn0 is created 

```
ip -o -br a | grep -i ovn0
```

Check geneve port in ovs normally connected to br-int

```
ovs-vsctl show | grep -i geneve -A5 -B5

```
or

```
ovn-sbctl show 
```

or

```
kubectl-ko sbctl show 

```

Check remote-ip  address of tunnel endpoint assigned to NIC 

```
ip -o -br a | grep "remote-ip"
```

To capture traffice on tunnel interface on port 6081

```
tcpdump -i <tunnel-interface> port 6081 
```


# custom subnet

Custom subnetes can be created to launch application isolated from default subnet either by associating subnet with namespace or my using annotation "ovn.kubernetes.io/logical_switch: <custom-subnet>"


# Each subnet is connected to logical router ovn-cluster
```
root@master01:~# kubectl-ko nbctl lr-list
49c58553-de73-4b8c-a821-efb83c453989 (ovn-cluster)
root@master01:~#
```
```
root@master01:~# kubectl-ko nbctl lrp-list 49c58553-de73-4b8c-a821-efb83c453989
c5b71592-49ce-4efc-bb41-dd776aec7b6f (ovn-cluster-join)
0b30e760-0978-49c3-89c5-437d45d6a6af (ovn-cluster-ovn-default)
9bbfe882-5df3-4fb1-8ac3-1bb996402c61 (ovn-cluster-subnet1)
root@master01:~#
```