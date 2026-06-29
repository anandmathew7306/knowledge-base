# Day 2 — Localnet Mapping + NAD (annotated)

Putting a workload directly on a physical VLAN (localnet) takes **two pieces**:

1. **Bridge-mapping (NMState)** — tells OVN that a named localnet exists and which OVS bridge it maps to.
2. **NAD (NetworkAttachmentDefinition)** — the network workloads actually attach to, referencing that localnet.

---

## 1. Localnet bridge-mapping (NNCP)

```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: localnet-vlan100
spec:
  desiredState:
    ovn:
      bridge-mappings:
        - bridge: br-vlans          # which OVS bridge carries this localnet
          localnet: vlan100         # the localnet NAME (referenced by the NAD)
          state: present
status:
  conditions:
    - type: Available
      status: 'True'
      reason: SuccessfullyConfigured
```

The default cluster network maps to `br-ex` the same way:

```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: localnet-br-ex
spec:
  desiredState:
    ovn:
      bridge-mappings:
        - bridge: br-ex             # the external bridge
          localnet: localnet-br-ex  # localnet name for the default external network
          state: present
status:
  conditions:
    - type: Available
      status: 'True'
      reason: SuccessfullyConfigured
```

A bridge-mapping just declares "**this localnet name → this bridge**." It creates no IPs; it's the wiring OVN needs before a NAD can use the localnet.

---

## 2. Localnet NAD

```yaml
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: localnet-vlan100
  namespace: my-namespace          # NADs are namespaced — workloads attach within the same ns
spec:
  config: |-
    {
      "cniVersion": "0.3.1",
      "name": "localnet-vlan100",        # must match the localnet/bridge-mapping name
      "type": "ovn-k8s-cni-overlay",     # OVN-Kubernetes CNI
      "topology": "localnet",            # localnet = attach directly to the physical VLAN
      "netAttachDefName": "my-namespace/localnet-vlan100"   # namespace/name of this NAD
    }
```

A workload (pod or VM) then references this NAD by name in an annotation, and Multus attaches it to that localnet network — placing it directly on the VLAN.

---

## How the pieces connect

```
bridge-mapping (NMState)     localnet "vlan100" → bridge br-vlans
        │  (same name)
        ▼
NAD  topology: localnet, name: vlan100   (in a namespace)
        │  (referenced by annotation)
        ▼
pod / VM   attaches → lands directly on the physical VLAN
```

The **name** is the link: the bridge-mapping defines the localnet, the NAD references that localnet, and the workload references the NAD.

---

## Other NAD types

Not every NAD is localnet. The same NAD mechanism can use other CNI plugins — for example an **ipvlan** NAD attaches a workload to a VLAN via the ipvlan plugin instead of OVN:

```json
{ "cniVersion": "0.3.1", "type": "ipvlan", "master": "br-vlans.100", "mode": "l2",
  "routes": [ { "dst": "10.0.20.0/26" } ] }
```

The `type` field selects the plugin; `ovn-k8s-cni-overlay` (localnet) is the common one here, but ipvlan and others are possible for specific needs.