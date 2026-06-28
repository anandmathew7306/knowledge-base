# Open vSwitch (OVS)

## Linux bridge (the basic version)

Linux has a **built-in software bridge** — a basic virtual switch. Interfaces attach to it, and it forwards frames between them based on **MAC address learning**, exactly like a simple physical switch. It works, but it's "dumb": it only forwards by MAC and has no programmable behaviour.

---

## What OVS is

**Open vSwitch (OVS)** is a **programmable software switch** that runs on the host. It does everything a Linux bridge does, plus much more:

- Programmable **flow rules** (not just MAC learning)
- Native VLAN handling per port
- **Tunnels** (Geneve, VXLAN) to other hosts
- Can be driven by a central controller

This makes OVS the switch of choice once a host runs many VMs/containers that need rich, dynamic networking.

---

## Bridge

A **bridge** is one OVS software switch. It has **ports**, and forwards traffic between them. A host can run multiple OVS bridges, each isolated unless deliberately connected.

---

## Port and port types

A **port** is a connection point on a bridge. Common port types:

| Type | What it is |
|------|-----------|
| system | A real Linux interface attached to the bridge (NIC, bond, VLAN subinterface) |
| internal | A virtual port OVS creates that the OS sees as a NIC (lets Linux use the bridge) |
| patch | A virtual cable linking two OVS bridges |
| tunnel (geneve/vxlan) | An encapsulated link carrying traffic to another host |

---

## Interface

Inside each port is an **interface** — the actual endpoint that sends/receives. In most cases it's 1:1 with the port (one port, one interface).

---

## Flows / flow rules

OVS forwards traffic using **flow rules**: each rule has a **match** (e.g. source/destination, VLAN, port) and an **action** (forward, drop, modify, send to another port). Instead of only learning MACs, OVS can be programmed with exact rules — and a controller can push these rules dynamically.

---

## Patch ports

A **patch port** is a virtual cable connecting two OVS bridges on the same host. Each bridge has one end; whatever enters one end comes out the other. This is how multi-bridge setups are stitched together.

---

## Tunnels (encapsulation)

A **tunnel port** (Geneve or VXLAN) wraps traffic inside another packet so it can travel to another host over an existing IP network. The far host unwraps it and delivers it. This is how OVS extends a virtual network **across multiple physical hosts**.

---

## Datapath

The **datapath** is where the actual packet forwarding happens. OVS uses a **kernel datapath** for speed — once a flow's decision is known, packets are forwarded in the kernel rather than going up to userspace each time.