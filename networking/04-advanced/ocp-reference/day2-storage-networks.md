# Day 2 — Storage Networks NNCP (annotated)

**Day-2** NodeNetworkConfigurationPolicies for the storage networks. There are two patterns:

- **iSCSI (block storage)** — on **dedicated NICs** (not bonded), one per path (A and B) for multipath redundancy.
- **Backup / NAS (Cohesity, NetApp)** — VLAN subinterfaces on **bond0**.

All storage networks here are **L2 only** — they have an IP on a directly-connected subnet but **no gateway** (server and storage array sit on the same subnet, no routing needed). No VRF.

---

## Pattern 1 — iSCSI on a dedicated NIC

```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: control1-storage-iscsi-a-vlan1244
spec:
  nodeSelector:
    kubernetes.io/hostname: control1.cluster-01.example.com   # one node
  desiredState:
    interfaces:
    - name: nic2                       # dedicated storage NIC (not in any bond)
      type: ethernet
      state: up
      ipv4:
        enabled: false                 # no IP on the raw NIC
      ipv6:
        enabled: false
      mtu: 8700                        # jumbo frames (important for storage)

    - name: nic2.1244                  # VLAN subinterface for iSCSI path A
      description: VLAN 1244 subint on nic2
      type: vlan
      state: up
      vlan:
        base-iface: nic2               # parent = the dedicated NIC
        id: 1244
      ipv4:
        address:
        - ip: 10.0.10.133              # node's iSCSI-A IP
          prefix-length: 27
        enabled: true                  # no gateway/route → L2 only
status:
  conditions:
    - type: Available
      status: 'True'
      reason: SuccessfullyConfigured
```

> **iSCSI-B** is identical but on the **other** dedicated NIC (`nic3`) with a different VLAN (e.g. 1245) and its own subnet — a separate path to the same storage for multipath redundancy.

---

## Pattern 2 — Backup / NAS VLAN on bond0

```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: control1-storage-bond0-vlan1212
spec:
  nodeSelector:
    kubernetes.io/hostname: control1.cluster-01.example.com
  desiredState:
    interfaces:
    - name: bond0.1212                 # VLAN subinterface directly on bond0
      description: VLAN 1212 subint on bond0
      type: vlan
      state: up
      vlan:
        base-iface: bond0              # parent = the bond (not a dedicated NIC, not OVS)
        id: 1212
      ipv4:
        address:
        - ip: 10.0.20.16               # node's backup-network IP
          prefix-length: 23
        enabled: true                  # no gateway → L2 only
      ipv6:
        enabled: false
status:
  conditions:
    - type: Available
      status: 'True'
      reason: SuccessfullyConfigured
```

> **NetApp** uses the same shape — another VLAN on `bond0` (e.g. 1490) with its own subnet. Cohesity and NetApp differ only by VLAN ID and subnet.

---

## What this configures

| Network | Parent | Type | Gateway |
|---------|--------|------|---------|
| iSCSI-A | dedicated NIC (nic2) | block storage, multipath A | none (L2) |
| iSCSI-B | dedicated NIC (nic3) | block storage, multipath B | none (L2) |
| Backup (Cohesity) | bond0 | NAS/backup | none (L2) |
| NetApp | bond0 | NAS | none (L2) |

---

## Key points

- **iSCSI uses dedicated NICs**, not the bond — separate physical paths (A and B) give multipath redundancy to the storage array.
- **Backup/NAS ride bond0** as ordinary VLAN subinterfaces — they don't need dedicated NICs.
- **All are L2 only** — an IP on a directly-connected subnet, no gateway, no VRF. Storage traffic never routes; the array is on the same subnet.
- **Jumbo frames (`mtu: 8700`)** matter for storage throughput.

---

## Note on NIC naming

The dedicated storage NIC may enumerate **with or without an `npX` suffix** depending on the host (e.g. `nic2np1` vs `nic2`). The policy must use whatever name the OS actually presents on that node — so per-node storage NNCPs can legitimately differ in the interface name even though everything else matches.