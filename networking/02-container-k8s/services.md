# Services

## Why Services exist

Pods are **ephemeral** — they come and go, and their IPs change. You can't point clients at a pod IP that won't stay fixed. A **Service** solves this by giving a **stable address** in front of a changing set of pods.

---

## How it works

A Service has a **fixed virtual IP**. It forwards traffic to the pods that match its **label selector**, load-balancing across them. As pods are added or removed, the Service automatically updates which pods it sends to — but its own IP stays constant.

```
client → Service (stable IP) → one of several matching pods (changing)
```

---

## Service types

| Type | What it does |
|------|-------------|
| **ClusterIP** | A stable **internal-only** IP — reachable from inside the cluster (default type) |
| **NodePort** | Opens a fixed **port on every node's IP**, routing to the Service |
| **LoadBalancer** | Gives the Service an **external IP** in front of it (needs a cloud provider, or MetalLB on bare metal) |

These build on each other: a **NodePort** also has a ClusterIP; a **LoadBalancer** also has a NodePort and ClusterIP.

---

## Endpoints

Behind each Service is an **endpoint list** — the set of pods currently backing it. Kubernetes keeps this list updated live as matching pods start and stop, so traffic only goes to healthy, existing pods.

---

## How it's implemented

In a default cluster, **kube-proxy** runs on every node and programs **iptables (or IPVS)** rules to route a Service's ClusterIP to its backing pods and load-balance across them.

> Some CNIs implement this differently — for example **OVN-Kubernetes** handles Service routing in its own dataplane (OVN flows) instead of kube-proxy/iptables. The Service concept is identical; only the mechanism underneath changes. (OCP specifics covered separately.)