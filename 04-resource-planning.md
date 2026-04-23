# Module 4 — Resource Planning & Sizing

> **Status: 🔜 Coming**

## Topics Covered Here
- Sizing formula: daily ingest GB → number of indexers needed
- Storage calculation: daily GB × retention days ÷ compression ratio (~10:1 for text logs)
- CPU/RAM profiles per role (from Splunk capacity planning guide)
- Hot vs. warm vs. cold storage — different disk class per tier
- Network bandwidth: estimating forwarder → indexer throughput
- HA minimums: RF=3 SF=2 for indexers, 3 nodes for SHC
- Dedicated network for indexer replication traffic
- Firewall rules table: all ports between all tiers
- Common mistakes: undersizing indexer disk, over-provisioning search heads
- Example: sizing a 50 GB/day enterprise SOC deployment end-to-end

---

*Content added when we reach this module.*
