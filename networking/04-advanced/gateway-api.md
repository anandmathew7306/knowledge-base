# Gateway API

## The problem

**Ingress** is limited: it's HTTP-focused, anything advanced needs vendor-specific annotations, and it crams everything into one resource type. There's also no clean separation between the team that runs the infrastructure and the teams that define routes. **Gateway API** was designed to replace it.

---

## What Gateway API is

**Gateway API** is a newer, more expressive Kubernetes standard for getting traffic into a cluster. It is **role-oriented** (different resources for different responsibilities) and **protocol-aware** (not just HTTP). It aims to be the long-term successor to Ingress.

---

## Core resources

| Resource | What it is | Who owns it (typically) |
|----------|-----------|-------------------------|
| **GatewayClass** | The implementation/controller type (which Gateway implementation to use) | cluster operator |
| **Gateway** | An actual entry point — listeners, ports, protocols | infrastructure team |
| **HTTPRoute / TCPRoute / …** | Routing rules attached to a Gateway | application teams |

The split lets the infra team own the **Gateway** (the entry point) while app teams own their **Routes** — without editing one shared object.

---

## vs Ingress / Route

| | Ingress / Route | Gateway API |
|---|-----------------|-------------|
| Protocols | mainly HTTP/HTTPS | HTTP, TCP, TLS, gRPC, … |
| Advanced features | vendor annotations | first-class fields |
| Role separation | one resource | Gateway (infra) vs Routes (apps) |
| Maturity | established | newer, the emerging standard |

---

## Status

Gateway API is **emerging** and is intended to replace Ingress over time. OpenShift supports it via its Gateway API implementation, alongside the existing Routes. For most current clusters, Routes/Ingress are still the norm; Gateway API is where the ecosystem is heading.