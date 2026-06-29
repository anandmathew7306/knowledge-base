# Components

MetalLB runs as two main components, with FRR-K8s backing the BGP side.

---

## Controller

The **controller** is responsible for **assigning IPs** to services. It watches for LoadBalancer services, picks an available address from an eligible pool, and assigns it. There is one controller for the cluster (the allocation brain).

---

## Speaker

The **speaker** is responsible for **announcing** services — via L2 or BGP. It runs on each node (a DaemonSet), so any node can attract and serve traffic for a service IP. The speaker is what actually makes an assigned IP reachable on the network.

---

## FRR-K8s

For BGP, MetalLB uses **FRR-K8s** as the default backend:

- **FRR (Free Range Routing)** is a free, open-source routing software suite — it speaks BGP (and other routing protocols).
- **FRR-K8s** is a Kubernetes wrapper around FRR.
- In FRR-K8s mode, an **FRR-K8s instance is deployed on the same nodes as the speaker**. The speaker decides what to advertise; FRR-K8s handles the actual BGP sessions and route exchange with the routers.

---

## How they fit together

```
Controller   assigns an IP to the service
     │
Speaker      announces that IP (per node)
     │
FRR-K8s      (BGP mode) runs the BGP sessions and advertises the route
     │
Router       learns the route and forwards traffic to the node(s)
```

The controller allocates, the speaker announces, and in BGP mode FRR-K8s does the routing-protocol heavy lifting.