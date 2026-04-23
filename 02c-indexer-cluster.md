# Module 2c — Indexer & Indexer Cluster Deep Dive

> **Status: 🔜 Coming**

## Topics Covered Here
- Indexer core job: receive → parse → index → store → serve
- Bucket lifecycle: hot → warm → cold → frozen → thawed
- Bucket types and disk layout (`$SPLUNK_DB`)
- Indexer clustering: replication factor (RF), search factor (SF)
- What RF=3 SF=2 actually means — data durability vs. search availability
- Cluster Manager role in replication
- Peer node states: up, down, detention, restarting
- Rolling restart procedure
- SmartStore (S3/object storage cold tier) — enterprise scale
- `indexes.conf` — defining indexes, retention policy, storage paths
- Resource profile: 12-16 CPU, 32-64 GB RAM, SSD for hot/warm
- Disk sizing formula: daily ingest × retention days ÷ compression ratio
- CLI commands for indexer/cluster management
- Cert exam touchpoints

---

*Content added when we reach this module.*
