# Aruba AP-515 — SSID → VLAN Map

Pulled from `show running-config` on 2026-06-06. Three SSIDs active.

## SSIDs

### BearFamily (Trusted)

| Setting | Value |
|---------|-------|
| SSID | `BearFamily` |
| VLAN | Default (no `vlan` directive in profile → switch port 22 native = VLAN 10) |
| Client subnet | 10.10.10.x |
| Security | WPA2-PSK AES |
| Band | 2.4 GHz + 5 GHz (band steering: prefer 5 GHz) |
| Type | employee |
| Max clients | 64 |
| Purpose | Trusted personal devices — phones, laptops, tablets, Sonos, smart TVs |

Clients on this SSID get MGMT VLAN. pfSense rule: `MGMT → any allow`. They resolve `*.wittycomp.com` to `10.10.30.5` via Technitium.

### BearGuest (Internet-only)

| Setting | Value |
|---------|-------|
| SSID | `BearGuest` |
| VLAN | Default (VLAN 1 on switch; falls through to pfSense LAN interface) |
| Security | Enhanced-Open (OWE — encrypted but no passphrase) |
| Band | 2.4 GHz + 5 GHz |
| Type | guest |
| Max clients | 64 |
| Purpose | Short-term guests; internet access; no passphrase needed |

> BearGuest currently lands on DEFAULT_VLAN (1), which may reach the LAN subnet. A dedicated Guest VLAN (50?) with `IoT → WAN only` rules is the correct long-term posture.

### BearThings (IoT)

| Setting | Value |
|---------|-------|
| SSID | `BearThings` |
| VLAN | **40** (explicitly set in SSID profile) |
| Client subnet | 10.10.40.x |
| Security | WPA2-AES |
| Band | **2.4 GHz only** (most IoT devices are 2.4 GHz) |
| Auth | InternalServer (AP-local) |
| Type | employee |
| Max clients | 64 |
| Purpose | Smart home devices, printers, IP cameras (future), Chromecasts |

Clients on this SSID are isolated by pfSense IoT firewall rules: `IoT → WAN allow; IoT → LAN/MGMT/DATA deny`.

## Wired Port: F43-Hardwire (enet1)

The AP's second ethernet port runs a separate wired profile:

| Setting | Value |
|---------|-------|
| Port | enet1 on AP |
| Profile | F43-Hardwire |
| Mode | Access |
| VLAN | **40** (native, no PoE) |
| Access rule | deny all (restrictive) |
| Purpose | Samsung F43 device hardwired directly to AP port — IoT VLAN |

## Currently Connected Clients (2026-06-06)

| Name | IP | MAC | SSID | Band/Channel |
|------|----|-----|------|--------------|
| Galaxy-S22-Ultra | 10.10.10.106 | 52:0d:2d:c5:1b:82 | BearFamily | 5 GHz ch36E |
| wn-elec-01 | 10.10.10.130 | e4:5f:01:73:6e:6b | BearFamily | 5 GHz AC |
| SonosZP | 10.10.10.51 | 00:0e:58:7a:8a:54 | BearFamily | 2.4 GHz ch1 |
| Display1-[BearShow] | 10.10.10.100 | 44:5c:e9:2a:91:b5 | BearFamily | 5 GHz ch36+ |

All 4 active clients are on BearFamily. BearThings and BearGuest currently have 0 clients.

## IAP Web UI — SSID Configuration Path

1. Log in: `https://10.10.10.101` → admin credentials from Vaultwarden
2. **Configuration → Networks → Add New Network**
3. Set SSID name, security settings
4. Under **VLAN**: choose "Client VLAN assignment → Static → [VLAN ID]"
5. Apply

## Planned Future SSIDs

| SSID | VLAN | Use Case |
|------|------|---------|
| BearGuest (fix) | 50 (new Guest VLAN) | Proper isolation — internet-only, no pfSense LAN reach |
| BearCam | 40 (IoT) or new Camera VLAN | IP cameras when deployed; may warrant VLAN 50 if traffic segregation needed |

## Aruba Instant AP CLI

```bash
ssh admin@10.10.10.101

show network          # all SSIDs, clients, VLAN assignments
show clients          # connected client list with IP/MAC/signal
show version          # firmware, uptime
show running-config   # full config dump

# Reboot all APs in cluster:
reboot ap all
```
