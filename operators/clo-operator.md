# Cluster Logging Operator (CLO)

CLO is the operator that manages OpenShift's logging stack: deploying the log collector, and forwarding logs based on a `ClusterLogForwarder` resource.

## Operator vs CRD

- **CRD (CustomResourceDefinition)** — a schema registration. Tells the API "this kind of object exists." Doesn't run anything by itself.
- **Operator** — the running controller that watches for objects of that CRD and acts on them (e.g. deploys collector pods, configures forwarding).

A CRD can exist on a cluster with zero instances of it — that's normal, not a conflict. What matters is whether an actual instance (custom resource) exists and which operator is managing it.

## Two CRD API groups

CLO has evolved to support two API groups for `ClusterLogForwarder`:

- `logging.openshift.io` — legacy group, older schema
- `observability.openshift.io` — current group, being adopted going forward

One CLO installation (single operator) can support both groups depending on its version — this isn't two competing operators. Both CRD types can be registered on a cluster simultaneously; only the group under which an actual `ClusterLogForwarder` instance exists is "live."

Migrating from the old group to the new one is a manual step (create the new resource, retire the old one) — it doesn't happen automatically just because the operator version supports the new schema.

## Installation and upgrades (OLM)

CLO is installed via OLM (Operator Lifecycle Manager):

- **Subscription** — pins the operator to a **channel** (e.g. `stable-6.4`), and defines the upgrade approval mode (`Automatic` or `Manual`).
- **CSV (ClusterServiceVersion)** — records the actual installed version, and what version it upgraded from.

Check both to understand current state:

```
oc get sub -n <namespace>          # channel, approval mode
oc get csv -A | grep cluster-logging   # installed version, upgrade history
```

Note: some CLI shortnames can resolve ambiguously when multiple CRDs share a kind across API groups (e.g. `subscription` may resolve differently than the `sub` shortname, depending on what else is registered on the cluster). Always verify with an explicit/short form rather than assuming a "no resources found" result is conclusive.