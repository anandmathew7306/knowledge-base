# DNS

## What DNS is

DNS (Domain Name System) turns **names into IP addresses** (and back). Humans use names like `example.com`; networks route on IPs like `93.184.216.34`. DNS is the lookup system that bridges the two.

---

## Resolution flow

When a name needs to be resolved, the lookup walks a chain until it gets an answer:

```
app / browser cache      already known? return it
        ↓
OS cache                 recently resolved? return it
        ↓
/etc/hosts               local override file checked
        ↓
resolver (DNS server)    does the legwork on your behalf
        ↓
root server              "ask the .com servers"
        ↓
TLD server (.com)        "ask example.com's servers"
        ↓
authoritative server     holds the actual record → returns the IP
        ↓
answer                   returned to the app (and cached on the way back)
```

---

## Record types

| Record | Purpose |
|--------|---------|
| A | name → IPv4 address |
| AAAA | name → IPv6 address |
| CNAME | alias pointing to another name |
| PTR | reverse lookup: IP → name |
| NS | which servers are authoritative for a domain |
| MX | mail servers for a domain |