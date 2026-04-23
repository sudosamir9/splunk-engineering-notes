# Module 2d — Search Head Cluster (SHC) Deep Dive

> **Status: 🔜 Coming**

## Topics Covered Here
- How a search is executed: compile SPL → fan out to indexers → merge results
- SHC internals: Captain election (Raft consensus), member roles
- Knowledge object replication: dashboards, saved searches, lookups shared across SHs
- KV Store (MongoDB-backed): what it stores, why it matters for ES and UEBA
- Dispatch directory: where in-progress and completed searches live
- Search concurrency limits and how to tune them
- Load balancer in front of SHC — what type to use, sticky sessions
- Resource profile: 8-16 CPU, 16-32 GB RAM
- Deploying apps to SHC via Search Head Deployer (SHD)
- CLI commands for SHC management
- Cert exam touchpoints

---

*Content added when we reach this module.*
