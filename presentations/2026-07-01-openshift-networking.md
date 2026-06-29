# OpenShift Networking — Internal Presentation

## Part 1 — Fundamentals

**1. OSI model** → [osi-model.md](../networking/00-fundamentals/osi-model.md)

**2. IP & subnetting** → [ip-subnetting.md](../networking/00-fundamentals/ip-subnetting.md)

**3. Switch & router (LAN/WAN)** → [switch-router.md](../networking/00-fundamentals/switch-router.md)

**4. VLAN** → [vlan.md](../networking/00-fundamentals/vlan.md)

**5. VRF** → [vrf.md](../networking/00-fundamentals/vrf.md)

**6. DNS** → [dns.md](../networking/00-fundamentals/dns.md)

**7. Protocols + BGP** → [protocols.md](../networking/00-fundamentals/protocols.md), [bgp.md](../networking/00-fundamentals/bgp.md)

---

## Part 2 — Linux networking

**8. Interfaces & bonds** → [interfaces.md](../networking/01-linux/interfaces.md)

**9. Routing tables** → [routing-tables.md](../networking/01-linux/routing-tables.md)

**10. OVS** → [ovs.md](../networking/01-linux/ovs.md)

**11. NetworkManager / NMState** → [network-manager.md](../networking/01-linux/network-manager.md)

---

## Part 3 — Containers & Kubernetes

**12. Network namespaces & veth** → [network-namespaces.md](../networking/02-container-k8s/network-namespaces.md)

**13. Container networking** → [container-networking.md](../networking/02-container-k8s/container-networking.md)

**14. Pod networking** → [pod-networking.md](../networking/02-container-k8s/pod-networking.md)

**15. Services** → [services.md](../networking/02-container-k8s/services.md)

**16. Ingress** → [ingress.md](../networking/02-container-k8s/ingress.md)

**17. CNI** → [cni.md](../networking/02-container-k8s/cni.md)

**18. Network policy** → [network-policy.md](../networking/02-container-k8s/network-policy.md)

---

## Part 4 — OpenShift networking

**19. OVN-Kubernetes** → [ovn-kubernetes.md](../networking/03-openshift/ovn-kubernetes.md)

**20. OVS bridges** → [ovs-bridges.md](../networking/03-openshift/ovs-bridges.md)

**21. Geneve overlay** → [geneve-overlay.md](../networking/03-openshift/geneve-overlay.md)

**22. Localnet** → [localnet.md](../networking/03-openshift/localnet.md)

**23. NAD & Multus** → [nad-multus.md](../networking/03-openshift/nad-multus.md)

**24. Routes** → [routes.md](../networking/03-openshift/routes.md)

**25. MetalLB** → [metallb.md](../networking/03-openshift/metallb.md)

**26. NMState & NNCP** → [nmstate-nncp.md](../networking/03-openshift/nmstate-nncp.md)

---

## Part 5 — Our architecture

**27. Reference diagram** → (architecture diagram)

**28. Day-1: SiteConfig** → [day1-siteconfig.md](../networking/04-advanced/ocp-reference/day1-siteconfig.md)

**29. Day-2: bridge / VRF / storage / localnet** → [br-vlans](../networking/04-advanced/ocp-reference/day2-br-vlans-bond0.md) · [vrf](../networking/04-advanced/ocp-reference/day2-vrf-vlan.md) · [storage](../networking/04-advanced/ocp-reference/day2-storage-networks.md) · [localnet](../networking/04-advanced/ocp-reference/day2-localnet-nad.md)

**30. Day-2: MetalLB config** → [day2-metallb.md](../networking/04-advanced/ocp-reference/day2-metallb.md)

---

## Part 6 — Advanced

**31. MetalLB deep** → [metallb/](../networking/04-advanced/metallb/README.md)

**32. EgressService · NAD-deep · Gateway API** → [egress-service.md](../networking/04-advanced/egress-service.md) · [nad-multus-deep.md](../networking/04-advanced/nad-multus-deep.md) · [gateway-api.md](../networking/04-advanced/gateway-api.md)
