# Pod Networking

## What a pod is (networking view)

A **pod** is one or more containers that **share a single network namespace**. Because they share one namespace, all containers in a pod share the **same IP** and reach each other over **`localhost`**.

---

## IP per pod

Every pod gets its **own IP**, unique across the whole cluster — the **IP-per-pod** model. The IP belongs to the pod, not to individual containers inside it.

---

## Cluster network

The whole cluster draws pod IPs from one large range called the **cluster network** (or pod network), e.g. `10.0.0.0/14`.

Each **node** is given a **slice** of that range — a smaller subnet (e.g. a `/23`) — and assigns pod IPs from its own slice:

```
cluster network   10.0.0.0/14
├── node 1 slice  10.2.0.0/23   → pods on node 1 get IPs from here
├── node 2 slice  10.2.2.0/23   → pods on node 2 get IPs from here
└── ...
```

Slicing the range per node guarantees every pod gets a **unique** cluster-wide IP with no collisions.

---

## The Kubernetes networking model

Kubernetes mandates three rules for pod networking:

1. Every pod can reach every other pod **without NAT**
2. Pods see each other's **real IPs** (no address translation)
3. This holds **across all nodes**, not just within one host

This is the key difference from the plain container bridge model, where containers are only directly reachable on the same host.

---

## Pod-to-pod, same node

Both pods are on the same node's virtual switch/bridge, so traffic goes directly between them on that node — no need to leave the host.

---

## Pod-to-pod, different nodes

Traffic must cross from one node to another. **How** depends on the CNI plugin — commonly an **overlay/tunnel** that wraps pod traffic to carry it between nodes, sometimes plain routing. (The mechanism is CNI-specific.)

---

## How a pod gets wired

When a pod starts, the CNI plugin does the wiring (recapping the namespace model):

```
pod namespace → veth pair → node bridge/switch → CNI assigns IP + sets up routes
```

Kubernetes itself doesn't implement this — it **delegates to the CNI** (see the CNI note).

---

## Pod IPs are ephemeral

Pods are created and destroyed constantly, and each new pod gets a **new IP**. You can't rely on a pod's IP staying fixed. This is exactly **why Services exist** — to give a stable address in front of changing pods (see the services note).