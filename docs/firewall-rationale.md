# Firewall Rule Rationale

> Sanitized architecture doc. Explains the *logic* behind the ruleset, not a
> config export. No public IPs, credentials, or site identity.

## How the firewall thinks
- **Default deny.** If no rule explicitly allows traffic, it is dropped.
- **Stateful.** Allowed outbound flows have their return traffic permitted
  automatically; you only write rules for the initiating direction.
- **Top-down, first match wins.** Rule order matters — specific blocks sit
  above general allows.
- **Least privilege per zone.** Each VLAN gets only what it needs.

## Aliases (reusable rule objects)
- **`RFC1918`** = `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` — i.e. "all
  internal networks." Used to block a zone from reaching *any* other internal
  subnet in one rule.
- **`DoH_Servers`** = a list of known DNS-over-HTTPS resolver IPs. Used to stop
  clients bypassing local DNS filtering by tunnelling DNS inside HTTPS.

## Per-zone intent

### Student labs (VLAN 20, VLAN 30) — User trust
1. **Allow** DNS to the local gateway only — forces all name resolution through
   the filtered resolver.
2. **Block** to "This Firewall" — labs cannot reach the firewall's admin
   surfaces.
3. **Block** to `RFC1918` — labs cannot reach any other internal VLAN
   (no lateral movement to MGMT, INFRA, STAFF, or each other across VLANs).
4. **Block** to `DoH_Servers` on 443 — kills DNS-over-HTTPS bypass.
5. **Allow** to the internet — general browsing, after the blocks above.

### Staff (VLAN 50) — Medium trust
- Same DNS-forcing + DoH block as labs.
- **Block** internal (`RFC1918`) — staff devices don't need to reach lab or
  infrastructure subnets.
- **Allow** internet.

### Infrastructure (VLAN 15) — High trust
- Allow DNS to gateway.
- **Block** the user VLANs (labs, staff).
- Allow onward (reaches MGMT for centralized monitoring/management of APs).

### Management (VLAN 10) — Highest trust
- Full administrative access. This is where the admin workstation, firewall,
  hypervisor, switch management, logging, and monitoring live.

## Forced DNS (NAT redirect)
A destination-NAT rule on each user VLAN rewrites **any** outbound traffic to
port 53 — regardless of the destination the client typed — to the local
resolver. This catches devices hardcoded to `8.8.8.8`/`1.1.1.1` and guarantees
every DNS query is filtered. Combined with the DoH block, clients cannot opt
out of content filtering.

## Defense-in-depth summary
Segmentation (VLANs) → default-deny inter-VLAN firewall → forced local DNS →
DNS filtering (blocklists + SafeSearch) → DoH bypass block → WAN intrusion
detection → centralized logging. Each layer assumes the one before it might be
evaded.
