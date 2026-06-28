# OSI Model

A 7-layer framework describing how data moves across a network. Each layer has one job and talks only to the layers directly above and below it.

| # | Layer | Job | Example |
|---|-------|-----|---------|
| 7 | Application | User-facing protocols | HTTP, DNS, SSH |
| 6 | Presentation | Format, encrypt, compress | TLS, JPEG |
| 5 | Session | Open / maintain / close sessions | login session, TLS session |
| 4 | Transport | End-to-end delivery, ports | TCP, UDP |
| 3 | Network | Routing between networks, IP addressing | IP, BGP, routers |
| 2 | Data Link | Framing, MAC addressing, switching | Ethernet, VLAN, switches |
| 1 | Physical | Bits on the wire | cables, NICs |

---

## Encapsulation

Each layer wraps the data from the layer above with its own header. Going **down** the stack on send:

```
L7  data
L4  segment   (+ TCP/UDP header — ports)
L3  packet    (+ IP header — source/destination IP)
L2  frame     (+ Ethernet header — source/destination MAC, VLAN tag)
L1  bits      (on the wire)
```

The receiver unwraps in reverse (de-encapsulation). This is exactly what **Geneve** does later — it wraps a whole frame inside another packet to tunnel pod traffic between nodes.

---

## TCP/IP Model

The model actually used in practice. It collapses the 7 OSI layers into 4:

| TCP/IP Layer | Maps to OSI | Examples |
|--------------|-------------|----------|
| Application | L5–L7 | HTTP, DNS, SSH, TLS |
| Transport | L4 | TCP, UDP |
| Internet | L3 | IP, BGP |
| Network Access | L1–L2 | Ethernet, VLAN |

OSI is the teaching/reference model; TCP/IP is what the internet runs on.
