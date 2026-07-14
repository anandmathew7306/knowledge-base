# SIEM & Splunk basics

## What is a SIEM

SIEM (Security Information and Event Management) is a category of software — not a protocol. Splunk is one such product. A SIEM:

- **Ingests** logs/events from many sources (servers, firewalls, clusters, apps)
- **Correlates** them to detect suspicious patterns across systems
- **Alerts** a security team, and retains data for compliance/forensics

Source systems don't run the SIEM — they just send logs to it.

## Syslog

Syslog is the protocol most commonly used to deliver logs to a SIEM. It's old (RFC 5424 is the modern spec), lightweight, and supported by virtually every SIEM product — which is why it's the default choice regardless of which SIEM is on the receiving end.

- Fire-and-forward: sends to one destination, no built-in replay or multi-consumer support (unlike Kafka)
- Can run over UDP, TCP, or TLS-wrapped TCP



## Why audit logs specifically go to a SIEM

Audit logs record every request made to an API server — who did what, on what resource, allowed or denied. This is the security/compliance-relevant log type, which is why it's the one typically forwarded to a SIEM (as opposed to application or infrastructure logs, which are more about operational debugging).

## Why not just have the SIEM consume from Kafka instead

Technically possible — Kafka is a real streaming platform, and a SIEM could run a Kafka consumer. In practice, direct syslog delivery to a SIEM is still common because it:

- Uses SIEM's native, low-effort ingestion path (most SIEMs support syslog out of the box; Kafka ingestion usually needs a custom connector)
- Decouples the security team from depending on another team's Kafka infrastructure being available/healthy
- Keeps the audit log delivery path independent of a Kafka outage
- Can align with compliance requirements that specify direct delivery to a SIEM

