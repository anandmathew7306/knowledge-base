# Day 2 — OVS Bridge (br-vlans) NNCP (annotated)

A **Day-2** NodeNetworkConfigurationPolicy that creates the custom OVS bridge (`br-vlans`) carrying the workload VLANs. It defines two interfaces: the bridge itself, and the internal handoff port (`ovs0`) that exposes those VLANs to Linux.

---

## Annotated NNCP

```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy        # NNCP — declares desired node network state
metadata:
  name: br-vlans
spec:
  desiredState:
    interfaces:
      - name: ovs0                           # OVS internal port — Linux sees it as a NIC
        mtu: 8700                            # jumbo frames (match the bond)
        ipv4:
          enabled: false                     # no IP on ovs0 itself
        ipv6:
          enabled: false
        state: up
        type: ovs-interface                  # exposes the bridge to the Linux stack
                                             # (VLAN subinterfaces get created on top of this)

      - name: br-vlans                       # the custom OVS bridge
        mtu: 8700
        bridge:
          allow-extra-patch-ports: true      # permit OVN to add its localnet patch ports
          options:
            stp: false                       # spanning tree off (not needed here)
          port:
            - name: ovs0                      # internal handoff port (to Linux)
            - name: bond0                     # uplink port — the physical bond
              vlan:
                mode: trunk                   # trunk = carry multiple tagged VLANs
                trunk-tags:                   # the VLAN IDs allowed on this trunk
                  - id: 654
                  - id: 664
                  - id: 1165
                  - id: 1491
        state: up
        type: ovs-bridge
status:
  conditions:
    - type: Available
      status: 'True'
      reason: SuccessfullyConfigured          # applied successfully on the node
```

---

## What this configures

| Element | Field | Result |
|---------|-------|--------|
| OVS bridge | `type: ovs-bridge` | creates `br-vlans` |
| Uplink | port `bond0` + `mode: trunk` | bond carries the listed VLANs into the bridge |
| Trunked VLANs | `trunk-tags` | which VLAN IDs are allowed on the uplink |
| Linux handoff | `ovs0` (`type: ovs-interface`) | exposes the bridge to Linux so VLAN subinterfaces can sit on it |
| OVN integration | `allow-extra-patch-ports: true` | lets OVN attach its localnet patch ports |
| MTU | `mtu: 8700` | jumbo frames end-to-end |

---

## How it fits

```
bond0 (trunk: VLAN 654, 664, 1165, 1491)
   │
br-vlans (OVS bridge)
   │
ovs0 (internal handoff to Linux)
   │
VLAN subinterfaces (br-vlans.654, br-vlans.664, ...) created in later NNCPs
```

`bond0` brings the tagged VLANs in; `br-vlans` switches them; `ovs0` hands them to Linux, where each VLAN gets its own subinterface (configured by separate VRF/VLAN NNCPs).