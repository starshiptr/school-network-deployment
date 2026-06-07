# VLAN Map & Trust Zones

> Sanitized. Private RFC1918 addressing only — no public IPs, MACs, hostnames,
> or site identity. The third octet maps to the VLAN ID for readability.

## Design principle
Each user group lives in its own broadcast domain (VLAN) with its own subnet.
Trust decreases as you move away from management; the firewall enforces what
each zone may reach. Switch and firewall VLAN definitions are kept identical.

## VLAN table

| VLAN | Name   | Subnet           | Gateway        | Trust   | Purpose                                  |
|------|--------|------------------|----------------|---------|------------------------------------------|
| 10   | MGMT   | 192.168.10.0/24  | 192.168.10.1   | Highest | Firewall, hypervisor, switch mgmt, admin |
| 15   | INFRA  | 192.168.15.0/24  | 192.168.15.1   | High    | Access points, wireless controller       |
| 20   | LAB1   | 192.168.20.0/24  | 192.168.20.1   | User    | Student lab 1                            |
| 30   | LAB23  | 192.168.30.0/24  | 192.168.30.1   | User    | Student labs 2 & 3 (merged — see note)   |
| 50   | STAFF  | 192.168.50.0/24  | 192.168.50.1   | Medium  | Staff devices                            |

Reserved for future use: Guest (60), Printer (70).

## Trunk & tagging
- **VLAN 10 (MGMT) is the untagged native VLAN** on the firewall↔switch trunk.
- VLANs 15/20/30/50 are **802.1Q tagged** on the trunk and on AP uplink ports.
- Lab/staff access ports are untagged into their single VLAN (PVID set per port).

## Note — why Labs 2 & 3 share one VLAN
The downstream switches feeding Labs 2 and 3 are **unmanaged** and cannot
process 802.1Q tags. Rather than run additional cabling or replace hardware,
Labs 2 and 3 were accepted at the **same trust level** on a single VLAN (30).
This is a documented, deliberate trade-off — intra-VLAN visibility between
those two rooms is by design; isolation from all other VLANs still holds.

## Addressing convention
- `192.168.<VLAN>.1` = gateway (firewall interface) for every VLAN.
- DHCP pool per VLAN (e.g. `.100`–`.200`); infrastructure devices static below `.100`.
- /24 per VLAN — ample headroom (busiest segment ~80 devices vs 254 capacity).
