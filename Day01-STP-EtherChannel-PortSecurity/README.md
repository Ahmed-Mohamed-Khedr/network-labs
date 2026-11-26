# Day 1 — Enterprise Campus Switching: STP Root Election, EtherChannel & Port Security

## Objective
Design a campus network that survives a core link failure without users noticing, and stops rogue devices at the access port.

## Topology
- 2x Cisco 3560 (Core1, Core2) — Layer 3 switches
- 2x Cisco 2960 (Access1, Access2) — dual-homed to both cores
- 4x PCs — Fa0/10 = VLAN 10, Fa0/20 = VLAN 20 on each access switch
- Core-to-core: 2 links bundled as Port-channel 1 (LACP)

## IP Plan
| Device | IP | VLAN |
|---|---|---|
| PC10 (Access1) | 192.168.10.10/24 | 10 Sales |
| PC10 (Access2) | 192.168.10.11/24 | 10 Sales |
| PC20 (Access1) | 192.168.20.20/24 | 20 HR |
| PC20 (Access2) | 192.168.20.21/24 | 20 HR |

## Configuration Highlights
- VLAN 10 (Sales) and VLAN 20 (HR) on all switches
- 802.1Q trunks on all inter-switch links (2960s: no encapsulation command)
- STP root split by design: Core1 root for VLAN 10 (priority 4096), Core2 root for VLAN 20 (priority 4096), each is backup for the other (8192)
- EtherChannel LACP between the cores: channel-group 1 mode active on Fa0/1 and Fa0/3
- Access ports: PortFast + BPDU Guard + Port Security (sticky, maximum 2, violation restrict)

## Verification
- show interfaces trunk — Po1 trunking, VLANs 1,10,20 active
- show spanning-tree vlan 10 — Core1: "This bridge is the root"
- show etherchannel summary — Po1(SU), LACP, Fa0/1(P) Fa0/3(P)
- show port-security interface fa0/10 — sticky MAC learned, Total = 1

## Failure Tests Performed
1. EtherChannel member failure: shut down Fa0/1 on Core1 — ping continued with 0% loss over the second member.
2. Port Security violation: connected a third unknown PC to Fa0/10 — traffic dropped (100% ping loss), %PORT_SECURITY-2-PSECURE_VIOLATION logged, violation count increased.

## Files
- Day1-STP-EtherChannel-PortSecurity.pkt — Packet Tracer file
- Screenshots: topology, verification outputs, failure tests
