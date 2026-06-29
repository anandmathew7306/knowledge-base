# Troubleshooting

## Key principle

MetalLB's job is only to **attract LoadBalancer traffic to a node**. Once traffic lands on a node, the **CNI** takes over. So:

- Reaching the LB IP **from a node** proves the *CNI* works — not MetalLB.
- **Pinging the IP won't work** — you must actually **access the service** to test it (and confirm the app itself is healthy).

---

## Which component to check

| Symptom | Check |
|---------|-------|
| Service is **not getting an IP** | **controller** (does IP allocation) |
| Service **has an IP but isn't advertised** | **speaker** (does announcement) |

---

## Configuration validity

MetalLB's config is the **whole set** of CRs (pools, advertisements, peers) validated together — a single piece isn't valid/invalid on its own. If the new config is invalid, MetalLB **marks it stale and keeps the last valid one**. To check:

- Look for `failed to parse the configuration` errors in the component logs.
- Watch the `metallb_k8s_client_config_stale_bool` Prometheus metric (true = running on stale config).

---

## IP assignment issues (controller)

A service may not get an IP if:

- No **IPAddressPool** is compatible (including selectors).
- The service is **dual-stack** but no pool offers both IPv4 and IPv6.
- It requests a **specific IP** but no pool provides it (or selectors don't match).
- It requests a **shared IP** without respecting the IP-sharing rules.

---

## Advertisement issues (speaker)

`kubectl describe svc <name>` shows which speaker(s) are announcing. A speaker **won't** advertise if:

- There are **no active endpoints** backing the service.
- `externalTrafficPolicy: Local` and **no running endpoints on that speaker's node**.
- No L2/BGP advertisement **matches the node** (when node selectors are set).
- The node reports **network not available**.
- The node carries the `node.kubernetes.io/exclude-from-external-load-balancers` label (MetalLB honours it). Use `--ignore-exclude-lb` to override.

---

## L2 checks

- `arping <lb-ip>` from a host on the same L2 subnet shows which MAC MetalLB returns for the IP.
- `tcpdump ... arp` on the elected node confirms ARP requests arrive and replies leave.
- **Multiple MACs for one IP** → split-brain (two speakers replying), the IP also on another interface, or the CNI answering ARP.
- Anti-MAC-spoofing on the fabric is a common cause of blocked ARP.

---

## BGP checks

On a speaker that should advertise, confirm:

1. The **BGP session is up** — metric `frrk8s_bgp_session_up` (or `metallb_bgp_session_up` in native mode).
2. The **IP is being advertised**.

With FRR, query the FRR container:

- `vtysh show running-config` — current FRR config
- `vtysh show bgp neigh <ip>` — session status (`Established` = healthy)
- `vtysh show ipv4 / ipv6` — advertised routes

If the session won't establish: check **ASNs and passwords**, the FRR logs, `tcpdump` the BGP port, and the **router's routing table**.

---

## Announced but unreachable

Use `tcpdump` to localise where traffic stops:

- **Doesn't reach the node** → network infrastructure or MetalLB issue.
- **Reaches the node but not the pod** → likely a **CNI** issue.

**Reverse path filtering (`rp_filter`)** is a common BGP gotcha: traffic arrives on a non-default interface, the node has no route back to the client on that interface, and packets get dropped. Fixes include static routes, the FRR-K8s variant (so the fabric sends return routes), or source-based routing.

**Intermittent traffic** often comes from: a changing entry point (ECMP in BGP), an L2 leader bouncing between nodes, or some endpoints restarting/misbehaving. Narrow it down by limiting the advertisement to **one node** and the service to **one pod**.

---

## Status commands

```
kubectl get servicel2statuses  -n metallb-system
kubectl get servicebgpstatuses -n metallb-system
```

For deeper debugging, set the controller/speaker **loglevel to debug**, and collect the CRs, service, and endpointslices YAML when filing a bug report.