# Day 1 — SiteConfig (annotated)

SiteConfig is the **Day-1 provisioning** resource used by ZTP (Zero Touch Provisioning) via RHACM. It defines the cluster identity, networking, and the bare-metal nodes to install. One SiteConfig describes one cluster.

---

## Annotated SiteConfig

```yaml
apiVersion: ran.openshift.io/v1
kind: SiteConfig
metadata:
  name: "cluster-01"                      # cluster name (also used as namespace)
  namespace: cluster-01
spec:
  baseDomain: "example.com"               # DNS base domain for the cluster
  pullSecretRef:
    name: "pullsecret-cluster-01"         # secret holding registry pull credentials
  clusterImageSetNameRef: "img4.16.15-x86-64-appsub"  # OCP version/image set to install
  sshPublicKey: "ssh-ed25519 AAAA...REDACTED"          # SSH key injected into nodes

  clusters:
  - clusterName: "cluster-01"
    extraManifestPath: extra-manifests    # folder of extra Day-1 manifests
    extraManifests:
      filter:
        inclusionDefault: exclude         # exclude all, then include only the listed ones
        include:
          - timezone-master.yaml
          - timezone-worker.yaml
          - cis-remediation-master-machineconfig.yaml   # CIS hardening
          - cis-remediation-worker-machineconfig.yaml
          - ssh-config-master.yaml
          - ssh-config-worker.yaml
          - cli-banner-machineconfig-master.yaml
          - cli-banner-machineconfig-worker.yaml

    networkType: "OVNKubernetes"          # CNI plugin

    clusterLabels:                        # labels used by ACM/GitOps to target policies
      sites: "cluster-01"
      ztp.app.cloud/base: false
      ztp.app.cloud/group: "group-a"
      ztp.app.cloud/overlay: "cluster-01"
      ztp.app.cloud/environment: "npe"    # environment (e.g. non-prod)
      location: "site-a"
      cluster.open-cluster-management.io/clusterset: "group-a"

    clusterNetwork:                        # pod network
      - cidr: 10.0.0.0/14
        hostPrefix: 23                     # /23 slice per node

    apiVIP: 192.0.2.5                      # HA virtual IP for the API server
    ingressVIP: 192.0.2.6                  # HA virtual IP for ingress

    serviceNetwork:
      - 192.168.0.0/16                     # ClusterIP service network

    additionalNTPSources:                  # time sync sources
      - ntp1.example.com
      - ntp2.example.com

    # Optional overrides applied to generated CRs for this cluster:
    crTemplates:
      KlusterletAddonConfig: "KlusterletAddonConfigOverride.yaml"
      BareMetalHost: "baremetalhost-override.yml"        # remove if cluster must be rebuilt
      ClusterDeployment: "clusterdeployment-override.yml" # remove if cluster must be rebuilt

    proxy:                                 # cluster-wide proxy (if egress is via a proxy)
      httpProxy: http://USER:PASS@proxy.example.com:8080
      httpsProxy: http://USER:PASS@proxy.example.com:8080
      noProxy: .example.com,192.0.2.0/24,localhost,127.0.0.1   # bypass proxy for these

    nodes:
      - hostName: control1.cluster-01.example.com
        role: "master"
        bmcAddress: "redfish-virtualmedia://192.0.2.73/redfish/v1/Systems/1"  # out-of-band mgmt (Redfish)
        bmcCredentialsName:
          name: "bmc-secret-control1"      # secret with BMC login
        bootMACAddress: 00:11:22:33:44:01  # MAC the node PXE/virtual-media boots from
        bootMode: "UEFI"
        rootDeviceHints:
          deviceName: "/dev/nvme0n1"        # which disk to install onto

        nodeNetwork:                        # NMState — Day-1 host networking for this node
          config:
            dns-resolver:
              config:
                server:
                - 10.0.0.10                 # DNS servers
                - 10.0.0.11
            interfaces:
            - name: bond0                   # the bonded interface
              ipv4:
                enabled: false              # no IP on the bond itself
              ipv6:
                enabled: false
              link-aggregation:
                mode: 802.3ad               # LACP
                options:
                  lacp_rate: fast
                  miimon: "100"             # link poll interval (ms)
                port:
                - "ens3f0np0"               # bond member NICs
                - "ens2f0np0"
              mtu: 8700                      # jumbo frames
              state: up
              type: bond

            - name: bond.934                # machine-network VLAN subinterface on bond0
              ipv4:
                address:
                - ip: 10.0.1.7
                  prefix-length: 24
                dhcp: false
                enabled: true
              ipv6:
                address:
                - ip: "2001:db8:1::7"
                  prefix-length: 64
                dhcp: false
                enabled: true
              state: up
              type: vlan
              vlan:
                base-iface: bond0
                id: 934                      # VLAN ID

            routes:
              config:
              - destination: 0.0.0.0/0       # default route (IPv4)
                next-hop-address: 10.0.1.1    # machine-network gateway
                next-hop-interface: bond.934
              - destination: ::/0            # default route (IPv6)
                next-hop-address: "2001:db8:1::1"
                next-hop-interface: bond.934

          interfaces:                        # map of expected NIC names → MACs
          - macAddress: 00:11:22:33:44:0a
            name: ens3f0np0
          - macAddress: 00:11:22:33:44:01
            name: ens2f0np0
```

---

## What this configures (Day 1)

| Area | Field(s) | Result |
|------|----------|--------|
| Cluster identity | name, baseDomain, clusterImageSetNameRef | which cluster, what version |
| CNI | networkType | OVN-Kubernetes |
| Pod / service nets | clusterNetwork, serviceNetwork | overlay IP ranges |
| HA VIPs | apiVIP, ingressVIP | API + ingress virtual IPs |
| Bond | link-aggregation | bonds the two NICs (LACP) |
| Machine network | bond.934 (vlan) | node IP + default gateway |
| Provisioning | bmcAddress, bootMACAddress, rootDeviceHints | how each node is installed |
| Proxy / DNS / NTP | proxy, dns-resolver, additionalNTPSources | infra services |

A SiteConfig lists every node to be provisioned; each node entry repeats the same `nodeNetwork` structure with its own host details.