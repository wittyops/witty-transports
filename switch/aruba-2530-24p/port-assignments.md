# Aruba 2530-24P — Port Assignments

**Status**: template — fill in as cables are run. Update this file and commit whenever you patch a cable.

## Port Map

| Port | Device | VLAN | Mode | Notes |
|------|--------|------|------|-------|
| 1 | Aruba AP-515 (wn-ap-tt-01) | 10 (native) + 40 (tagged) | Trunk | PoE powers AP; native VLAN 10 for AP mgmt |
| 2 | wn-docker-01 | 10 (native) + 30 (tagged) | Trunk | MGMT for host, DATA for macvlan containers |
| 3–22 | *unassigned / end devices* | varies | Access | Update as devices connect |
| 23 | *reserved* | 10 | Access | Spare MGMT port |
| 24 | pfSense (wn-router-01) | all tagged | Trunk | All-VLAN trunk uplink |
| 25–26 | SFP uplinks | — | — | Not in use |

> Ports 3–22 are access ports assigned per-device. Document each cable run here.
> Always check this file before patching to avoid VLAN mismatches.

## PoE Budget Tracking

| Port | Device | PoE Class | Draw |
|------|--------|-----------|------|
| 1 | AP-515 | Class 4 | ~15W |
| — | — | — | — |
| **Total** | | | **~15W / 195W** |

## Cable Run Log

| Date | Port | Device | Run by | Notes |
|------|------|--------|--------|-------|
| 2026-06-06 | 1 | AP-515 | bearboss | Initial setup |
| 2026-06-06 | 24 | pfSense uplink | bearboss | Trunk to router |
| 2026-06-06 | 2 | wn-docker-01 | bearboss | Server NIC |
