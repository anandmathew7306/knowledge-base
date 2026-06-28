# NetworkManager

## The problem

Commands like `ip addr` and `ip route` change networking **live** — but the changes are lost on reboot. Something needs to **apply and persist** network configuration so it survives restarts. That's what NetworkManager does.

---

## NetworkManager

**NetworkManager** is the standard Linux service that manages network interfaces and makes their configuration **persistent** across reboots. It stores each interface's settings as a **connection profile** and re-applies them on boot. Its command-line tool is **`nmcli`**.

---

## Connection profiles

A **connection profile** is a saved set of settings for an interface — its IP, VLAN tag, bond membership, routes, DNS, and so on. NetworkManager applies the matching profile when an interface comes up, so the config is consistent every boot rather than something you re-enter each time.

---

## Declarative vs imperative

| Approach | How it works | Examples |
|----------|-------------|----------|
| **Imperative** | You run commands step by step to reach a state | `ip`, `nmcli` |
| **Declarative** | You describe the **desired end state**; the tool works out the steps | NMState |

Imperative = "do this, then this, then this." Declarative = "here's what it should look like — make it so."

---

## Declarative config with NMState

**NMState** is a tool that takes a **declarative** description of the desired network state (written in YAML) and makes the system match it. Instead of issuing individual commands, you declare the end result — interfaces, bonds, VLANs, routes, VRFs — and NMState drives **NetworkManager** underneath to apply it.

As a standalone Linux tool its CLI is `nmstatectl`. (In OpenShift the same engine runs as an operator applied cluster-wide — covered separately in the OpenShift notes.)

---

## How they stack

```
NMState  (declares desired state, YAML)
   ↓
NetworkManager  (applies and persists config)
   ↓
Linux kernel  (live interfaces, routes, VRFs)
```

Each layer hands down to the next: you declare intent, NetworkManager makes it persistent, the kernel runs it.