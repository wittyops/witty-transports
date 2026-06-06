# Network Diagram

```
                         ┌─────────────────────────┐
                         │        INTERNET          │
                         └────────────┬────────────┘
                                      │ WAN
                         ┌────────────▼────────────┐
                         │     pfSense CE           │
                         │     wn-router-01         │
                         │     10.10.10.1           │
                         │  Inter-VLAN routing      │
                         │  DHCP · Firewall · DNS   │
                         └────────────┬────────────┘
                              802.1Q trunk (all VLANs tagged)
                         ┌────────────▼────────────┐
                         │   Aruba 2530-24P         │
                         │   Layer 2 Switch         │
                         │   VLAN tagging · PoE     │
                         └──┬──────┬──────┬────────┘
               PoE uplink   │      │      │
          ┌──────────────────┘      │      └────────────────────┐
          │                         │                            │
┌─────────▼────────┐     ┌──────────▼──────────┐    ┌──────────▼──────────┐
│  Aruba AP-515     │     │  wn-docker-01        │    │  Other wired         │
│  wn-ap-tt-01      │     │  10.10.10.10         │    │  MGMT devices        │
│  10.10.10.101     │     │  Docker host         │    │  (untagged VLAN 10)  │
│  IAP mode (VC)    │     │  VLAN 10 (MGMT)      │    └─────────────────────┘
│                   │     │                      │
│  SSID: WittyComp  │     │  macvlan bridge      │
│  → VLAN 10 (MGMT) │     │  ┌───────────────┐  │
│                   │     │  │  VLAN 30 DATA  │  │
│  SSID: WittyComp- │     │  │  10.10.30.x   │  │
│  IoT              │     │  │               │  │
│  → VLAN 40 (IoT)  │     │  │  wn-caddy-01  │  │
└───────────────────┘     │  │  .5 (macvlan) │  │
                          │  │               │  │
                          │  │  wn-vault-01  │  │
                          │  │  .4 (macvlan) │  │
                          │  │               │  │
                          │  │  ...50+ svcs  │  │
                          │  └───────────────┘  │
                          └─────────────────────┘

VLAN Flow:
  Client WiFi → AP tags frame with VLAN ID → Switch trunk → pfSense routes
  Container traffic → macvlan bridge on VLAN30 → Switch → pfSense (intra-VLAN: switch only)
  External → Cloudflare tunnel → wn-tunnel-01 (witty_network bridge) → Caddy (172.18.0.100)
```

## Key Traffic Paths

### External HTTPS request (e.g., cloud.wittycomp.com)
```
Internet → Cloudflare CDN → CF Tunnel (3e0a5181...) → wn-tunnel-01 container
→ witty_network bridge → Caddy at 172.18.0.100:443 → SNI route → upstream service
```

### Internal LAN request (e.g., cloud.wittycomp.com from phone on WiFi)
```
Phone (VLAN 10) → AP → Switch → pfSense → DNS: 10.10.10.10 → returns 10.10.30.5
→ Switch → Caddy at 10.10.30.5:443 → upstream service
```

### Container-to-container (e.g., Radarr → SABnzbd)
```
Radarr container (VLAN 30) → macvlan → Switch (same VLAN, no pfSense needed)
→ SABnzbd container (VLAN 30)
```
> Note: macvlan containers on the same host **cannot** reach each other via VLAN30 IPs
> due to Linux macvlan isolation. Use container hostnames on the witty_network bridge,
> or route via the host shim at 10.10.30.250.
