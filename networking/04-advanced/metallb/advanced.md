# Advanced

A reference of advanced MetalLB options, grouped by area. These tune *how* allocation and announcement behave.

---

## IPAddressPool

- **Control automatic allocation** — mark a pool as auto-assign or explicit-only.
- **Scope to namespace / service** — restrict which namespaces or services may draw from a pool.
- **Handle buggy networks** — `avoidBuggyIPs` skips `.0` / `.255` addresses.
- **Change a service's IP** — reassign by updating the request/pool.
- **PreferDualStack IP family policy** — prefer assigning both IPv4 and IPv6 when available.

---

## L2

- **Limit announcing nodes** — restrict which nodes may own/announce a service IP.
- **Specify interfaces** — limit which network interfaces the LB IP is announced from.
- **Limit to specific services** — scope an L2 advertisement to particular services.

---

## BGP

- **Advertisement configuration** — control aggregation length, local preference, communities.
- **Limit peers to certain nodes** — only specific nodes peer with a given router.
- **Announce from a subset of nodes** — restrict which nodes advertise a service.
- **Announce to a subset of peers** — advertise a pool only to chosen peers.
- **Limit to specific services** — scope a BGP advertisement to particular services.
- **BGP source address** — set the source address used for sessions.
- **Community aliases** — name BGP communities for readable config.
- **Peering and announcing via a VRF** — bind sessions/announcements to a specific VRF.
- **Configuring with FRR-K8s** — advanced FRR-backed options.
- **Graceful restart** — keep forwarding during a BGP session restart.
- **Local AS override** — present a different local AS to a peer.

> Known interoperability issues exist with some networking stacks (e.g. Calico, k3s, kube-router) — check the upstream docs when combining MetalLB with another CNI's BGP.

---

## BFD

**BFD (Bidirectional Forwarding Detection)** can back a BGP session to provide **much faster failure detection** than BGP keepalives alone — dropping a dead session in milliseconds so traffic reroutes quickly.