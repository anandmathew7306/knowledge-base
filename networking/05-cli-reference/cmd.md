# CLI Reference

Core commands per topic. All read-only. `oc` against the cluster; `ip`/`ovs-vsctl` on a node (via `oc debug node/<node> -- chroot /host ...`).

---

## Interfaces & bonds

```bash
ip link show                       # list all interfaces and their state
ip addr show                       # interfaces with their IP addresses
ip -d link show bond0              # bond detail (mode, lacp_rate, miimon)
cat /proc/net/bonding/bond0        # live bond status and member links
```

---

## VLAN

```bash
ip -d link show bond0.934          # VLAN subinterface detail (shows VLAN id + parent)
ip link show type vlan             # list only VLAN interfaces
```

---

## VRF & routing tables

```bash
ip vrf show                        # list VRFs and their table IDs
ip link show type vrf              # list VRF devices
ip route show table <id>           # routes in a specific table (e.g. a VRF's table)
ip route show table all            # all routes across all tables
ip rule show                       # policy rules (which table a packet uses)
```

---

## BGP

```bash
# MetalLB runs BGP via FRR inside the speaker pod:
oc -n metallb-system get pods                                  # find speaker pods
oc -n metallb-system exec <speaker-pod> -c frr -- vtysh -c "show bgp summary"   # session state
oc -n metallb-system exec <speaker-pod> -c frr -- vtysh -c "show bgp neighbors" # peer detail
```

---

## NMState / NNCP

```bash
oc get nncp                        # node network config policies (desired state)
oc get nnce                        # per-node enactment status (applied / failed)
oc get nns                         # node network state (live, per node)
oc get nns <node> -o yaml          # full live network state for one node
```

---

## OVN-Kubernetes

```bash
oc get pods -n openshift-ovn-kubernetes              # OVN-K control/data plane pods
oc get pods -n openshift-ovn-kubernetes -o wide      # which pods on which node
oc logs -n openshift-ovn-kubernetes <ovnkube-node-pod> -c ovnkube-controller   # node controller logs
```

---

## OVS bridges

```bash
ovs-vsctl show                     # all bridges, ports, and their connections
ovs-vsctl list-br                  # list bridge names (br-int, br-ex, br-vlans)
ovs-vsctl list-ports br-vlans      # ports on a specific bridge
ovs-ofctl dump-flows br-int        # OpenFlow rules on a bridge (deep inspection)
```

---

## Geneve overlay

```bash
ovs-vsctl get Open_vSwitch . external_ids    # node's OVN/encap info
ip -d link show genev_sys_6081               # the geneve tunnel device
```

---

## Localnet

```bash
oc get nncp                                  # localnet bridge-mappings are defined here
ovs-vsctl get Open_vSwitch . external_ids:ovn-bridge-mappings   # active localnet → bridge mappings
oc get net-attach-def -A                     # NADs that attach workloads to localnet
```

---

## MetalLB

```bash
oc -n metallb-system get bgppeers            # configured BGP peers
oc -n metallb-system get ipaddresspools      # IP pools
oc -n metallb-system get bgpadvertisements   # which pools advertised to which peers
oc -n metallb-system get servicebgpstatuses  # live advertisement status
oc get svc -A --field-selector spec.type=LoadBalancer   # all LoadBalancer services + their IPs
```

---

## Quick full report

```bash
python3 ocp-net-report.py          # one-command network reference for the whole cluster
```