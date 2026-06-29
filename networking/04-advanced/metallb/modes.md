# Modes

MetalLB announces service IPs in one of two modes: **L2** or **BGP**.

---

## L2 mode (ARP / NDP)

- Uses standard address-discovery protocols — **ARP** for IPv4, **NDP** for IPv6.
- **One node** in the cluster takes ownership of each service IP (the "leader" for that IP).
- All traffic for that service IP goes to the **owning node**.
- **kube-proxy** then spreads the traffic to the service's pods.
- A **failover** mechanism re-elects a new owner if the node fails.

**Limitations:**
- **Single-node bottleneck** — all traffic for an IP enters through one node.
- **Potentially slow failover** — re-election and ARP/NDP cache updates take time.

L2 is simple and needs no special network gear — just nodes on the same L2 segment.

---

## BGP mode

- Each node establishes a **BGP peering session** with the network routers.
- Nodes **advertise** the service IPs over those sessions and tell routers how to forward traffic.
- **kube-proxy** handles the final hop, getting packets to a specific pod.
- Enables **true load balancing across multiple nodes** and finer traffic control.
- Load balancing is **per-connection**: all packets of a single TCP/UDP session go to one node; spreading happens **between** connections, not within one.

**Limitations:**
- When a node goes down, **active connections to it are broken** (the router re-hashes connections across the remaining nodes).

BGP announcement is backed by **FRR-K8s** (see `components.md`).

---

## L2 vs BGP

| | L2 | BGP |
|---|----|----|
| Discovery | ARP / NDP | BGP routing |
| Traffic entry | one owning node per IP | all nodes (ECMP) |
| Load balancing | single node, then kube-proxy | across nodes, per-connection |
| Network needs | nodes on same L2 segment | BGP-capable routers |
| Failover | re-elect owner (slower) | router re-hashes (faster) |
| Main limit | single-node bottleneck | node loss breaks its connections |

---

## Traffic policies (externalTrafficPolicy)

A service's `externalTrafficPolicy` controls what a node does with traffic once it arrives. Combined with the mode, there are four behaviours:

### L2 — Cluster
Traffic → owning node → **kube-proxy spreads to pods across all nodes**. Even distribution, but an extra hop and the source IP is lost.

### L2 — Local
Traffic → owning node → **only pods on that node** receive it. No second hop, source IP preserved — but only the owning node's local pods serve traffic.

### BGP — Cluster
Every node attracts traffic (ECMP). Each node then applies a **second layer of load balancing** (kube-proxy) to pods **across the whole cluster**. Maximum spreading, extra hop, source IP lost.

### BGP — Local
Nodes only attract traffic **if they run one or more of the service's pods locally**, and deliver only to those local pods. Source IP preserved, no second hop.

| Mode + policy | Entry | Then delivers to |
|---------------|-------|------------------|
| L2 + Cluster | one owning node | all pods (via kube-proxy) |
| L2 + Local | one owning node | local pods only |
| BGP + Cluster | all nodes (ECMP) | all pods (via kube-proxy) |
| BGP + Local | nodes with local pods only | local pods only |

**Trade-off in one line:** `Cluster` spreads traffic most evenly but adds a hop and hides the client's source IP; `Local` preserves the source IP and avoids the extra hop but only delivers to pods on the receiving node.