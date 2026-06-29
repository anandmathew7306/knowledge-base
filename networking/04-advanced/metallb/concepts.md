# Concepts

MetalLB works through two cooperating features: **address allocation** and **external announcement**.

---

## Address allocation

MetalLB hands out IP addresses to LoadBalancer services from configured **address pools**.

- Pools can hold **public, private, or both** kinds of addresses, depending on the environment.
- Addresses can be defined by **CIDR** (`192.0.2.0/24`), by **range** (`192.0.2.1-192.0.2.10`), or a mix.
- Both **IPv4 and IPv6** can be assigned (dual-stack).
- A pool can **auto-assign** (any service gets an IP from it automatically) or be **explicit-only** (services must request that pool by name).
- Multiple pools can coexist.

When a service of type LoadBalancer is created, MetalLB picks an available address from an eligible pool and assigns it as the service's external IP.

---

## External announcement

Allocating an IP isn't enough — the surrounding network has to know **where that IP lives** so traffic can reach the right node. This is **announcement**, and MetalLB does it in one of two modes:

- **L2 (ARP/NDP)** — one node answers for the IP on the local network.
- **BGP** — nodes advertise the IP as a route to upstream routers.

BGP announcement is handled internally by **FRR** (the routing software bundled with the speaker). *(Modes are detailed in `modes.md`; components including FRR in `components.md`.)*

---

## Split of responsibility

The two features are handled by two different components:

| Feature | Component |
|---------|-----------|
| Address allocation | **Controller** — assigns IPs to services |
| External announcement | **Speaker** — announces the IPs via L2 or BGP |

The controller decides *which IP*; the speaker makes that IP *reachable*. *(Detailed in `components.md`.)*