# Switch & Router

## Switch (L2)

- Connects devices within the **same network**
- Forwards **frames** by **MAC address**
- Learns which MAC sits on which port and stores it in a **MAC table**
- Doesn't understand IP addresses

## Router (L3)

- Connects **different networks** together
- Forwards **packets** by **IP address**
- Uses a **routing table** to decide the path
- This is what lets one subnet reach another, or reach the internet

---

## Side by side

| | Switch | Router |
|---|--------|--------|
| OSI layer | L2 | L3 |
| Forwards by | MAC address | IP address |
| Connects | same network | different networks |
| Decision table | MAC table | routing table |

---

## LAN vs WAN

- **LAN (Local Area Network)** — devices in one location / one network, connected together by switches.
- **WAN (Wide Area Network)** — connects LANs across distance: linked sites, or the internet itself. Routers sit at the boundary between networks.

Put simply: a **switch** moves traffic *inside* a LAN; a **router** moves traffic *between* LANs, or from a LAN out to a WAN.

---

## Default gateway

The **default gateway** is the router IP a device sends traffic to when the destination is **outside its own subnet**.

**Example:** host `10.106.4.7` (in `10.106.4.0/25`) wants to reach something outside that subnet → it sends the packet to its gateway `10.106.4.1`, and the router forwards it onward.

---

## How they work together

- **Destination in the same subnet** → the switch handles it directly (L2).
- **Destination in a different subnet** → traffic goes to the default gateway → the router forwards it (L3).