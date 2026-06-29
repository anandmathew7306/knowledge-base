# OVS Bridges on OpenShift

## Context

On an OVN-Kubernetes node, OVS runs with a specific set of bridges. (Generic OVS concepts — bridges, ports, patch ports — are in the Linux notes; this covers the bridges OpenShift actually uses.)

---

## The three bridges

| Bridge | Role | Name | Created by |
|--------|------|------|-----------|
| `br-int` | Integration bridge — every pod/VM connects here; OVN's main workspace | **fixed** | OVN-K (automatic) |
| `br-ex` | External bridge — cluster egress/ingress and node traffic to the physical network | **fixed** | OVN-K (automatic) |
| `br-vlans` | Carries additional tagged VLANs (e.g. OSS / VRF networks) | **custom** | Added via NMState |

> `br-int` and `br-ex` are **fixed names** that OVN-Kubernetes always creates — they're the same on any OCP cluster. `br-vlans` is just the **name someone chose** for the custom VLAN bridge; another team/cluster could call it `br-oss`, `br-data`, etc.

---

## br-int (integration bridge)

The heart of pod networking. Every pod and VM on the node connects here via a veth port, alongside internal and tunnel ports. **OVN programs all of br-int's flow rules** — nothing on it is configured by hand. It's where service routing, network policy, and pod-to-pod forwarding actually happen.

---

## br-ex (external / gateway bridge)

Connects the cluster to the **physical network**. Its uplink is the machine-network interface. It carries the cluster's external-facing traffic: API, ingress, node traffic, and (encapsulated) pod egress on its way out.

---

## br-vlans (custom VLAN bridge)

An **added** bridge for workload-facing VLANs (OSS, secure, etc.). Its uplink is a **trunk** carrying multiple tagged VLANs, and an **internal handoff port** exposes those VLANs to Linux as subinterfaces. This is what lets VMs/pods reach those external VLAN networks.

---

## Patch ports — how they connect

`br-int` connects to **both** `br-ex` and `br-vlans` through **patch ports** (virtual cables). `br-ex` and `br-vlans` **never connect directly** — all traffic between them passes through `br-int`.

```
     br-ex          br-vlans
        \              /
      (patch)      (patch)
          \          /
           br-int
             |
          pods / VMs
```

br-int is the central hub; the other two are its connections to the outside (external network and VLAN networks respectively).