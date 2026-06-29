# OVN-Kubernetes

## What OVN-Kubernetes is

**OVN-Kubernetes** is the CNI plugin OpenShift uses to implement pod networking. It builds pod connectivity, services, and network policy on top of **OVN (Open Virtual Network)**, which in turn drives **OVS**.

---

## OVS vs OVN

Two separate pieces that work together:

- **OVS** — the **dataplane**: actually moves packets, in the kernel, fast.
- **OVN** — the **control plane**: decides what the rules should be and programs them into OVS.

OVN sits on top of OVS. OVN-Kubernetes is the layer that translates Kubernetes objects into OVN.

---

## OVN components

| Component | Runs where | Role |
|-----------|-----------|------|
| **ovn-northd** | control plane | The "brain" — translates high-level intent into logical network objects |
| **ovn-controller** | every node | Turns those logical objects into actual OVS flow rules on its node |

---

## OVN logical objects

OVN models the network as **logical objects** (definitions, not physical things):

| Object | What it is |
|--------|-----------|
| Logical switch | A virtual L2 switch (e.g. one per node for its pods) |
| Logical router | Routes between logical switches |
| Logical port | A single pod's connection point |
| ACL | A firewall rule (used to enforce NetworkPolicy) |

---

## How a pod gets networked

```
pod created
   → OVN assigns an IP and creates a logical port
   → ovn-controller programs the matching OVS port on the node
   → a veth pair connects the pod's namespace to OVS
   → pod has working networking
```

Kubernetes asks; OVN-Kubernetes does the wiring through OVN and OVS.

---

## Logical vs physical

OVN objects (switches, routers, ports, ACLs) are **logical** — entries in OVN's database describing what the network *should* look like. **ovn-controller** on each node turns those logical definitions into **real OVS flow rules** that move actual packets. Logical = intent; physical = the flows that enforce it.

---

## What OVN handles

- **NetworkPolicy** → enforced as **OVN ACLs**
- **Service routing** → done in **OVN flows** (replacing the iptables/kube-proxy approach)
- **Cross-node pod traffic** → carried over **Geneve tunnels** between nodes