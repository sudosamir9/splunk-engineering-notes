# Module 1 — Distributed Splunk Architecture Overview

> **The core question this module answers:**
> Why does distributed Splunk exist, what are its components, and how do they relate to each other?

---

## Architecture Diagram (SVG)

> Open in browser or VS Code for full quality:
> `splunk/assets/01-distributed-architecture.svg`

---

## Official Sources

All content in this module is sourced from official Splunk documentation:

| Document | Covers | Link |
|----------|--------|------|
| Types of Distributed Deployments | Deployment sizes, which components each requires | [Deploymentcharacteristics](https://docs.splunk.com/Documentation/Splunk/9.3.0/Deploy/Deploymentcharacteristics) |
| Start Implementing Distributed Deployment | Installation order, component roles | [Implementationoverview](https://docs.splunk.com/Documentation/Splunk/9.4.2/Deploy/Implementationoverview) |
| SVA C1/C11 — Single-Site Clustered Deployment | The validated architecture this module's topology is based on | [SVA C1C11](https://docs.splunk.com/Documentation/SVA/current/Architectures/C1C11) |
| About Splunk Validated Architectures | What SVAs are, why to use them | [SVA Introduction](https://help.splunk.com/en/splunk-cloud-platform/splunk-validated-architectures/introduction-to-splunk-validated-architectures/about-splunk-validated-architectures) |
| High Availability: Indexer Cluster | Clustering architecture, replication factor, search factor | [Indexercluster](https://docs.splunk.com/Documentation/Splunk/9.4.2/Deploy/Indexercluster) |
| Basics of Indexer Cluster Architecture | Manager node, peer nodes, bucket replication | [Basicclusterarchitecture](https://docs.splunk.com/Documentation/Splunk/9.4.2/Indexer/Basicclusterarchitecture) |
| Medium-Large Enterprise: SHC with Indexers | Integrating SHC with Indexer Cluster | [SHCwithindexers](https://docs.splunk.com/Documentation/Splunk/9.4.2/Deploy/SHCwithindexers) |
| Deploy a Search Head Cluster | SHC deployment, Captain election, deployer role | [SHCdeploymentoverview](https://docs.splunk.com/Documentation/Splunk/9.4.0/DistSearch/SHCdeploymentoverview) |
| Deployment Server Architecture | Phone-home model, server classes | [Deploymentserverarchitecture](https://docs.splunk.com/Documentation/Splunk/9.4.2/Updating/Deploymentserverarchitecture) |

---

## Why Distributed? The Single-Instance Problem

A single-instance Splunk deployment puts everything on one machine:
receive → parse → index → store → search → serve UI.

This breaks in enterprise for three reasons:

| Problem | What Happens |
|---------|--------------|
| **Resource contention** | A 30-day threat hunt at 9am competes with live ingestion for CPU/RAM. Logs drop. You create SOC blind spots *during* incidents. |
| **Single point of failure** | Box goes down = zero visibility. In a SOC, that's a P1 incident on its own. |
| **No horizontal scale** | Can't add a second identical server. Storage, ingest rate, and search performance all hit a wall together. |

**Distributed Splunk solves this by separating concerns.** Each function runs on dedicated hardware, scaled independently, with redundancy at each tier.

---

## The Full Architecture Diagram

```
╔══════════════════════════════════════════════════════════════════════════════════════╗
║                      DISTRIBUTED SPLUNK — ENTERPRISE ARCHITECTURE                   ║
╚══════════════════════════════════════════════════════════════════════════════════════╝

   ENDPOINTS              COLLECTION TIER         INDEXING TIER          SEARCH TIER
   ─────────              ───────────────         ─────────────          ───────────

 ┌──────────────┐
 │ Windows DC   │──[UF]──┐
 │ Windows Srv  │──[UF]──┤                        ┌─────────────┐
 │ Linux Host   │──[UF]──┤   ┌────────────────┐   │  Indexer 1  │
 │ SQL Server   │──[UF]──┼──▶│     Heavy      │──▶│  Indexer 2  │◀──┐   ┌────────────────┐
 │ App Server   │──[UF]──┘   │   Forwarder    │   │  Indexer 3  │   ├──▶│  Search Head 1 │──▶ Analyst
 └──────────────┘            │  (optional)    │   │  Indexer N  │   │   │  Search Head 2 │──▶ Analyst
                             └────────────────┘   └─────────────┘   │   │  Search Head 3 │──▶ Analyst
 ┌──────────────┐                   ▲                    ▲           │   └────────────────┘
 │  Palo Alto   │──syslog──┐        │                    │           │          ▲
 │  Cisco ASA   │──syslog──┼────────┘             Cluster Manager   │          │
 │  Cisco SW    │──syslog──┘                      (coordinates)     │    [port 8000]
 │  WinDNS      │──[UF]───────────────────────────────────────────┘  Browser/SIEM
 └──────────────┘
                 ╔══════════════════════════════════════════╗
                 ║            MANAGEMENT PLANE              ║
                 ║  ┌─────────────────────────────────┐    ║
                 ║  │  Cluster Manager                │    ║
                 ║  │  (controls indexer cluster)     │    ║
                 ║  ├─────────────────────────────────┤    ║
                 ║  │  License Manager                │    ║
                 ║  │  (tracks daily GB ingested)     │    ║
                 ║  ├─────────────────────────────────┤    ║
                 ║  │  Deployment Server              │    ║
                 ║  │  (pushes configs to UFs)        │    ║
                 ║  └─────────────────────────────────┘    ║
                 ╚══════════════════════════════════════════╝
```

---

## The 7 Roles — One-Line Summary

| Role | One Job |
|------|---------|
| **Universal Forwarder (UF)** | Collect logs on the endpoint, ship them out. Nothing else. |
| **Heavy Forwarder (HF)** | Parse, filter, and route logs before they hit the indexer. |
| **Indexer** | Receive events, parse, write to disk buckets, serve search results. |
| **Cluster Manager** | Control the indexer cluster — replication, peer state, rolling restarts. |
| **Search Head** | Accept analyst queries, fan them out to all indexers, merge results. |
| **Deployment Server** | Push config updates to every forwarder in the fleet. |
| **License Manager** | Track how many GB/day are indexed against the license. |

---

## Data Plane vs. Management Plane

This distinction matters for HA planning, firewall rules, and cert exams.

```
┌─────────────────────────────────────────────────────────────────────┐
│  DATA PLANE — where logs actually flow                              │
│                                                                     │
│  UF ──[TCP 9997]──▶ HF ──[TCP 9997]──▶ Indexers ◀──▶ Search Heads │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  MANAGEMENT PLANE — where config and control signals flow           │
│                                                                     │
│  Deployment Server ──[8089]──▶ UFs (push apps)                     │
│  Cluster Manager   ──[8089]──▶ Indexer peers (replication control) │
│  License Manager   ◀──[8089]── Indexers (daily GB reporting)       │
└─────────────────────────────────────────────────────────────────────┘
```

> **SOC relevance:** When you're troubleshooting missing data during an incident, you need to know whether the problem is in the data plane (log not being collected/forwarded/indexed) or the management plane (config not pushed, cluster degraded). These are completely separate failure domains.

---

## Component Deep Dive — High Level

### 1. Universal Forwarder (UF)

```
┌──────────────────────────────────────────────┐
│             Universal Forwarder              │
│  ┌─────────────┐      ┌────────────────────┐ │
│  │ inputs.conf │      │   outputs.conf     │ │
│  │ - files     │─────▶│ - indexer IPs      │ │
│  │ - WinEvtLog │      │ - load balance     │ │
│  │ - syslog rx │      │ - TLS settings     │ │
│  └─────────────┘      └────────────────────┘ │
│                                              │
│  RAM: ~50-100 MB    CPU: negligible          │
│  NO index  NO search  NO web UI              │
└──────────────────────────────────────────────┘
```

- Stripped-down binary — safe to run on DCs, production SQL servers, POS terminals
- Reads files, Windows Event Log, registry, executes scripts
- Compresses + TLS encrypts before sending
- **Cannot parse syslog.** Cannot route. Cannot transform.

---

### 2. Heavy Forwarder (HF)

```
┌──────────────────────────────────────────────────────┐
│                  Heavy Forwarder                     │
│                                                      │
│  Receives:                  Forwards to indexers:    │
│  - syslog UDP/TCP 514  ──▶  - parsed events          │
│  - API pulls (O365, AWS)──▶  - correct sourcetype    │
│  - UF aggregation     ──▶  - correct index           │
│                                                      │
│  Uses: props.conf + transforms.conf to route         │
│  RAM: 8-16 GB    CPU: 4-8 cores                      │
└──────────────────────────────────────────────────────┘
```

- Full Splunk instance — just configured NOT to store data locally
- 1–4 per environment, positioned as aggregation/routing points
- Key use cases: syslog from network gear, CEF/LEEF parsing, API data ingestion, index routing

---

### 3. Indexer Cluster

```
┌─────────────────────────────────────────────────────────────┐
│                     Indexer Cluster                         │
│                                                             │
│  ┌───────────┐   ┌───────────┐   ┌───────────┐             │
│  │ Indexer 1 │   │ Indexer 2 │   │ Indexer 3 │             │
│  │           │   │           │   │           │             │
│  │ hot bucket│   │ hot bucket│   │ hot bucket│  ← active   │
│  │ warm      │   │ warm      │   │ warm      │             │
│  │ cold      │   │ cold      │   │ cold      │  ← slower   │
│  └───────────┘   └───────────┘   └───────────┘             │
│        ▲               ▲               ▲                    │
│        └───────────────┴───────────────┘                    │
│                        │                                    │
│              Cluster Manager                                │
│              (manages replication,                          │
│               peer health, config)                          │
│                                                             │
│  RAM: 32-64 GB    CPU: 12-16 cores    Disk: SSD for hot    │
└─────────────────────────────────────────────────────────────┘
```

- Receives events from forwarders on **TCP 9997**
- Each peer replicates buckets to other peers (replication factor = 3 typical)
- Bucket lifecycle: **hot → warm → cold → frozen → thawed**
- Cluster Manager holds control plane; does NOT index data itself

---

### 4. Search Head Cluster (SHC)

```
┌──────────────────────────────────────────────────────────────┐
│                   Search Head Cluster                        │
│                                                              │
│   ┌────────────┐   ┌────────────┐   ┌────────────┐          │
│   │  SH 1      │   │  SH 2      │   │  SH 3      │          │
│   │ (Captain)  │   │ (Member)   │   │ (Member)   │          │
│   └────────────┘   └────────────┘   └────────────┘          │
│          │                │                │                 │
│          └────────────────┴────────────────┘                 │
│                           │                                  │
│              Shared knowledge objects:                       │
│              saved searches, dashboards, lookups             │
│                                                              │
│  Browser connects to any SH on port 8000                    │
│  RAM: 16-32 GB    CPU: 8-16 cores                            │
└──────────────────────────────────────────────────────────────┘
```

- Analysts hit port **8000** on any Search Head
- SH fans out the query to all indexers simultaneously → merges results
- No event data stored on SH — only knowledge objects
- Captain elected automatically; if Captain dies, new election happens

---

### 5. Deployment Server

```
┌──────────────────────────────────────────────────────────────┐
│                    Deployment Server                         │
│                                                              │
│   serverclass.conf                                           │
│   ├── ServerClass: Windows_DCs                               │
│   │   ├── App: windows-inputs                               │
│   │   └── App: sysmon-inputs                                │
│   ├── ServerClass: Linux_Servers                             │
│   │   └── App: linux-inputs                                 │
│   └── ServerClass: Network_Devices_Syslog                    │
│       └── App: hf-routing-config                            │
│                                                              │
│   Every forwarder phones home every 60s → gets updates       │
└──────────────────────────────────────────────────────────────┘
```

- Config management for the entire forwarder fleet
- You never SSH into individual forwarders to change `inputs.conf`
- Changes propagate automatically on the next phone-home interval

---

### 6. Cluster Manager

```
  Cluster Manager
       │
       ├──▶ Tracks peer health (is Indexer 2 alive?)
       ├──▶ Manages bucket replication (ensures RF=3 is met)
       ├──▶ Coordinates rolling restarts (upgrade one peer at a time)
       └──▶ Tells Search Heads which peers to query
```

- Does NOT index or store data
- If Cluster Manager goes down: **searches and ingestion continue**
  - Peers can't receive new config changes
  - Bucket replication can't self-heal
  - But data flows — this is by design

---

### 7. License Manager

```
  All Indexers ──[daily GB report]──▶ License Manager
                                            │
                                    ┌───────▴───────┐
                                    │  License Pool  │
                                    │  100 GB/day   │
                                    └───────────────┘
                                            │
                              If exceeded >5x in 30 days:
                              Search disabled (data still indexed)
```

- Typically co-located with Cluster Manager
- Violations are warnings, not hard stops — **data is never dropped**
- Search is what gets blocked on sustained overage

---

## Hardware Placement Reference

| Role | Own Box? | Why |
|------|----------|-----|
| Universal Forwarder | No — on endpoint | Lightweight by design |
| Heavy Forwarder | Yes | Dedicated collection; handles syslog storms |
| Indexer (each peer) | Yes | Disk I/O intensive; can't share with search |
| Search Head (each) | Yes | Memory intensive for result merging |
| Cluster Manager | Shares with License Manager | Minimal CPU/RAM, just control signals |
| Deployment Server | Shares with Cluster Manager | Same reason |

---

## Self-Check Questions

Before moving to Module 2, answer these without looking up:

1. A Universal Forwarder receives syslog from a Cisco ASA. Can it parse the CEF format and route it to the `firewall` index? Why or why not?
2. Your Cluster Manager server crashes. Do SOC analysts lose the ability to search? Do logs stop being indexed?
3. An analyst's SPL query runs against 4 indexers. Where is the final result merged and returned from?
4. You have 2,000 Windows servers each running a UF. A new `inputs.conf` change needs to go to all of them. What component handles this, and how?

---

*Next: [Module 2a — Universal Forwarder Deep Dive](02a-universal-forwarder.md)*
