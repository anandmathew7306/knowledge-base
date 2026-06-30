# OpenShift Networking — Internal Presentation

> Goal: understand our cluster's network architecture. ~30 min. 

## Part 1 — Foundations (the vocabulary)

**1. Switch & router (LAN/WAN)** → [switch-router.md](../networking/00-fundamentals/switch-router.md)

**2. VLAN** → [vlan.md](../networking/00-fundamentals/vlan.md)

**3. VRF** → [vrf.md](../networking/00-fundamentals/vrf.md)

**4. BGP** → [bgp.md](../networking/00-fundamentals/bgp.md)

**5. Interfaces & bonds** → [interfaces.md](../networking/01-linux/interfaces.md)

---

## Part 2 — How it's configured

**6. NMState / NNCP** → [network-manager.md](../networking/01-linux/network-manager.md) · [nmstate-nncp.md](../networking/03-openshift/nmstate-nncp.md)

---

## Part 3 — Cluster dataplane

**7. OVN-Kubernetes** → [ovn-kubernetes.md](../networking/03-openshift/ovn-kubernetes.md)

**8. OVS bridges** → [ovs-bridges.md](../networking/03-openshift/ovs-bridges.md)

**9. Geneve overlay** → [geneve-overlay.md](../networking/03-openshift/geneve-overlay.md)

**10. Localnet** → [localnet.md](../networking/03-openshift/localnet.md)

**11. MetalLB** → [metallb.md](../networking/03-openshift/metallb.md)

---

## Part 4 — Our architecture (the focus)

**12. Reference diagram** → (architecture diagram — draw.io)

**13. Day-1: SiteConfig** → [day1-siteconfig.md](../networking/04-advanced/ocp-reference/day1-siteconfig.md)

**14. Day-2: bridge / VRF / storage / localnet** → [br-vlans](../networking/04-advanced/ocp-reference/day2-br-vlans-bond0.md) · [vrf](../networking/04-advanced/ocp-reference/day2-vrf-vlan.md) · [storage](../networking/04-advanced/ocp-reference/day2-storage-networks.md) · [localnet](../networking/04-advanced/ocp-reference/day2-localnet-nad.md)

**15. Day-2: MetalLB config** → [day2-metallb.md](../networking/04-advanced/ocp-reference/day2-metallb.md)