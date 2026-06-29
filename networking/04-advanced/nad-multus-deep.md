# NAD & Multus (Deep Dive)

> Basics — what Multus and a NAD are, and how a workload attaches — are in `03-openshift/nad-multus.md`. This note goes deeper: NAD config anatomy, plugin types, and IPAM.

---

## NAD config anatomy

A NAD's real content is its `spec.config` — a **CNI config in JSON**. The key field is `type`, which selects **which CNI plugin** wires up the interface; the remaining fields are plugin-specific.

```json
{
  "cniVersion": "0.3.1",
  "type": "<plugin>",          // which CNI plugin to use
  ...                          // plugin-specific fields
  "ipam": { "type": "<ipam>" } // how this interface gets an IP
}
```

So a NAD is really "**which plugin + what config + which IPAM**" for one extra interface.

---

## Plugin types

| `type` | What it does |
|--------|-------------|
| `ovn-k8s-cni-overlay` | OVN-Kubernetes — used for localnet/overlay secondary networks |
| `ipvlan` | Attaches to a parent interface; pod gets its own IP, **shares the parent's MAC** |
| `macvlan` | Attaches to a parent interface; pod gets its own IP **and its own MAC** |
| `bridge` | Attaches the pod to a Linux bridge |

---

## ipvlan vs macvlan

Both put a workload directly on a network via a parent interface, without the OVN overlay. The difference is the MAC:

| | ipvlan | macvlan |
|---|--------|---------|
| Own IP | yes | yes |
| Own MAC | **no** — shares parent's MAC | **yes** — own unique MAC |
| Looks like (on the wire) | parent device, extra IPs | a fully separate device |
| Good when | the network limits MACs per port, or MAC count matters | each workload must appear as its own host/MAC |

---

## IPAM (IP address management)

The **primary** pod network gets IPs from the CNI (OVN). **Secondary** networks (macvlan/ipvlan/bridge) need their **own** IPAM. Options:

| IPAM `type` | Behaviour |
|-------------|-----------|
| `static` | Fixed IP written into the config/annotation |
| `dhcp` | Gets an IP from a DHCP server on that network |
| `whereabouts` | **Cluster-wide** automatic allocation from a range |

### whereabouts

**whereabouts** is an IPAM plugin that assigns IPs for secondary interfaces **across the whole cluster** — it tracks allocations cluster-wide so two pods on different nodes don't get the same IP. This solves the gap that plain `static`/`dhcp` leave for secondary networks at scale. (A `whereabouts-shim` NAD is part of the standard install plumbing.)

---

## Attaching a workload

A pod/VM references the NAD by name in the `k8s.v1.cni.cncf.io/networks` annotation, and Multus attaches that extra interface alongside the primary one. (See `03-openshift/nad-multus.md`.)