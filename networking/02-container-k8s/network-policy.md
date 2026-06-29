# Network Policy

## The default — everything is allowed

By default in Kubernetes, **all pods can talk to all other pods** — there's no restriction. For many clusters that's too open: you don't want, say, a frontend pod reaching a database it has no business touching.

---

## What a NetworkPolicy is

A **NetworkPolicy** is a Kubernetes resource that **allows or denies pod traffic** based on rules. It lets you control which pods can communicate, turning the open-by-default model into a controlled one.

---

## How rules are expressed

Rules select traffic using:

- **Pod labels** — e.g. "pods with `app: db`"
- **Namespaces** — e.g. "only pods in the `backend` namespace"
- **Ports** — e.g. "only on port 5432"

A policy can control:

- **Ingress** — what is allowed **to** the selected pods
- **Egress** — what is allowed **from** the selected pods

---

## Important behaviour — deny by default once selected

As soon as a NetworkPolicy selects a pod, that pod switches to **deny by default** for the direction(s) the policy covers — only what the policy explicitly allows gets through.

- A pod with **no** policy → open (default).
- A pod **selected** by a policy → everything is denied except the allowed rules.

---

## Example (in words)

> "Pods labelled `app: db` may receive traffic **only** from pods labelled `app: backend`, **only** on port 5432."

Everything else to those db pods is blocked.

---

## How it's enforced

NetworkPolicy is just a set of rules — a **CNI plugin** enforces it. Not all CNIs support NetworkPolicy; those that do translate the rules into their own mechanism.

> On OpenShift, **OVN-Kubernetes** enforces NetworkPolicy as **OVN ACLs** in its flow rules. (OCP specifics covered in the OpenShift notes.)