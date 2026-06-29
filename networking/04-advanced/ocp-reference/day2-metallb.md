# Day 2 — MetalLB (annotated)

The three MetalLB objects that work **together** to assign and advertise a LoadBalancer IP. They reference each other by name:

```
IPAddressPool      defines the IPs MetalLB can hand out
      ▲ referenced by
BGPAdvertisement   "advertise these pools to these peers"
      ▼ references
BGPPeer            the BGP sessions to the upstream router
```

This set repeats per VRF/network — each VRF has its own pool, peers, and advertisement of the same shape.

---

## 1. IPAddressPool

Defines the range of IPs MetalLB can assign to LoadBalancer services.

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: oss-pool-ipv4
  namespace: metallb-system
spec:
  addresses:
    - 192.0.2.128/25        # the assignable IP range (CIDR, or a-b range)
  autoAssign: false         # don't auto-hand-out; services must request this pool explicitly
  avoidBuggyIPs: true       # skip .0 and .255 (network/broadcast) addresses
```

---

## 2. BGPPeer

Defines one BGP session to the router. One object per peer.

```yaml
apiVersion: metallb.io/v1beta2
kind: BGPPeer
metadata:
  name: peer-oss-b-ipv4
  namespace: metallb-system
spec:
  myASN: 64500              # this cluster's ASN
  peerASN: 64501            # the router's ASN (different → eBGP)
  peerAddress: 192.0.2.3    # router's IP on this VLAN (same subnet as the node's VRF interface)
  peerPort: 179             # BGP TCP port
  password: REDACTED        # MD5 session auth
  vrf: vrf100               # bind this session to a specific VRF's routing table
  disableMP: false          # multiprotocol BGP enabled
```

> Peers come in sets per VRF — typically **a + b** (two routers, redundancy) × **IPv4 + IPv6** (dual-stack) = 4 peers. Only `name`, `peerAddress`, `vrf`, and address family change between them.

---

## 3. BGPAdvertisement

Links pool(s) to peer(s) — tells MetalLB what to advertise and where.

```yaml
apiVersion: metallb.io/v1beta1
kind: BGPAdvertisement
metadata:
  name: bgpadvertisement-oss-ipv4
  namespace: metallb-system
spec:
  aggregationLength: 32     # advertise as /32 host routes (one exact IP each)
  localPref: 100            # BGP local preference (higher = more preferred)
  ipAddressPools:           # which pool(s) to advertise
    - oss-pool-ipv4
    - oss-pool-anycast-ipv4
  peers:                    # advertise them to these peers
    - peer-oss-a-ipv4
    - peer-oss-b-ipv4
```

---

## What each does

| Object | Role |
|--------|------|
| IPAddressPool | The IPs MetalLB can assign to services |
| BGPPeer | A BGP session to the router (per VRF, per router, per address family) |
| BGPAdvertisement | Connects pools → peers: what to advertise, to whom, how (/32, localPref) |

---

## The end-to-end flow

```
service (type: LoadBalancer, requests oss-pool-ipv4)
   → MetalLB controller assigns an IP from the pool
   → speaker adds that IP as /32 on the node interface
   → BGPAdvertisement says "advertise it to peer-oss-a/b"
   → BGPPeer sessions (in vrf100) announce the /32 to the router
   → router routes external traffic to the service IP
```

---

## Repeats per VRF/network

Each VRF/network has its own copy of all three, differing only in:

| Field | Varies |
|-------|--------|
| pool `addresses` | the VRF's IP range |
| peer `peerAddress` / `vrf` | the router IP + VRF |
| advertisement `ipAddressPools` / `peers` | which pool ↔ which peers |

Structure and field set are identical across them.