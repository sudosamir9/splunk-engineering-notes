# Module 5 — Deployment & Configuration

> **Status: 🔜 Coming**

## Topics Covered Here
- Installation order (critical — wrong order breaks clustering):
  1. License Manager
  2. Cluster Manager
  3. Indexer peers (join the cluster)
  4. Search Heads (join the SHC)
  5. Search Head Deployer
  6. Deployment Server
  7. Heavy Forwarders
  8. Universal Forwarders
- Installing Splunk on Linux — directory layout, `$SPLUNK_HOME`
- Initial bootstrap commands for each role
- `server.conf` — the most important config file for clustering
- Config file hierarchy deep dive — system/default vs. local vs. app
- `btool` — how to debug config merging and precedence
- Deploying apps to indexers (via Cluster Manager bundle push)
- Deploying apps to search heads (via Search Head Deployer)
- Deploying apps to forwarders (via Deployment Server)
- Common misconfiguration pitfalls

---

*Content added when we reach this module.*
