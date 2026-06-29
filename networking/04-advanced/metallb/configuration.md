# Configuration

MetalLB is configured with a few custom resources. Pools define the IPs; advertisements announce them; peers define BGP sessions.

---

## IPAddressPool

Defines the addresses MetalLB can assign. Supports CIDR, ranges, and dual-stack.

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.10.0/24                  # CIDR
    - 192.168.9.1-192.168.9.5          # range
    - fc00:f853:0ccd:e799::/124        # IPv6
```

---

## L2Advertisement

Announces a pool over **L2** (ARP/NDP).

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
    - first-pool
```

---

## BGPPeer

Defines a **BGP session** to a router.

```yaml
apiVersion: metallb.io/v1beta2
kind: BGPPeer
metadata:
  name: sample
  namespace: metallb-system
spec:
  myASN: 64500          # this cluster's ASN
  peerASN: 64501        # the router's ASN
  peerAddress: 10.0.0.1 # the router's IP
```

---

## BGPAdvertisement

Announces a pool over **BGP** (optionally to specific peers).

```yaml
apiVersion: metallb.io/v1beta1
kind: BGPAdvertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
    - first-pool
```

---

## How they reference each other

```
IPAddressPool  (the IPs)
     ▲
     │ referenced by
L2Advertisement  /  BGPAdvertisement   (what to announce, and how)
                          │
                          │ (BGP) targets
                       BGPPeer   (the session to the router)
```

- **L2** needs: IPAddressPool + L2Advertisement.
- **BGP** needs: IPAddressPool + BGPAdvertisement + BGPPeer.

---

## Note — no pool selector = all pools

If an advertisement (L2 or BGP) sets **no** `ipAddressPools` selector, it is interpreted as applying to **all** available pools. Be explicit when you want to scope an advertisement to specific pools.