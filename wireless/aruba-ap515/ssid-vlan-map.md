# Aruba AP-515 — SSID → VLAN Map

## SSIDs

### WittyComp (Trusted)

| Setting | Value |
|---------|-------|
| SSID | `WittyComp` |
| VLAN | 10 (MGMT) |
| Client subnet | 10.10.10.x |
| Security | WPA3-Personal (WPA2 fallback for older devices) |
| Band | 2.4 GHz + 5 GHz (band steering enabled) |
| Hidden | No |
| Purpose | Trusted personal devices — phones, laptops, tablets |

Devices on this SSID get MGMT VLAN access, which means `pass → any` firewall rule applies. They can reach all services at wittycomp.com internally via Technitium DNS.

### WittyComp-IoT

| Setting | Value |
|---------|-------|
| SSID | `WittyComp-IoT` |
| VLAN | 40 (IoT) |
| Client subnet | 10.10.40.x |
| Security | WPA2-Personal (many IoT devices don't support WPA3) |
| Band | 2.4 GHz preferred (most IoT is 2.4 GHz only) |
| Hidden | No |
| Purpose | Smart home devices, IP cameras (future), printers, Chromecasts |

Devices on this SSID are isolated — pfSense firewall rule: `IoT → WAN allow, IoT → LAN/MGMT/DATA deny`.

## IAP Web UI — SSID Configuration Path

1. Log in: `https://10.10.10.101` → admin credentials from Vaultwarden
2. **Configuration → Networks → Add New Network**
3. Set SSID name, security settings
4. Under **VLAN**: choose "Client VLAN assignment → Static → [VLAN ID]"
5. Apply

## Future SSIDs to Consider

| SSID | VLAN | Use Case |
|------|------|---------|
| `WittyComp-Cam` | 40 (IoT) or new Camera VLAN | IP cameras once deployed; could use dedicated VLAN if camera traffic needs more isolation |
| `WittyComp-Guest` | New Guest VLAN (50?) | Short-term guest access; internet-only, isolated from everything |
| `WittyComp-Media` | 10 (MGMT) | Dedicated band for Chromecast/Apple TV if band steering causes issues |

## Aruba Instant AP CLI (advanced)

If web UI is unavailable, SSH into the VC:
```bash
ssh admin@10.10.10.101

# Show all SSIDs and their VLAN assignments:
show network

# Show connected clients:
show clients

# Show AP cluster:
show ap database

# Reboot VC (all APs reboot):
reboot
```
