# iptables

## What iptables is

**iptables** is the Linux kernel's built-in **packet filter** — it decides what happens to network packets as they pass through the host. It does two main jobs:

- **Firewall** — allow or block traffic
- **NAT** — rewrite source/destination addresses

It works under the hood. You rarely touch it directly — tools like `firewalld` on Fedora are friendly front-ends that write iptables rules for you. But it's always there, enforcing the rules.

---

## How it's organised — tables & chains

**Tables** group rules by the kind of job:

| Table | Job |
|-------|-----|
| filter | Allow or block traffic (firewall) |
| nat | Rewrite addresses |

**Chains** are the checkpoints a packet passes through:

| Chain | When it's checked |
|-------|-------------------|
| INPUT | Traffic coming *to* this host |
| OUTPUT | Traffic going *from* this host |
| FORWARD | Traffic passing *through* this host |

At each checkpoint, rules are checked top to bottom. Each rule has a **match** (which packets) and an **action** (`ACCEPT`, `DROP`, etc.). The first matching rule wins.

---

## NAT in one line

NAT rewrites addresses — for example, changing a private source IP to the host's IP so internal machines can reach the internet behind one address. (This is the mechanism behind internet sharing and container networking.)

---

## Good to know

- **nftables** is the modern replacement for iptables — same job, newer framework. Most systems are moving to it.