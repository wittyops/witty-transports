# pfSense — Interface → VLAN Assignments

pfSense CE at `10.10.10.1` (wn-router-01). Connects to the Aruba 2530-24P via a tagged trunk port carrying all VLANs.

## Interfaces

| pfSense Interface | VLAN | Subnet | Gateway | Role |
|------------------|------|--------|---------|------|
| WAN | — | DHCP from ISP | ISP | Internet uplink |
| LAN | 1 | 10.10.1.0/24 | 10.10.1.1 | Base LAN (legacy wired home devices) |
| opt1 (MGMT) | 10 | 10.10.10.0/24 | 10.10.10.1 | Management — servers, AP, trusted WiFi |
| opt2 (WEB) | 20 | 10.10.20.0/24 | 10.10.20.1 | Web-facing tier (reserved, limited use) |
| opt3 (DATA) | 30 | 10.10.30.0/24 | 10.10.30.1 | Service containers — all `wn-*` Docker services |
| opt4 (IoT) | 40 | 10.10.40.0/24 | 10.10.40.1 | IoT devices, smart home, future cameras |

## VLAN Subinterfaces

pfSense creates VLAN subinterfaces on the physical trunk NIC:
```
em0.10 → opt1 (MGMT)
em0.20 → opt2 (WEB)
em0.30 → opt3 (DATA)
em0.40 → opt4 (IoT)
```

## DHCP Scopes

Each interface runs a DHCP scope. All scopes serve Technitium (`10.10.10.10`) as primary DNS.

| Interface | DHCP Range | DNS Primary | DNS Secondary |
|-----------|-----------|-------------|--------------|
| LAN | 10.10.1.100–250 | 10.10.10.10 | 10.10.1.1 |
| MGMT | 10.10.10.100–200 | 10.10.10.10 | 10.10.10.1 |
| WEB | 10.10.20.100–200 | 10.10.10.10 | 10.10.20.1 |
| DATA | 10.10.30.100–200 | 10.10.10.10 | 10.10.30.1 |
| IoT | 10.10.40.100–200 | 10.10.10.10 | 10.10.40.1 |

> DNS set to Technitium so `*.wittycomp.com` resolves to `10.10.30.5` (Caddy) for all VLANs — no CF tunnel hairpin.

## DHCP Static Mappings (key devices)

| MAC | IP | Hostname | VLAN |
|-----|----|---------|------|
| *(see pfSense UI)* | 10.10.10.2 | aruba-switch | MGMT |
| *(see pfSense UI)* | 10.10.10.10 | wn-docker-01 | MGMT |
| *(see pfSense UI)* | 10.10.10.101 | wn-ap-tt-01 | MGMT |

See [pfSense firewall-rules.md](firewall-rules.md) for traffic policies between VLANs.
