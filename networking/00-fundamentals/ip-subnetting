# IP Addressing & Subnetting

## What an IP address is

An IPv4 address is a **32-bit** number written as four 8-bit **octets** separated by dots:

```
192.168.1.10
```

Each octet ranges 0–255. (IPv6 is the newer **128-bit** scheme written in hex — covered separately.)

---

## Network portion vs host portion

Every IP splits into two parts:

- **Network portion** — identifies *which network* the address belongs to
- **Host portion** — identifies *which device* within that network

The **subnet mask** decides where the split falls.

---

## Subnet mask & CIDR notation

The mask marks the network bits with `1`s and host bits with `0`s.

```
255.255.255.0   =  /24   (first 24 bits = network)
```

CIDR notation writes the address with the prefix length:

```
192.168.1.10/24
```

The number after `/` = how many bits are the network portion. The remaining bits are for hosts.

---

## Prefix reference table

| CIDR | Subnet mask | Host bits | Usable hosts |
|------|-------------|-----------|--------------|
| /24 | 255.255.255.0   | 8 | 254 |
| /25 | 255.255.255.128 | 7 | 126 |
| /26 | 255.255.255.192 | 6 | 62 |
| /27 | 255.255.255.224 | 5 | 30 |
| /28 | 255.255.255.240 | 4 | 14 |
| /29 | 255.255.255.248 | 3 | 6 |
| /30 | 255.255.255.252 | 2 | 2 |

---

## How hosts are calculated

```
usable hosts = 2^(host bits) − 2
```

Subtract 2 because the first address is the **network address** and the last is the **broadcast address** — neither can be assigned to a device.

**Example — /25:**
- 32 − 25 = 7 host bits
- 2^7 = 128 total addresses
- 128 − 2 = **126 usable hosts**

---

## Network address, broadcast, usable range

**Worked example — `10.106.4.0/25`:**

- Total addresses: 128 (`10.106.4.0` – `10.106.4.127`)
- **Network address:** `10.106.4.0` (first)
- **Broadcast address:** `10.106.4.127` (last)
- **Usable range:** `10.106.4.1` – `10.106.4.126` (126 hosts)

The next subnet would begin at `10.106.4.128/25`.

---

## Public vs private IP

- **Public IP** — globally unique, routable on the internet. Assigned by ISPs / registries.
- **Private IP** — used inside local/internal networks only. Not routable on the internet. Multiple organisations reuse the same private ranges. To reach the internet, private addresses are translated to a public one via **NAT**.

---

## Private IP ranges (RFC 1918)

| Class | Range | CIDR |
|-------|-------|------|
| A | 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 |
| B | 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 |
| C | 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 |

(The A/B/C labels come from the older classful system; today addressing is classless/CIDR, but the names are still used informally.)

---

## Special cases

| Range | Purpose |
|-------|---------|
| `127.0.0.0/8` | **Loopback** — the host itself (`127.0.0.1` = localhost) |
| `169.254.0.0/16` | **Link-local** — auto-assigned when no DHCP; only valid on the local link |
| `/30` | Classic **point-to-point** link — 4 addresses, 2 usable (two router ends) |
| `/31` | Point-to-point link with **no network/broadcast** — both addresses usable (2 hosts) |
| `/32` | A **single host** — one exact address (e.g. a host route advertised on its own) |