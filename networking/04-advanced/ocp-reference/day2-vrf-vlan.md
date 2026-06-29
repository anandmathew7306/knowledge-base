# Day 2 — VRF + VLAN NNCP (annotated)

A **Day-2** NodeNetworkConfigurationPolicy that adds one workload VLAN and places it in its own **VRF** for routing isolation. This pattern repeats per VLAN/VRF — each gets its own NNCP with its own VLAN ID, subnet, gateway, and route-table ID.

---

## Annotated NNCP

```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: vrf-vlan-100-control1            # one policy per node, per VLAN/VRF
spec:
  desiredState:
    interfaces:
      - name: br-vlans.100               # VLAN subinterface on the OVS handoff port
        ipv4:
          address:
            - ip: 10.0.2.7               # this node's IP on this VLAN
              prefix-length: 25
          enabled: true
        ipv6:
          address:
            - ip: "2001:db8:2::7"
              prefix-length: 64
          enabled: true
        state: up
        type: vlan
        vlan:
          base-iface: ovs0               # sits on ovs0 (the br-vlans handoff)
          id: 100                        # VLAN ID

      - name: vrf100                     # the VRF that isolates this VLAN's routing
        state: up
        type: vrf
        vrf:
          port:
            - br-vlans.100               # the subinterface placed inside this VRF
          route-table-id: 100            # dedicated routing table for this VRF

    route-rules:                          # policy rules: send these dests to the MAIN table (254)
      config:                             # i.e. cluster-internal traffic bypasses the VRF
        - ip-to: 192.168.0.0/16           # service network
          priority: 998
          route-table: 254
        - ip-to: 10.0.0.0/14              # pod network
          priority: 998
          route-table: 254
        - ip-to: 169.254.169.0/29         # OVN metadata/link-local
          priority: 998
          route-table: 254
        - ip-to: "2001:db8:a00::/112"     # IPv6 cluster ranges → main table
          priority: 998
          route-table: 254
        - ip-to: "2001:db8:100::/56"
          priority: 998
          route-table: 254
        - ip-to: fe80::/64                # IPv6 link-local
          priority: 998
          route-table: 254

    routes:                               # the VRF's own default routes (in table 100)
      config:
        - destination: 0.0.0.0/0
          metric: 150
          next-hop-address: 10.0.2.1      # this VLAN's gateway
          next-hop-interface: br-vlans.100
          table-id: 100                   # installed in the VRF's table, not main
        - destination: ::/0
          metric: 150
          next-hop-address: "2001:db8:2::1"
          next-hop-interface: br-vlans.100
          table-id: 100

  maxUnavailable: 1                       # roll out to one node at a time
  nodeSelector:
    kubernetes.io/hostname: control1.cluster-01.example.com   # targets one node
status:
  conditions:
    - type: Available
      status: 'True'
      reason: SuccessfullyConfigured
```

---

## What this configures

| Element | Field | Result |
|---------|-------|--------|
| VLAN subinterface | `type: vlan`, `base-iface: ovs0`, `id` | the VLAN interface on the OVS handoff |
| Node IP | `ipv4/ipv6 address` | this node's address on the VLAN |
| VRF | `type: vrf`, `route-table-id` | isolated routing table for the VLAN |
| VRF membership | `vrf.port` | binds the subinterface into the VRF |
| Default route | `routes` with `table-id` | the VRF's own gateway (in its own table) |
| Cluster-traffic exception | `route-rules` (priority 998 → table 254) | pod/service/internal traffic uses the main table, bypassing the VRF |
| Rollout | `maxUnavailable`, `nodeSelector` | one node at a time, targeted by hostname |

---

## Key idea — route-rules vs routes

- **`routes`** (table-id 100) — the VRF's own routes: anything in this VRF defaults out the VLAN gateway.
- **`route-rules`** (→ table 254) — exceptions: cluster-internal destinations (pods, services, metadata) are forced back to the **main** table so the VRF doesn't break in-cluster traffic.

Together: VLAN traffic stays isolated in the VRF, but the pod/service networks still work normally.

---

## Repeats per VRF/VLAN

Each workload VLAN has its own NNCP of this exact shape — only these change:

| Field | Varies per VRF |
|-------|----------------|
| policy `name` | vrf-vlan-`<id>`-`<node>` |
| `br-vlans.<id>` / `vlan.id` | the VLAN ID |
| node IP + gateway | that VLAN's subnet |
| `route-table-id` / `table-id` | the VRF's table number |

(One NNCP per node per VLAN; the `route-rules` block is identical across them.)