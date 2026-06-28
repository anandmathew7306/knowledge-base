# Protocols

A growing reference of common networking protocols. Add more over time.

---

## TCP

Connection-oriented, **reliable**, ordered delivery. Establishes a connection with a **3-way handshake** (`SYN → SYN-ACK → ACK`) before sending data, and retransmits anything lost. Used when delivery must be guaranteed (web, databases, SSH).

## UDP

Connectionless — **no handshake, no delivery guarantee**. Lower overhead and faster. Used where speed matters more than guaranteed delivery, or where the app handles reliability itself (DNS queries, DHCP, streaming, VoIP).

## TCP vs UDP

| | TCP | UDP |
|---|-----|-----|
| Connection | Yes (handshake) | No |
| Reliable | Yes (retransmits) | No |
| Ordered | Yes | No |
| Overhead | Higher | Lower |
| Use when | Delivery must be guaranteed | Speed/low overhead matters |

---

## ICMP

Control and **diagnostic** messages, not application data. Reports errors and reachability. `ping` and `traceroute` rely on it.

## ARP

Resolves an **IP address to a MAC address** on the local network (L2). This is the step that lets the first frame actually leave toward its next hop.

---

## Common application protocols

| Protocol | Purpose |
|----------|---------|
| HTTP / HTTPS | Web traffic (HTTPS = HTTP over TLS) |
| DNS | Name → IP resolution |
| DHCP | Automatic IP address assignment |
| SSH | Secure remote shell access |
| NTP | Time synchronisation |
| TLS | Encryption layer beneath HTTPS and others |

## BGP

Routing protocol that exchanges routes between networks / autonomous systems. *(Covered in detail in `bgp.md`.)*

---

## Ports

L4 uses **port numbers** to identify which service traffic belongs to. Well-known ports:

| Port | Protocol |
|------|----------|
| 22 | SSH |
| 53 | DNS |
| 67 / 68 | DHCP |
| 80 | HTTP |
| 123 | NTP |
| 179 | BGP |
| 443 | HTTPS |

---

## Protocol → OSI layer

| Protocol | OSI layer |
|----------|-----------|
| HTTP, DNS, SSH, DHCP, NTP | L7 — Application |
| TLS | L6 — Presentation |
| TCP, UDP | L4 — Transport |
| IP, ICMP, BGP* | L3 — Network |
| ARP, Ethernet, VLAN | L2 — Data Link |

\* BGP rides on top of TCP (port 179) but is a routing protocol operating at the network layer.