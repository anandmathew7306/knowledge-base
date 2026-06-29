# MetalLB (OpenShift)

## The problem

A `Service` of type **LoadBalancer** expects a cloud provider to hand out an external IP. On **bare metal** there's no cloud provider — so without help, the service stays **pending** forever, never getting an external IP.

---

## What MetalLB does

**MetalLB** provides external IPs for LoadBalancer services on bare metal. It assigns an IP from a configured pool and makes the network route traffic to it. It's the bare-metal equivalent of a cloud **NLB** (L4).

---

## Two responsibilities

- **IP assignment** — picks an IP from a configured pool and assigns it to the service.
- **Advertisement** — tells the network **where that IP lives**, so external traffic can find its way to the right node.

---

## Modes

| Mode | How it advertises |
|------|-------------------|
| L2 | ARP-based — one node answers for the IP |
| BGP | Advertises the IP as a route to an upstream router |

Our clusters use **BGP mode**.

---

## Where it sits

MetalLB operates at **L4** — it gives a service a raw IP for any TCP/UDP traffic. This is different from **Routes**, which work at **L7** (HTTP/HTTPS, hostname routing). Use MetalLB for non-HTTP services (databases, Kafka, DHCP); use Routes for web apps. (See the routes note.)

---

> Detailed MetalLB — BGP peers, IP address pools, advertisements, VRF integration, and the full configuration flow — is covered in the advanced notes.