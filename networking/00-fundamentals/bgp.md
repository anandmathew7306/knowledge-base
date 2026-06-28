# BGP

## What BGP is

**BGP (Border Gateway Protocol)** is the protocol routers use to **exchange routes between networks**. It's how different networks tell each other which destinations they can reach — and it's the protocol that runs the internet.

---

## Autonomous System (AS) & ASN

An **Autonomous System (AS)** is a network under one administrative control. Each AS is identified by an **ASN (Autonomous System Number)**. BGP is fundamentally about ASes announcing to each other which routes (prefixes) they can reach.

---

## eBGP vs iBGP

- **eBGP (external)** — a BGP session between **different ASNs**.
- **iBGP (internal)** — a BGP session **within the same ASN**.

---

## Peering / sessions

BGP runs over **TCP port 179**. Two routers form a **peer (neighbor)** relationship and exchange routes over that connection. The session is **always-on**, kept alive with periodic **keepalive** messages.

---

## Route advertisement

Once peered, routers exchange routing information through messages:

- **UPDATE** — announce reachable prefixes (or withdraw them)
- **WITHDRAW** — remove a prefix that's no longer reachable

Each side builds its routing table from what it learns from its peers.

---

## BGP message types

| Message | Purpose |
|---------|---------|
| OPEN | Starts the session, exchanges parameters (ASN, etc.) |
| UPDATE | Advertises new routes or withdraws old ones |
| KEEPALIVE | Confirms the session is still alive |
| NOTIFICATION | Reports an error and closes the session |

---

## Path selection

When multiple paths to the same destination exist, BGP picks the **best path** using route attributes (such as local preference and AS-path length). Only the chosen best path is installed and used.

---

## ECMP / multipath

When two paths to the same destination are **equally good**, traffic can be load-balanced across both — **ECMP (Equal-Cost Multi-Path)**. This also provides redundancy: if one path fails, the other continues.

---

## Anycast

The **same IP address** can be advertised from **multiple locations** at once. Routers send traffic to the nearest/best one. Combined with ECMP, this gives high availability — if one location stops advertising, traffic shifts to another automatically.

---

## BFD

**BFD (Bidirectional Forwarding Detection)** is a lightweight mechanism that detects a dead link in **milliseconds**. BGP can use BFD to tear down a failed session far faster than waiting for the much slower keepalive timeout.