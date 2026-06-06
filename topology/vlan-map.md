# VLAN Map

Authoritative VLAN definitions for the Wittycomp Lab. Physical tagging is done by the Aruba 2530-24P switch; routing between VLANs is handled by pfSense at the trunk uplink.

## VLAN Definitions

| VLAN ID | pfSense Interface | Name | Subnet | Gateway | DHCP Range | DNS |
|---------|------------------|------|--------|---------|------------|-----|
| 1 | LAN | LAN | 10.10.1.0/24 | 10.10.1.1 | 10.10.1.100–250 | 10.10.10.10 |
| 10 | opt1 (MGMT) | MGMT | 10.10.10.0/24 | 10.10.10.1 | 10.10.10.100–200 | 10.10.10.10 |
| 20 | opt2 (WEB) | WEB | 10.10.20.0/24 | 10.10.20.1 | 10.10.20.100–200 | 10.10.10.10 |
| 30 | opt3 (DATA) | DATA | 10.10.30.0/24 | 10.10.30.1 | 10.10.30.100–200 | 10.10.10.10 |
| 40 | opt4 (IoT) | IoT | 10.10.40.0/24 | 10.10.40.1 | 10.10.40.100–200 | 10.10.10.10 |

> All VLANs use Technitium at `10.10.10.10` as primary DNS. This ensures `*.wittycomp.com` resolves to `10.10.30.5` (Caddy) internally instead of routing through the Cloudflare tunnel.

## Traffic Policy Summary

| Source VLAN | Destination | Policy | Reason |
|-------------|-------------|--------|--------|
| LAN (1) | DATA (30) | Allow 80/443 | Access services via Caddy |
| MGMT (10) | Any | Allow all | Trusted — servers, admin hosts |
| WEB (20) | DATA (30) | Allow (shim) | Web-tier to services |
| DATA (30) | DATA (30) | Allow | Container-to-container |
| IoT (40) | Internet only | Allow WAN, deny LAN/MGMT/DATA | Isolation |
| IoT (40) | DATA (30) | Deny | IoT cannot reach services |

## Key Static Assignments (MGMT / DATA)

These devices use static IPs (DHCP reservations or manual) outside the DHCP range:

| IP | Device | VLAN |
|----|--------|------|
| 10.10.10.1 | pfSense router (wn-router-01) | MGMT |
| 10.10.10.2 | Aruba 2530-24P switch management IP | MGMT |
| 10.10.10.10 | wn-docker-01 host + Technitium DNS | MGMT |
| 10.10.10.101 | Aruba AP-515 management IP (wn-ap-tt-01) | MGMT |
| 10.10.30.1 | DATA VLAN gateway (pfSense opt3) | DATA |
| 10.10.30.5 | wn-caddy-01 (primary reverse proxy, macvlan) | DATA |
| 10.10.30.250 | Host shim — host-process services port binding | DATA |

See [witty-workflow/reference/ip-allocation.md](http://git.wittycomp.com/bearboss/witty-workflow/src/branch/main/reference/ip-allocation.md) for full DATA VLAN service registry.

## SSID → VLAN Mapping (AP-515)

| SSID | VLAN | Client Subnet | Notes |
|------|------|--------------|-------|
| WittyComp (trusted) | MGMT (10) | 10.10.10.x | Trusted devices — phones, laptops |
| WittyComp-IoT | IoT (40) | 10.10.40.x | Smart home, cameras, printers |

See [wireless/aruba-ap515/ssid-vlan-map.md](../wireless/aruba-ap515/ssid-vlan-map.md) for full AP config.
