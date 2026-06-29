# Geneve Overlay

## The problem

Pods get IPs from the **cluster network** (e.g. `10.0.0.0/14`). The physical network doesn't know how to route those pod IPs — they only mean something inside the cluster. So how does a pod on **node 1** reach a pod on **node 2**?

---

## The answer — an overlay

OVN builds an **overlay network**. Pod-to-pod traffic is **wrapped (encapsulated)** inside an ordinary packet addressed **node-to-node**, sent across the physical network, then **unwrapped** on the far node and delivered to the target pod.

The physical network only ever sees **node-to-node** traffic — it never sees or routes pod IPs. The pod network effectively rides "on top of" the physical network.

---

## What Geneve is

**Geneve (Generic Network Virtualization Encapsulation)** is the encapsulation protocol OVN-Kubernetes uses for this. It's similar in idea to VXLAN, but more flexible. It's what does the wrapping.

---

## Where it runs

Geneve tunnels run **between nodes over the machine network** (using the node IPs). Each node is a tunnel endpoint, with a tunnel to every other node.

---

## The flow

```
pod A (node 1)
   → br-int
   → encapsulate in Geneve  (outer packet: node1 IP → node2 IP)
   → across the machine network
   → node 2
   → br-int
   → decapsulate
   → pod B
```

The **outer** packet carries node IPs (which the physical network *can* route); the **inner** packet is the original pod-to-pod traffic.

---

## Encapsulation point

The wrapping and unwrapping happen at **br-int** (driven by OVN). br-int on the sending node adds the Geneve header; br-int on the receiving node strips it.

---

## Same node vs different node

| Case | Path |
|------|------|
| Pods on the **same node** | No Geneve — OVN forwards locally within br-int |
| Pods on **different nodes** | Geneve tunnel over the machine network |

---

## MTU note

Encapsulation adds header overhead, so the **overlay MTU is slightly smaller** than the physical interface MTU (the inner packet must leave room for the outer Geneve header). This is one reason jumbo frames on the physical network are helpful — they leave plenty of room for the overhead.