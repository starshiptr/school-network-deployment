# Production-Pattern School Network — Design & Deployment

Solo design and deployment of a segmented, security-hardened, monitored
network for a school (paid consulting engagement). One firewall, one
managed switch, ~100–150 endpoints, full observability, and a clean handover.

> School identity and all credentials/IPs are intentionally omitted or
> anonymized. This repo documents architecture and decisions, not secrets.

## Stack
Proxmox VE · pfSense CE · TP-Link Omada (EAP225) · pfBlockerNG · Suricata
· Graylog · Zabbix · Oxidized · Tailscale · WireGuard

## Architecture
![topology](docs/topology.png)

- 5 VLANs — MGMT / INFRA / LAB / STAFF, consistent switch↔firewall
- Default-deny inter-VLAN firewall, forced-DNS + DoH blocking
- WAN intrusion detection, centralized syslog, git-versioned config backups
- CGNAT-proof remote management (Tailscale subnet router)

## Highlights
- **L2 hardening:** DHCP snooping, BPDU Guard, Root Protect, storm control,
  port security
- **Defense in depth:** segmentation → DNS filtering → IDS → logging
- **Observability:** Graylog log pipeline + Zabbix monitoring + dashboards
- **Resilience:** Oxidized + pfSense AutoConfigBackup, documented handover

## What I built & debugged
- Diagnosed scrambled hypervisor NIC↔bridge mapping that kept severing host access
- Caught a pfSense Plus-vs-CE ISO trap that blocked all package installs
- Worked around ISP CGNAT (inbound WireGuard dead) → Tailscale subnet router
- Traced a lab outage to a damaged copper run forced to 100M

## Docs
- `docs/topology.png` — network diagram (sanitized)
- `docs/vlan-map.md` — VLAN/subnet/trust-zone table
- `docs/firewall-rationale.md` — rule logic and the "why" per rule
- `docs/runbook.md` — handover + remote-support procedure

## Skills demonstrated
Network segmentation · firewall engineering · IDS · DNS security ·
virtualization · centralized logging · monitoring · config automation ·
documentation & handover
