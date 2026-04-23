# Splunk Engineering — Study Notes

> **Role:** Security Engineer (build/deploy/maintain → detection → UEBA)
> **Scope:** Enterprise on-premises distributed deployment
> **Goal:** Deploy for org + Splunk Core Admin/Architect cert + SOC detection

---

## Modules

| # | File | Topic | Status |
|---|------|--------|--------|
| 01 | [01-architecture-overview.md](01-architecture-overview.md) | Distributed Architecture — Big Picture | ✅ Done |
| 02a | [02a-universal-forwarder.md](02a-universal-forwarder.md) | Universal Forwarder (UF) | 🔜 Next |
| 02b | [02b-heavy-forwarder.md](02b-heavy-forwarder.md) | Heavy Forwarder (HF) | 🔜 |
| 02c | [02c-indexer-cluster.md](02c-indexer-cluster.md) | Indexer & Indexer Cluster | 🔜 |
| 02d | [02d-search-head-cluster.md](02d-search-head-cluster.md) | Search Head Cluster (SHC) | 🔜 |
| 02e | [02e-deployment-server.md](02e-deployment-server.md) | Deployment Server (DS) | 🔜 |
| 02f | [02f-license-manager.md](02f-license-manager.md) | License Manager & Monitoring Console | 🔜 |
| 03 | [03-data-journey.md](03-data-journey.md) | Log Journey — Endpoint to Analyst UI | 🔜 |
| 04 | [04-resource-planning.md](04-resource-planning.md) | Resource Planning & Hardware Sizing | 🔜 |
| 05 | [05-deployment-configuration.md](05-deployment-configuration.md) | Deployment, Install Order & Config | 🔜 |
| 06 | [06-cli-management.md](06-cli-management.md) | CLI Reference & Day-2 Operations | 🔜 |
| 07 | [07-security-hardening.md](07-security-hardening.md) | Security Hardening & RBAC | 🔜 |
| 08 | [08-ui-and-detection.md](08-ui-and-detection.md) | ES App, CIM, Detection & UEBA | 🔜 |

---

## Quick Reference — Ports

| Port | Protocol | Used For |
|------|----------|----------|
| `9997` | TCP | Forwarder → Indexer (data ingestion) |
| `8089` | HTTPS | Splunk management API (all components) |
| `8000` | HTTP/S | Search Head web UI |
| `8191` | TCP | KV Store (Search Head Cluster) |
| `9887` | TCP | Indexer Cluster replication |
| `8080` | TCP | Indexer Cluster replication (alternate) |
| `514` | UDP/TCP | Syslog → Heavy Forwarder |

---

## Quick Reference — Key Config Files

| File | Lives On | Controls |
|------|----------|----------|
| `inputs.conf` | Forwarder, Indexer | What data to collect / receive |
| `outputs.conf` | Forwarder | Where to send data |
| `server.conf` | All | Clustering, replication, identity |
| `props.conf` | HF, Indexer | Parsing rules, sourcetype transforms |
| `transforms.conf` | HF, Indexer | Routing, field extraction, lookup |
| `indexes.conf` | Indexer | Index definitions, storage paths, retention |
| `serverclass.conf` | Deployment Server | Which apps go to which forwarder groups |
| `authorize.conf` | Search Head | RBAC roles and capabilities |

---

## Config File Hierarchy (precedence — bottom overrides top)

```
$SPLUNK_HOME/etc/system/default/     ← Splunk's shipped defaults (never edit)
$SPLUNK_HOME/etc/system/local/       ← Your system-wide overrides
$SPLUNK_HOME/etc/apps/<app>/default/ ← App-bundled defaults
$SPLUNK_HOME/etc/apps/<app>/local/   ← App local overrides (highest app priority)
$SPLUNK_HOME/etc/users/<user>/       ← Per-user knowledge objects
```
