# Service Usage

How applications consume MetalLB when creating LoadBalancer services.

---

## Requesting a specific IP

Use the `metallb.io/loadBalancerIPs` annotation to ask for a particular address:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  annotations:
    metallb.io/loadBalancerIPs: 192.168.1.100   # request this exact IP
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

---

## Requesting a specific pool

Use the `metallb.io/address-pool` annotation to draw from a particular pool (MetalLB picks the next free IP in it):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  annotations:
    metallb.io/address-pool: production-public-ips   # request from this pool
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

---

## Traffic policies

A service's `externalTrafficPolicy` (`Cluster` or `Local`) controls how the receiving node delivers traffic to pods. The behaviour differs by mode — see `modes.md` for the full L2/BGP × Cluster/Local matrix.

- **Cluster** — spreads to pods across all nodes (extra hop, source IP lost).
- **Local** — delivers only to pods on the receiving node (no extra hop, source IP preserved).

---

## IPv6 and dual-stack services

MetalLB can assign IPv6 addresses, and **dual-stack** services (both IPv4 and IPv6) are supported when the pools provide both families. The service requests the appropriate family/families and MetalLB assigns from matching pools.

---

## IP address sharing

Multiple services can **share a single external IP** (for example to expose different ports on the same address), provided they're configured to allow sharing. This conserves addresses when several services can sit behind one IP.