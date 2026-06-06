# Aruba AP-515 — BearHomeNet

Wi-Fi 6 (802.11ax) access point running in **Instant AP (IAP) mode** — no controller required. The AP is the sole member of its cluster and serves as the Virtual Controller (VC).

## Specs

| | |
|--|--|
| Model | Aruba AP-515 |
| MAC | 9c:8c:d8:c9:2a:9c |
| VC Name | BearHomeNet-C9:2A:9C |
| VC IP | 10.10.10.101 (DHCP reservation on VLAN 10) |
| Wi-Fi | 802.11ax (Wi-Fi 6) dual-band |
| Radios | 2.4 GHz (4x4 MIMO) + 5 GHz (4x4 MIMO) |
| Firmware | AOS-8, Version 8.13.1.1 LSR (built 2025-11-27) |
| Ethernet | 2x ports: enet0 (uplink trunk) + enet1 (F43-Hardwire access) |
| Switch port | Port 22 on wn-sw-tt-01 (PoE critical, 25.5W) |
| Cluster size | 1 AP (single-AP cluster) |

## Access

```
Web UI:  https://10.10.10.101   (admin / <see Vaultwarden>)
SSH:     ssh admin@10.10.10.101  (requires interactive PTY)
```

## Instant AP Mode

In IAP mode, no external Aruba Mobility Controller is needed. This AP is the VC for the cluster `BearHomeNet`. The `allowed-ap` list contains only its own MAC, so no additional APs can auto-join without explicit approval.

```
show network        # list SSIDs and client counts
show clients        # connected client inventory
show version        # firmware + uptime
show running-config # full config dump
```

## SSIDs

See [ssid-vlan-map.md](ssid-vlan-map.md) for full config.

| SSID | VLAN | Security | Purpose |
|------|------|----------|---------|
| BearFamily | 10 (Witty-Mgmt-10) | WPA2-PSK AES | Trusted — phones, laptops, servers |
| BearGuest | Default (VLAN 1) | Enhanced-Open (OWE) | Guests — internet-only intent |
| BearThings | 40 (IoT-Simple) | WPA2-AES | IoT devices — 2.4 GHz only |

## Ethernet Ports

The AP-515 has two wired ports:

| Port | Profile | Mode | VLAN | Purpose |
|------|---------|------|------|---------|
| enet0 | default_wired_port_profile | Trunk | all (native: 1) | Uplink to switch port 22 |
| enet1 | F43-Hardwire | Access | 40 (native) | Hardwired Samsung F43 device — IoT VLAN |

The enet1 port is a wired downlink on the AP itself — a device plugged into it lands on VLAN 40 without needing to be on the BearThings SSID.

## VLAN Tagging Flow

```
BearFamily client → AP tags VLAN 10 frame → switch port 22 (VLAN 10 native) → pfSense
BearThings client → AP tags VLAN 40 frame → switch port 22 (VLAN 40 tagged) → pfSense
BearGuest client  → AP uses Default VLAN → switch port 22 (VLAN 1 native) → pfSense
enet1 device      → AP tags VLAN 40 frame → switch port 22 (VLAN 40 tagged) → pfSense
```

## ARM (Adaptive Radio Management)

| Setting | Value |
|---------|-------|
| 5 GHz band steering | `prefer-higher-band` |
| 80 MHz support | enabled |
| TX power 2.4 GHz | 6–9 dBm |
| TX power 5 GHz | 12–18 dBm |
| Client-aware | enabled |

## Adding a Second AP

1. Connect new AP-515 to any PoE switch port
2. AP powers on, sends multicast discovery for VC on VLAN 10
3. VC (10.10.10.101) must have the new AP's MAC in its `allowed-ap` list:
   ```
   # On existing VC via SSH:
   allowed-ap <new-ap-mac>
   ```
4. SSIDs propagate automatically once the AP joins
5. Assign a DHCP reservation for the new AP's MAC and update port-assignments.md
