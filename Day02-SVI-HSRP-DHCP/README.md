# Day 02 — Inter-VLAN Routing with SVIs, HSRP & DHCP

## Scenario
Built on the Day 1 campus topology (2 core 3560, 2 access 2960, VLAN 10 Sales / VLAN 20 HR).
Goal: route between VLANs at the core, remove the default gateway as a single point of failure, and automate IP addressing.

## What I built
- SVIs on both cores: Core1 = .2, Core2 = .3 in each VLAN, with `ip routing` enabled
- HSRP virtual gateways: 192.168.10.1 and 192.168.20.1
  - Core1 Active for VLAN 10 (priority 110, preempt)
  - Core2 Active for VLAN 20 (priority 110, preempt) — load shared between the cores
- DHCP pools on Core1 (SALES / HR) with infrastructure addresses .1–.25 excluded
- All PCs switched to DHCP — verified leases: 192.168.10.26 and 192.168.20.26 with the virtual gateway assigned automatically

## Verification
- `show standby brief` — Active/Standby split between the cores as designed
- `show ip route` — both VLAN networks directly connected
- Cross-VLAN ping between DHCP clients passed 4/4 in both directions (TTL 127)

## Failure test
- Shut down Core1's VLAN 10 SVI → Core2 took over as HSRP Active (Standby -> Active), ping continued with 0% loss
- `no shutdown` → preempt restored Core1 as Active automatically

## Troubleshooting note
The first inter-VLAN ping failed. Isolated step by step (gateway pings, routing table, host settings) — root cause was a missing default gateway on the PC, the exact manual error DHCP eliminates.

## Files
- Day2-SVI-HSRP-DHCP.pkt — Packet Tracer topology
- 1.png / 2.png / 3.png — verification screenshots
