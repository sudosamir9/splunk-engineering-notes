# Module 6 — CLI Management & Day-2 Operations

> **Status: 🔜 Coming**

## Topics Covered Here
- `splunk start | stop | restart | status`
- `splunk btool <conf> list [stanza] --debug` — the #1 troubleshooting command
- `splunk show config` — dump effective merged config
- `splunk list forward-server` — verify forwarder targets
- `splunk list peer-servers` — check indexer cluster peer status
- `splunk show cluster-status` — full cluster health
- `splunk add index` / `splunk edit index` — index management
- `splunk add monitor` — add a file monitor on the fly
- `splunk reload deploy-server` — force Deployment Server refresh
- Rolling restarts for indexer cluster upgrades
- Log file locations: `$SPLUNK_HOME/var/log/splunk/`
  - `splunkd.log` — main daemon log
  - `metrics.log` — performance counters
  - `audit.log` — user action audit trail
- Health check runbook: what to check when SOC reports missing data

---

*Content added when we reach this module.*
