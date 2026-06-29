# Localnet

## Recap — the overlay

By default, pod and VM traffic rides the **Geneve overlay**: workloads get **cluster-network** IPs that are hidden from the physical network and tunnelled between nodes.

---

## Why localnet is needed

Sometimes a workload — often a **VM** — must appear **directly on a physical VLAN**: get an IP from that VLAN's subnet and be reachable by external systems as if it were a normal physical host. The overlay can't do this (its IPs are cluster-internal and encapsulated). **Localnet** solves it.

---

## What localnet is

**Localnet** is an OVN connection type that bridges a logical network **directly onto a physical VLAN**, through the VLAN bridge (`br-vlans`). There's **no encapsulation, no NAT, no tunnel** — the workload sits straight on the VLAN.

---

## How it works

```
VM
  → br-int
  → localnet port
  → patch port
  → br-vlans
  → bond / trunk uplink
  → physical VLAN
  → router
```

The VM gets an IP from the **VLAN's own subnet**, and the router sees it like any other host on that VLAN.

---

## How it's configured

- An NMState **localnet mapping** ties an OVN localnet name to the bridge (`br-vlans`).
- A **NAD (NetworkAttachmentDefinition)** attaches workloads to that localnet network. *(NAD covered in its own note.)*

---

## Overlay vs localnet

| | Overlay (default) | Localnet |
|---|-------------------|----------|
| IP from | cluster network | the physical VLAN subnet |
| Encapsulation | Geneve tunnel | none — direct |
| Visibility | hidden inside the cluster | visible on the VLAN |
| Typical use | normal pods | VMs that must look like real hosts |

---

## They coexist

Overlay and localnet run side by side on the same cluster. OVN can **route between them**, so a localnet VM and an overlay pod can still communicate — they're not isolated worlds.