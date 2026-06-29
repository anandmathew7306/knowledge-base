# Network Namespaces & veth

## What a network namespace is

A **network namespace** is an isolated networking compartment inside one Linux host. Each namespace has its **own** interfaces, its **own** IP addresses, and its **own** routing table — completely separate from the host and from other namespaces.

(It's a Linux kernel feature — the same kernel, partitioned into independent network stacks.)

---

## Why it exists

Network namespaces let **many isolated network stacks run on one host**. This is the core trick behind containers: each container gets its **own** namespace, so it has its **own** `eth0` and its **own** IP, isolated from every other container — even though they all share one physical machine.

```
Host
├── default namespace   (the host's own networking)
├── namespace A          (container A — own eth0, own IP)
└── namespace B          (container B — own eth0, own IP)
```

---

## The isolation problem

A namespace is isolated **by design** — so on its own it can't reach anything outside itself. It needs a connection to the outside. That connection is a **veth pair**.

---

## veth pair

A **veth pair (virtual ethernet)** is a virtual cable with **two ends**. Whatever goes in one end comes out the other. It's used to connect a namespace to the outside world:

- **One end goes inside the namespace** → becomes the container's `eth0`
- **The other end stays outside** → plugs into a **bridge** on the host

```
container namespace                 host
   eth0 ─────── veth pair ───────  plugs into bridge ──── out to network
```

---

## The full picture

Putting it together, this is how a container gets working networking:

```
namespace (isolated)
     │  veth pair (the cable)
     ▼
bridge on host (shared switch)
     │
     ▼
out to the physical network
```

1. Container starts → gets its own network namespace (isolated)
2. A veth pair is created
3. One end goes inside the namespace → becomes `eth0`
4. The other end attaches to a bridge on the host
5. The bridge connects onward to the network
6. The container now has its own IP and can communicate

---

## Tie to containers and pods

Every container runs in its own network namespace, connected out by a veth pair to a bridge — exactly the setup above. In Kubernetes this is how pods connect to the node's network. (Pod-specific details follow in the pod networking note.)