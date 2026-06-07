# Operations Runbook

> Sanitized for portfolio. Procedures and structure only — no credentials,
> public IPs, Tailscale identifiers, or site identity. Real values live in the
> sealed handover document, not here.

## Remote access
Day-to-day management is over a **Tailscale subnet router** running on the
hypervisor host, which advertises the MGMT and INFRA subnets. This works
through the ISP's CGNAT with no inbound port-forwarding (inbound WireGuard is
not viable behind CGNAT). Admin reaches the firewall, hypervisor, switch,
logging, and monitoring UIs over the tailnet.

**Access is gated by:** Tailscale MFA + ACLs (who may reach the management
node), plus per-service authentication (firewall, hypervisor, and monitoring
each have their own login + 2FA where supported).

## On-site fallback
If Tailscale is ever unreachable, plug the admin laptop into the **designated
management switch port (untagged VLAN 10)** and reach every device by its
management IP directly. This is the break-glass path and is documented in the
handover.

## Backups
- **Switch config:** Oxidized container pulls the switch config hourly and
  git-versions every change (diff history retained).
- **Firewall config:** native auto-backup encrypts and stores a revision on
  every configuration change.
- **Full-VM backups:** pending dedicated backup storage on-site (planned).

## Routine checks
| Cadence | Check |
|---------|-------|
| Daily   | Intrusion-detection alerts (review, suppress noise, confirm no real hits) |
| Daily   | DNS-filter top-blocked report (confirm policy working, spot anomalies)    |
| Weekly  | Firewall block trends, top talkers, unexpected internal-to-internal hits  |
| Weekly  | Backup freshness — switch git commits + firewall revision list            |
| Monthly | Package/firmware updates on firewall and services (snapshot first)        |

## Common tasks
- **Add a device to a lab/staff VLAN:** plug into an access port already set to
  the right PVID; it pulls a DHCP lease automatically. No per-port change needed.
- **New SSID → VLAN:** in the wireless controller, set the SSID's VLAN ID to the
  target tag. The native/management SSID is left untagged.
- **Before any firewall/switch change:** snapshot the hypervisor VM (firewall)
  and confirm the config backup is current, so any lockout is recoverable.

## Known constraints
- Single firewall / switch / hypervisor — **no hardware redundancy**. Acceptable
  at this scale; a failure is a restore-from-backup event, not automatic failover.
- One lab cable run was previously forced to 100M as a workaround; re-termination
  to gigabit is the permanent fix.
- Second ISP line is planned to enable multi-WAN failover.

## Escalation
Tiered contact path (who to call, in order) is recorded in the sealed handover
package given to the site contact — not published here.
