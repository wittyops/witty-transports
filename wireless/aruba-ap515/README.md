# Aruba AP-515

Wi-Fi 6 (802.11ax) access point running in **Instant AP (IAP) mode** — no controller required. The first AP powered on becomes the Virtual Controller (VC); additional APs join automatically.

## Specs

| | |
|--|--|
| Model | Aruba AP-515 |
| Wi-Fi | 802.11ax (Wi-Fi 6) dual-band |
| Radios | 2.4 GHz (4x4 MIMO) + 5 GHz (4x4 MIMO) |
| Max throughput | ~1.7 Gbps |
| PoE | 802.3at (PoE+), Class 4 ~15W |
| Management IP | 10.10.10.101 (VLAN 10, DHCP reservation) |
| Hostname | wn-ap-tt-01 |
| Mode | Aruba Instant (IAP) — standalone |

## Access

```
Web UI: https://10.10.10.101      (admin / <see Vaultwarden>)
Aruba Central: optional cloud mgmt — not currently enrolled
```

## Instant AP Mode — How It Works

In IAP mode, no external controller is needed. The VC handles all client associations, DHCP for clients, and SSID management. When you connect to the web UI at `10.10.10.101`, you're configuring the VC directly.

Key concepts:
- **Virtual Controller (VC)**: the AP that's running the management plane — whichever boots first
- **Cluster**: if you add more AP-515s, they discover the VC via multicast on VLAN 10 and join automatically
- **SSID**: each SSID is a separate "network" with its own security + VLAN assignment

## SSIDs

See [ssid-vlan-map.md](ssid-vlan-map.md) for full config.

| SSID | VLAN | Security | Purpose |
|------|------|----------|---------|
| WittyComp | 10 (MGMT) | WPA3-Personal / WPA2 mixed | Trusted devices — phones, laptops |
| WittyComp-IoT | 40 (IoT) | WPA2 | Smart home, cameras, printers |

## VLAN Tagging on the AP

When a client connects to "WittyComp-IoT", the AP tags the outgoing frame with VLAN 40 before sending it to the switch. The switch trunk port (connected to AP port 1) accepts both VLAN 10 and VLAN 40 tagged frames and forwards them to pfSense for routing.

```
Client → WittyComp-IoT SSID → AP tags frame VLAN 40 → Switch trunk
→ pfSense opt4 (IoT 10.10.40.x) → firewall: internet-only
```

## PoE

The AP is powered by the Aruba 2530-24P switch on port 1 (PoE+). No separate power adapter needed. The switch must have `PoE` enabled on that port (it's on by default for all PoE+ ports).

To check from switch CLI:
```bash
show power-over-ethernet 1
```

## Adding a Second AP

1. Connect new AP-515 to any PoE switch port
2. AP powers on and sends multicast discovery for VC on VLAN 10
3. Existing VC (10.10.10.101) responds; new AP joins cluster
4. SSIDs propagate automatically — no config needed on new AP
5. Assign a DHCP reservation for the new AP's MAC → update port-assignments.md
