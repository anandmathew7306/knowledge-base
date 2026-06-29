# CNI (Container Network Interface)

## The problem

Kubernetes itself **does not implement pod networking**. It doesn't create the pod's interface, assign its IP, or set up routes. Instead of building this in, Kubernetes **delegates** it to a plugin.

---

## What CNI is

**CNI (Container Network Interface)** is a **standard / specification** — a defined way for Kubernetes (or any container runtime) to ask a plugin to set up a pod's networking. It's an **interface**, not a specific product.

---

## Why a standard

A standard keeps Kubernetes **pluggable**. Any networking solution that implements the CNI spec can be plugged in, and Kubernetes stays **networking-agnostic** — it doesn't care *how* the networking is done, only that the plugin follows the interface.

---

## What a CNI plugin does

When a pod starts (or stops), the runtime calls the CNI plugin to:

- Create the pod's network interface (the veth into its namespace)
- Assign an **IP** from the cluster network
- Set up **routes** so the pod can communicate
- Tear it all down when the pod is destroyed

---

## The flow

```
pod scheduled
   → runtime calls the CNI plugin
   → plugin wires up namespace + IP + routes
   → pod has working networking
```

---

## Examples

Common CNI plugins (names for awareness):

- Calico
- Flannel
- Cilium
- **OVN-Kubernetes** ← used on OpenShift / our cluster

Different CNIs make different design choices — overlay vs plain routing, how they enforce network policy, and so on — but they all speak the **same CNI interface** to Kubernetes. The specifics of OVN-Kubernetes are covered in the OpenShift notes.