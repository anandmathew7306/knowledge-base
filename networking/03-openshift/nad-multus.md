# NAD & Multus

## The default — one interface per pod

By default, a pod has a **single network interface** (`eth0`), provided by the primary CNI. Most pods only ever need this one interface.

---

## The need for more

Some workloads need a **second** network interface — for example:

- a **VM** that must sit directly on a specific physical VLAN
- a pod that needs a separate **data** network apart from the default pod network

The default single-interface model can't do this on its own. **Multus** enables it.

---

## What Multus is

**Multus** is a "meta-plugin" — a CNI **multiplexer**. It lets a pod have **multiple network interfaces** by calling other CNI plugins in addition to the primary one. The primary pod network stays as-is; Multus **adds extra** interfaces on top.

---

## What a NAD is

A **NAD (NetworkAttachmentDefinition)** is a resource that **defines an additional network** a pod or VM can attach to. It describes which plugin and configuration to use for that extra interface.

---

## How they work together

```
NAD                  defines the extra network
   ↓
pod / VM annotation  references the NAD
   ↓
Multus               reads it and attaches that extra interface
                     alongside the primary eth0
```

---

## Tie to localnet

A **localnet** network is attached to workloads through a **NAD** — this is how a VM gets placed directly onto a physical VLAN. (See the localnet note.)

---

> Deeper detail — IPAM for secondary interfaces, multiple NADs per pod, specific plugin configs — is covered in the advanced notes.