# Aruba 2530-24P (J9773A) — wn-sw-tt-01

24-port PoE+ Layer 2 managed switch. Handles all 802.1Q VLAN tagging in the lab. pfSense connects via a tagged trunk on port 1 carrying all 5 VLANs.

## Specs

| | |
|--|--|
| Model | HP/Aruba J9773A (2530-24G-PoEP) |
| Hostname | wn-sw-tt-01 |
| Ports | 24x 10/100/1000 PoE+, 4x Gigabit (2x RJ45 + 2x SFP) |
| PoE budget | 195W total |
| Management | Web UI (HTTPS) + ProVision CLI (SSH) |
| Management IP | 10.10.10.2 (VLAN 10 untagged on port 24) |
| Firmware | ArubaOS-Switch YA.16.11.0028 |
| Default gateway | 10.10.10.1 (pfSense MGMT interface) |
| DNS domain | wittycomp.com |
| NTP | 10.10.10.1 (pfSense), 1.1.1.1 |

## Access

```bash
# SSH — password only, no command injection (requires PTY/interactive session)
ssh admin@10.10.10.2

# Console: USB-A to mini-USB cable to console port
# 9600 baud, 8N1, no flow control
```

Credentials: `admin` / (see Vaultwarden — item: *Aruba 2530 — wn-sw-tt-01*)

> **Note**: ArubaOS-Switch rejects direct command execution over SSH (`SSH command execution is not supported`).
> Use `expect` scripts or interactive sessions only.

## ProVision CLI Quick Reference

```bash
show running-config              # full config dump
show vlan                        # list all VLANs
show vlan <id>                   # VLAN detail + port membership
show interfaces brief            # all port link states
show lldp info remote-device all # discover neighbors (pfSense, AP, etc.)
show spanning-tree               # STP state
show power-over-ethernet brief   # PoE allocation per port

# Enter config mode:
config
  vlan 30                        # enter VLAN context
    tagged 5                     # add trunk/tagged membership
    untagged 9                   # access port: native VLAN
  exit
write memory                     # save config to flash
```

## VLAN Summary

See [vlan-config.md](vlan-config.md) for the full VLAN setup and ProVision commands.

| VLAN | Name | Key Ports |
|------|------|-----------|
| 1 | DEFAULT_VLAN | Ports 4,6,8-9,11-21,25-28 (unassigned access ports) |
| 10 | Witty-Mgmt-10 | Port 1 (tagged), Port 2 (native), Port 22/23 (native), Port 24 (native, mgmt) |
| 20 | Witty-Web-20 | Port 1 (tagged), Port 2 (tagged), Port 10 (native), Port 22/23 (tagged) |
| 30 | Witty-Data-30 | Port 1 (tagged), Port 2 (tagged), Port 3 (native), Port 22/23 (tagged) |
| 40 | IoT-Simple | Port 1 (tagged), Port 2 (tagged), Port 22/23 (tagged) |

## Port Assignments

See [port-assignments.md](port-assignments.md) for the full port map.

Key trunks:
- **Port 1**: pfSense uplink — VLAN 10/20/30/40 all tagged
- **Port 2**: wn-docker-01 — VLAN 10 native, 20/30/40 tagged
- **Port 22**: AP-515 (BearAir) — VLAN 10 native, 20/30/40 tagged; PoE critical (25.5W)

## STP Mode

RSTP (`spanning-tree force-version rstp-operation`). Edge ports (ports 3, 7, 22, 23) have `admin-edge-port` set; ports 22 and 23 have `bpdu-filter`/`bpdu-protection`.

## PoE Notes

AP-515 draws **25.5W** (PoE+ Type 2, confirmed via LLDP negotiation). Port 22 is set `power-over-ethernet critical`.

```bash
show power-over-ethernet brief   # check per-port consumption and budget
```

## Backup Config

```bash
# From switch CLI — export running config via TFTP:
copy running-config tftp 10.10.10.10 wn-sw-tt-01-backup.cfg

# Restore:
copy tftp running-config 10.10.10.10 wn-sw-tt-01-backup.cfg
```

Commit exported config to this repo after any structural change (VLAN add/remove, port reassignment).
