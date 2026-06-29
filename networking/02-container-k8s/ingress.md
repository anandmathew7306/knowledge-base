# Ingress

## The problem

A LoadBalancer Service gives **each** service its own external IP. With many services that means many external IPs — and it offers no HTTP-level routing (no hostnames, no URL paths). Wasteful and limited.

---

## What Ingress is

**Ingress** is a single entry point for **HTTP/HTTPS** traffic that routes to different services based on **hostname** and **URL path**. One external IP can serve many services behind it.

---

## The full chain — user to pod

Ingress doesn't stand alone — it sits **behind** a LoadBalancer that exposes it. The full path from a user to a pod:

```
User
  │  DNS resolves hostname → external IP
  ▼
External IP  (fronting the ingress controller)
  ▼
LoadBalancer Service   (MetalLB on bare metal, or a cloud LB)
  ▼
Ingress Controller pods   (NGINX / HAProxy — does the L7 routing)
  │  matches hostname / path against Ingress rules
  ▼
Backend Service (ClusterIP)
  ▼
Pod
```

The efficiency: **one** external IP (the LoadBalancer fronting the ingress controller) serves **many** services, instead of one LoadBalancer IP per service.

---

## How it works

Ingress has two parts that work together:

- **Ingress resource** — the routing rules ("host `a.com` → service A", "path `/api` → service B")
- **Ingress controller** — the actual proxy (NGINX, HAProxy) that reads those rules and routes the traffic

Rules alone do nothing — an ingress controller must be running to act on them.

---

## Example routing

```
a.example.com      → service-a
b.example.com      → service-b
example.com/api    → api-service
```

All three arrive at the same external IP; the controller routes them by hostname/path.

---

## L7 vs L4

- **Ingress = L7** — HTTP-aware: routes on hostnames and paths, can terminate TLS.
- **LoadBalancer Service = L4** — only knows IP and port, no HTTP awareness.

That's why Ingress sits behind a LoadBalancer: the LB gets traffic *to* the cluster (L4), and Ingress decides *which service* it goes to (L7).

---

## Bridge to OpenShift

OpenShift has its own equivalent called **Routes** (which predates Ingress), and also supports the standard Ingress resource. Covered in the OpenShift notes.