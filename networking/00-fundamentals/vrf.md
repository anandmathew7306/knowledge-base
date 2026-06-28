# VRF

## What a VRF is

**VRF (Virtual Routing and Forwarding)** gives a set of interfaces their **own separate routing table**, isolated from the rest of the system. It's the L3 equivalent of what a VLAN does at L2 — but instead of separating who can talk at link level, it separates *which routes and gateways* are used.

---

## Quick recap — routing table

A **routing table** is the set of rules a host uses to decide where to send a packet, based on its destination IP (which subnet goes out which interface, and which gateway handles everything else).

---

## The problem it solves

By default a host has **one** routing table shared by all interfaces — so there can be only **one default route**. If a host has multiple routed networks on it, their traffic can end up using the **wrong gateway**, leaking between networks.

VRF fixes this by giving each network its **own** routing table — its own default route, its own routes — completely isolated from the others.

---

## How it works

Each VRF has its own routing table. An interface placed inside a VRF only ever consults **that VRF's table** — it cannot see or use routes from another VRF or the main table. Traffic entering on a VRF interface stays within that VRF's routing context.

---

## VLAN vs VRF

| | VLAN | VRF |
|---|------|-----|
| Layer | L2 | L3 |
| Isolates | broadcast domains | routing tables |
| Separates | who can talk at link level | which routes / gateways are used |

They're often used together: a VLAN separates the traffic on the wire, and a VRF isolates that VLAN's routing.

---

## VRF holds any interface type

A VRF isn't limited to VLAN subinterfaces — it can contain any interface (physical NIC, bond, VLAN subinterface, tunnel, etc.). Whatever interface is placed in the VRF uses that VRF's routing table.

---

## When to use

Use a VRF when **multiple routed networks live on one host and must stay isolated**. A single routed network doesn't need one — the main routing table is enough.