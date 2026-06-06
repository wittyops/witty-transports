# Aruba 2530-24P — Port Assignments

Pulled from switch `show running-config` + `show lldp info remote-device all` on 2026-06-06.
Physical port assignments confirmed by bearboss 2026-06-06.

## Port Map

| Port | Name / Device | Native VLAN | Tagged VLANs | Mode | Notes |
|------|--------------|-------------|--------------|------|-------|
| 1 | ISP ONT | — | 10, 20, 30, 40 | Trunk | WAN uplink; switch carries ISP path + VLAN tags to pfSense |
| 2 | wn-srv-01 | 10 | 20, 30, 40 | Trunk | MGMT for host NIC (10.10.10.10); macvlan containers on VLAN 30 |
| 3 | Samsung-F43-Hardwire | 30 | — | Access | Directly cabled device; DATA VLAN |
| 4 | *unassigned* | 1 | — | Access | Available |
| 5 | Sonos speaker | 10 | — | Access | MGMT VLAN (trusted audio device) |
| 6 | Sonos speaker | 10 | — | Access | MGMT VLAN — moved from VLAN 1 on 2026-06-06 for Sonos group consistency |
| 7 | Robin-Office-Hardline | 10 | — | Access | Wired MGMT drop — Robin's office |
| 8–9 | *unassigned* | 1 | — | Access | Available |
| 10 | *unknown WEB device* | 20 | — | Access | VLAN 20 native; no LLDP data |
| 11–21 | *unassigned* | 1 | — | Access | Available |
| 22 | AP-515 / BearAir | 10 | 20, 30, 40 | Trunk | PoE critical; STP edge + BPDU filter; 25.5W |
| 23 | WN-MOB-TT-002 | 10 | 1, 20, 30, 40 | Trunk | LLDP detected; device identity TBD |
| 24 | Laptop dock (built-in NIC) | 10 | — | Access | bearboss workstation dock; MGMT VLAN |
| 25–26 | SFP uplinks | 1 | — | — | Not in use |
| 27–28 | RJ45 uplinks | 1 | — | — | Not in use |


## PoE Budget

| Port | Device | PoE Class | Negotiated Draw |
|------|--------|-----------|-----------------|
| 22 | AP-515 (BearAir) | Type 2 PD | **25.5W** (LLDP confirmed) |
| — | — | — | — |
| **Total** | | | **25.5W / 195W** |

Port 22 is set `power-over-ethernet critical` — it will shed lower-priority PoE devices before cutting power to the AP.

## Named Interfaces (from running config)

```
interface 3  → "Samsung-F43-Hardwire"
interface 7  → "Robin-Office-Hardline"
interface 22 → "BearAir"    (AP-515; PoE critical; DLDP enabled; STP edge + BPDU filter)
```

## LLDP Neighbors (2026-06-06)

| Switch Port | Remote Device | Remote IP | Description |
|-------------|--------------|-----------|-------------|
| 22 | 9c:8c:d8:c9:2a:9c | 10.10.10.198 | AOS-8 (MODEL: 515) — AP-515 |
| 23 | WN-MOB-TT-002 | — | No system caps; device identity TBD |

> AP's LLDP-reported IP (10.10.10.198) differs from its management DHCP reservation (10.10.10.101). Both respond; `.101` is the VC management address used for SSH/web.

## Cable Run Log

| Date | Port | Device | Notes |
|------|------|--------|-------|
| 2026-06-06 | 1 | ISP ONT | WAN uplink |
| 2026-06-06 | 2 | wn-srv-01 | Docker host — MGMT + DATA VLAN trunk |
| 2026-06-06 | 3 | Samsung-F43-Hardwire | DATA VLAN access |
| 2026-06-06 | 5 | Sonos speaker | MGMT VLAN access |
| 2026-06-06 | 6 | Sonos speaker | Default VLAN (needs review) |
| 2026-06-06 | 6 | Sonos speaker | Moved VLAN 1 → VLAN 10 for Sonos group consistency |
| 2026-06-06 | 7 | Robin-Office-Hardline | MGMT VLAN access |
| 2026-06-06 | 22 | AP-515 (BearAir) | AP trunk + PoE |
| 2026-06-06 | 24 | Laptop dock | MGMT VLAN access — bearboss workstation |

> Update this log whenever a cable is patched or moved.
