# Module 7 — Security Hardening (SOC-Relevant)

> **Status: 🔜 Coming**

## Topics Covered Here
- TLS everywhere: UF → HF → Indexer → Search Head (certificate setup)
- Splunk-to-Splunk certificate authentication (not just encryption — auth too)
- Role-Based Access Control (RBAC):
  - Built-in roles: admin, power, user, can_delete
  - Custom roles for SOC tiers (L1 read-only, L2 can run reports, L3 full access)
  - Index-level access control — analyst can only see their assigned indexes
- Index segmentation: separate indexes for Windows, Linux, network, firewall, auth
  - Why this matters: least privilege for data access in a multi-team SOC
- `_audit` index — every search, login, config change is logged here
  - How to build an admin activity dashboard from _audit
- `_introspection` index — system performance data
- Disabling unused inputs and services (attack surface reduction)
- Hardening the management port (8089) — restrict by IP
- LDAP/SAML integration for enterprise SSO
- Splunk's own log sources for SIEM detection (detecting lateral movement via Splunk admin actions)

---

*Content added when we reach this module.*
