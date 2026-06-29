# OpenShift Routes

## What a Route is

A **Route** is OpenShift's way to expose an HTTP/HTTPS service to the outside world under a **hostname**. It's OpenShift's native **L7 entry point** for web traffic.

---

## Route vs Ingress

Routes are OpenShift's **original** mechanism for exposing services — they predate the Kubernetes Ingress resource. OpenShift supports **both**: when you create an Ingress object, OpenShift converts it into a Route under the hood. Same job; the Route is the native form.

---

## The OpenShift Router

The **OpenShift Router** is the component that does the actual work — an **HAProxy-based** ingress controller running in the cluster. It reads Route definitions and proxies incoming traffic to the correct service. It plays the L7 role (the equivalent of a cloud ALB).

---

## The full chain — user to pod

```
user
  → DNS resolves hostname → external IP
  → Router's external IP (LoadBalancer / VIP)
  → OpenShift Router (HAProxy)        ← matches hostname/path, terminates/handles TLS
  → Service (ClusterIP)
  → Pod
```

One external entry point (the Router) serves many Routes, fanning out by hostname/path.

---

## TLS handling

Routes support several TLS modes:

| Mode | Behaviour |
|------|-----------|
| edge | TLS terminated **at the Router**; plain traffic to the pod |
| passthrough | TLS passed **straight through** to the pod (pod handles certs) |
| re-encrypt | Terminated at the Router, then **re-encrypted** to the pod |

---

## Route vs MetalLB

| | Route | MetalLB |
|---|-------|---------|
| Layer | L7 (HTTP/HTTPS) | L4 (any TCP/UDP) |
| Routing | by hostname / path | by IP + port |
| Via | OpenShift Router | dedicated service IP |
| Use for | web apps, APIs | databases, Kafka, DHCP, raw TCP/UDP |

Use a **Route** when you need HTTP-level routing for a web app; use **MetalLB** when a service needs a raw L4 IP for a non-HTTP protocol.