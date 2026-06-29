# MetalLB

MetalLB is a **load-balancer implementation for bare-metal Kubernetes clusters**.

## The problem it solves

Kubernetes does not provide a network load balancer for bare-metal clusters. In the cloud, a `Service` of type `LoadBalancer` is fulfilled by the cloud provider; on bare metal there's nothing to fulfil it, so the service stays **pending**. MetalLB fills that gap — it assigns external IPs and makes the network route to them.

## Two features (the core model)

MetalLB does its job through two cooperating features:

- **Address allocation** — hands out IP addresses to services from configured **address pools** (public, private, or both).
- **External announcement** — tells the surrounding network **where each IP lives**, using either **L2** or **BGP**.

## Requirements

- A compatible Kubernetes version
- A cluster network configuration that can coexist with MetalLB
- Some IP addresses for MetalLB to hand out
- **L2 mode** — traffic on port 7946 (TCP & UDP, configurable) allowed between nodes
- **BGP mode** — routers capable of speaking BGP

## Installation methods

MetalLB can be installed via:

- **Manifest** — apply the published YAML directly
- **Kustomize** — overlay-based install
- **Helm** — chart-based install
- **Operator** — managed via an operator (typical on OpenShift)

## In this folder

| File | Covers |
|------|--------|
| `concepts.md` | Address allocation + announcement model |
| `modes.md` | L2 vs BGP, limitations, traffic-policy matrix |
| `components.md` | Controller, Speaker, FRR-K8s |
| `configuration.md` | IPAddressPool, BGPPeer, L2/BGP Advertisement |
| `service-usage.md` | Requesting IPs/pools, traffic policies, dual-stack, IP sharing |
| `advanced.md` | VRF peering, BFD, node/peer selection, communities, graceful restart |
| `troubleshooting.md` | Controller vs speaker, status commands |