# Container Networking

## Recap

A container is just a process running in its **own network namespace**, connected out by a **veth pair** to a **bridge** on the host. (See the network namespaces note.) This applies to all container engines — Docker, Podman, containerd, CRI-O. Docker is used here as the familiar example.

---

## The default bridge model

By default, a container engine wires up networking like this:

1. The engine creates a **bridge** on the host (Docker calls it `docker0`).
2. Each container gets its **own namespace + veth pair** — one end inside the container (`eth0`), the other on the bridge.
3. Containers on the **same bridge** can talk to each other directly.
4. The bridge connects to the host's network so containers can reach outside.

```
container A ──veth──┐
                    ├── bridge (docker0) ── host network ── outside
container B ──veth──┘
```

---

## Getting an IP

The engine assigns each container an IP from a **private subnet it manages** (e.g. `172.17.0.0/16`). These IPs are local to the host — not visible on the wider network.

---

## Reaching the outside (NAT)

Because container IPs are private, outbound traffic is **NAT'd** behind the host's IP — the container's private source IP is rewritten to the host's address on the way out. (See the iptables/NAT note.)

---

## Port publishing

Container IPs aren't reachable from outside the host by default. To expose a container, a **host port is mapped to a container port** (DNAT):

```
host :8080  →  container :80
```

A client connects to the host's IP on `:8080`, and the engine forwards it to the container's `:80`.

---

## Container-to-container

- **Same host / same bridge** → they reach each other directly through the bridge.
- This is the foundation Kubernetes builds on — and improves, so that pods are reachable across the **whole cluster**, not just one host.

---

## Network modes

A container can run in different networking modes. The behaviour is general to containers; the names in brackets are Docker's:

| Mode (Docker name) | Behaviour |
|--------------------|-----------|
| Isolated (`bridge`) | Own namespace + veth + bridge — the default |
| Shared (`host`) | Uses the host's namespace directly — no isolation, no veth |
| None (`none`) | Namespace created, but no connectivity |

---

## Bridging to Kubernetes

Kubernetes does **not** use this default per-host bridge model. It requires every pod to be routable across the entire cluster, and delegates the wiring to a **CNI plugin**. (See the pod networking and CNI notes.)