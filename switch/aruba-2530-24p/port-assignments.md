# Aruba 2530-24P — Port Assignments

Pulled from switch `show running-config` + `show lldp info remote-device all` on 2026-06-06.

## Port Map

| Port | Name / Device | Native VLAN | Tagged VLANs | Mode | Notes |
|------|--------------|-------------|--------------|------|-------|
| 1 | pfSense uplink (wn-router-01) | — | 10, 20, 30, 40 | Trunk | All-VLAN trunk to pfSense vtnet0 |
| 2 | wn-docker-01 | 10 | 20, 30, 40 | Trunk | MGMT for host NIC; macvlan containers use VLAN 30 |
| 3 | Samsung-F43-Hardwire | 30 | — | Access | Direct-cabled device; DATA VLAN |
| 4 | *unassigned* | 1 | — | Access | — |
| 5 | *unknown MGMT device* | 10 | — | Access | Named in VLAN 10 untagged list; no LLDP data |
| 6 | *unassigned* | 1 | — | Access | — |
| 7 | Robin-Office-Hardline | 10 | — | Access | Wired MGMT drop — Robin's office |
| 8–9 | *unassigned* | 1 | — | Access | — |
| 10 | *unknown WEB device* | 20 | — | Access | VLAN 20 native; no LLDP data |
| 11–21 | *unassigned* | 1 | — | Access | Available |
| 22 | AP-515 / BearAir | 10 | 20, 30, 40 | Trunk | PoE critical; STP edge + BPDU filter; 25.5W |
| 23 | WN-MOB-TT-002 | 10 | 1, 20, 30, 40 | Trunk | LLDP detected; unknown device class |
| 24 | Switch management | 10 | — | Access | Switch's own CLI/web access port |
| 25–26 | SFP uplinks | 1 | — | — | Not in use |
| 27–28 | RJ45 uplinks | 1 | — | — | Not in use |

## PoE Budget

| Port | Device | PoE Class | Negotiated Draw |
|------|--------|-----------|-----------------|
| 22 | AP-515 (BearAir) | Type 2 PD | **25.5W** (LLDP confirmed) |
| — | — | — | — |
| **Total** | | | **25.5W / 195W** |

Port 22 is set `power-over-ethernet critical` — it will shed lower-priority PoE devices first if budget is exceeded.

## Named Interfaces

From `interface` blocks in running config:
```
interface 3  → "Samsung-F43-Hardwire"
interface 7  → "Robin-Office-Hardline"
interface 22 → "BearAir"    (AP-515; PoE critical; DLDP enabled; STP edge + BPDU filter)
```

## LLDP Neighbors (2026-06-06)

| Switch Port | Remote Device | Remote IP | Description |
|-------------|--------------|-----------|-------------|
| 22 | 9c:8c:d8:c9:2a:9c | 10.10.10.198 | AOS-8 (MODEL: 515) — AP-515 |
| 23 | WN-MOB-TT-002 | — | Unknown; no system caps reported |

> Note: The AP's LLDP-reported IP (10.10.10.198) differs from its management DHCP reservation (10.10.10.101). Both respond to ping; `.101` is the VC management address.

## Cable Run Log

| Date | Port | Device | Notes |
|------|------|--------|-------|
| 2026-06-06 | 1 | pfSense (wn-router-01) | Trunk uplink to router |
| 2026-06-06 | 2 | wn-docker-01 | Server NIC — host + container VLAN trunk |
| 2026-06-06 | 3 | Samsung-F43-Hardwire | DATA VLAN access port |
| 2026-06-06 | 7 | Robin-Office-Hardline | MGMT VLAN access port |
| 2026-06-06 | 22 | AP-515 (BearAir) | AP trunk + PoE |

> Update this log whenever a cable is patched or moved.
