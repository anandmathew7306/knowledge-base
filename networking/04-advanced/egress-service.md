# EgressService

## The problem

By default, when a pod sends traffic **out** of the cluster, its source IP is **SNAT'd to the node's IP**. So external systems see the *node's* address, and which node depends on where the pod is scheduled — the source IP isn't stable or predictable.

This is a problem when an external system **filters by source IP** (firewalls, allow-lists). It can't allow-list an unpredictable, changing node IP.

---

## What EgressService does

**EgressService** lets traffic from a LoadBalancer service's pods **leave** the cluster using the service's **LoadBalancer IP** as the source address — instead of the node IP. The egress traffic then carries a **stable, predictable** source IP.

---

## Why it matters

External systems that allow-list or route by source IP need a fixed value. With EgressService:

- Outbound traffic from the service's pods uses the LB IP as its source.
- Firewalls/allow-lists can be configured against that one stable IP.
- Return traffic comes back to the right place.

This is common in OSS / telecom integrations where the far end enforces source-IP allow-lists.

---

## Assigned host

For a given EgressService, **one node** handles the egress at a time — shown as the **Assigned Host**. If that node goes down, the role **fails over** to another node, so egress keeps working with the same source IP.

---

## Relation to MetalLB

EgressService pairs with MetalLB symmetrically:

```
MetalLB        brings external traffic IN   → to the service's LB IP
EgressService  makes pod traffic go OUT      → using that same LB IP as source
```

Together they give a service a **single, consistent IP for both inbound and outbound** traffic.

---

## Note

EgressService is an **OVN-Kubernetes** feature (`egressservices.k8s.ovn.org`), configured per service.