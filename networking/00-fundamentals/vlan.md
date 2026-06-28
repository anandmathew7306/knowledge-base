# VLAN

## What a VLAN is

A **VLAN (Virtual LAN)** splits one physical switch/network into multiple **logically separate networks**. Devices on different VLANs behave as if they were on entirely separate switches, even though they share the same hardware.

---

## The problem it solves

Without VLANs, each isolated network needs its **own physical switch**. VLANs achieve the same isolation **in software on a single switch** — no extra hardware. One switch can host many independent networks.

---

## How it works — tagging (802.1Q)

When a frame travels on a link carrying multiple VLANs, the switch adds a small **802.1Q tag** marking which VLAN the frame belongs to. The receiving switch reads the tag, keeps the frame in the correct VLAN, and strips the tag before delivering it. This is how traffic for different VLANs stays separated on the same wire.

---

## Access port vs trunk port

- **Access port** — carries a **single VLAN**, traffic is **untagged**. Used for connections to end devices (a server NIC, a laptop).
- **Trunk port** — carries **multiple VLANs**, traffic is **tagged**. Used between switches, or from a switch to a server that needs several VLANs.

---

## Native VLAN

On a trunk port, the **native VLAN** is the one VLAN whose traffic is sent **untagged**. Any frame arriving without a tag is assumed to belong to the native VLAN. All other VLANs on the trunk are tagged as normal.

---

## VLAN ID range

Valid VLAN IDs are **1–4094**.

---

## Isolation

Devices in **different VLANs cannot talk at L2** — they are separate broadcast domains. To communicate, traffic must go up to **L3 (a router)**, which routes between the VLANs.

---

## Subinterfaces

On a server receiving a trunk, each VLAN is split into its own **virtual subinterface** — one logical interface per VLAN, named `parent.vlanid`:

```
bond0.100   → VLAN 100 on bond0
bond0.200   → VLAN 200 on bond0
```

Each subinterface behaves like an independent NIC, carrying only its own VLAN's traffic.