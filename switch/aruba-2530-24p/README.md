# Aruba 2530-24P (J9773A)

24-port PoE+ Layer 2 managed switch. Handles all VLAN tagging in the lab. pfSense connects via a trunk uplink carrying all 5 VLANs.

## Specs

| | |
|--|--|
| Model | HP/Aruba J9773A (2530-24P) |
| Ports | 24x 10/100 PoE+, 2x Gigabit uplink, 2x SFP |
| PoE budget | 195W total |
| Management | Web UI + ProVision CLI (SSH/console) |
| Management IP | 10.10.10.2 (VLAN 10, static) |
| Firmware OS | ProVision (ArubaOS-Switch) |

## Access

```bash
# SSH (enable SSH in switch config first)
ssh admin@10.10.10.2

# Console: USB-A to mini-USB cable to console port
# 9600 baud, 8N1, no flow control
```

Default credentials: `admin` / *(blank or set password — store in Vaultwarden)*

## ProVision CLI Quick Reference

```bash
show running-config          # full config dump
show vlan                    # list all VLANs
show vlan <id>               # VLAN detail + port membership
show interfaces brief        # all ports status
show lldp info remote-device # discover neighbors (pfSense, AP, etc.)
show spanning-tree           # STP state
show power-over-ethernet     # PoE allocation per port

# Enter config mode:
config
  vlan 30 name "DATA"        # rename a VLAN
  vlan 30                    # enter VLAN context
    tagged 24                # tag port 24 for VLAN 30 (trunk)
    untagged 5               # access port: untagged = native VLAN
  exit
write memory                 # save config to flash
```

## VLAN Configuration

See [vlan-config.md](vlan-config.md) for the full VLAN setup.

Quick state:
- **Uplink to pfSense**: tagged all VLANs (trunk port)
- **AP-515 port**: tagged VLAN 10 + 40 (trunk for multi-SSID)
- **wn-docker-01 port**: tagged VLAN 10 + 30 (MGMT for host, DATA for macvlan containers)
- **Other wired devices**: untagged on their respective VLAN (access ports)

## PoE Notes

AP-515 draws ~15W max (PoE+ Class 4). Budget 200mW headroom per port.

```bash
show power-over-ethernet brief   # check consumption
```

## Backup / Restore Config

```bash
# From switch CLI — copy running config to TFTP:
copy running-config tftp 10.10.10.10 witty-switch-backup.cfg

# Restore:
copy tftp running-config 10.10.10.10 witty-switch-backup.cfg
```

Commit the exported config to this repo after every significant change.
