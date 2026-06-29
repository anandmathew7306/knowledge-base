# NMState & NNCP (OpenShift)

## Recap

NMState is the **declarative** way to configure Linux networking — you describe the desired state and it makes the system match (covered in the Linux notes). This note is how NMState works **in OpenShift**, across a whole cluster.

---

## The NMState operator

The **NMState operator** runs in the cluster. It watches for NMState custom resources and applies them to nodes through **NetworkManager**. It's the cluster-wide equivalent of running `nmstatectl` on a single host — you declare intent once, and the operator configures the nodes.

---

## NNCP — NodeNetworkConfigurationPolicy

The **NNCP** is the resource you write to declare the **desired node network state** — bonds, VLAN subinterfaces, bridges, VRFs, localnet mappings. You apply it once, and the operator configures the matching nodes to match.

---

## Node selection

An NNCP uses a **node selector** to choose which nodes it applies to — all nodes, or a subset selected by label. This lets different policies target different node groups.

---

## Supporting objects

| Object | Role |
|--------|------|
| **NNCE** (NodeNetworkConfigurationEnactment) | Per-node status — whether the policy applied successfully on each node |
| **NNS** (NodeNetworkState) | Read-only per-node report of the **current actual** network state |

---

## The flow

```
write NNCP (desired state)
   → NMState operator picks it up
   → applies via NetworkManager on selected nodes
   → NNCE reports success/failure per node
   → NNS reflects the resulting live state
```

---

## What it configures here

On our clusters, NNCPs handle the **Day-2** node networking: bonds, VLAN subinterfaces, the custom VLAN bridge (`br-vlans`), VRFs, and localnet mappings. (Day-1 provisioning is done earlier via SiteConfig — see the architecture notes.)