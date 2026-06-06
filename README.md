# witty-transports

Network transport layer for the **Wittycomp Lab** — VLAN topology, physical switch configuration, wireless AP setup, and pfSense interface definitions. This is the foundation everything else runs on.

## Why a Separate Repo

Network changes have a different blast radius than application changes. A misconfigured trunk port can segment the entire lab. Config here is versioned separately so rollback is clean and changes are auditable independently of service deployments.

## Hardware

| Device | Model | Role | Management IP |
|--------|-------|------|--------------|
| Router | pfSense CE | Inter-VLAN routing, DHCP, firewall | 10.10.10.1 |
| Switch | Aruba 2530-24P (J9773A) | Layer 2 switching, VLAN tagging, PoE | 10.10.10.2 |
| Access Point | Aruba AP-515 | Wi-Fi 6 (802.11ax), Instant AP mode | 10.10.10.101 |
| DNS | Technitium (wn-docker-01) | Split-horizon DNS, DHCP authoritative | 10.10.10.10 |

## VLAN Map (Quick Reference)

| VLAN ID | Name | Subnet | Gateway | Purpose |
|---------|------|--------|---------|---------|
| 1 | LAN | 10.10.1.0/24 | 10.10.1.1 | Base LAN — wired home devices |
| 10 | MGMT | 10.10.10.0/24 | 10.10.10.1 | Management — servers, APs, docker host, WiFi trusted clients |
| 20 | WEB | 10.10.20.0/24 | 10.10.20.1 | Web-facing tier (future use) |
| 30 | DATA | 10.10.30.0/24 | 10.10.30.1 | All service containers (Caddy, Vaultwarden, etc.) |
| 40 | IoT | 10.10.40.0/24 | 10.10.40.1 | IoT devices — isolated, internet-only |

Full IP registry: [wittycomp-lab/SERVICES.md](http://git.wittycomp.com/bearboss/wittycomp-lab/src/branch/main/SERVICES.md)

## Structure

```
topology/           VLAN map, network diagram, traffic flow
switch/
  aruba-2530-24p/   Port assignments, VLAN config, trunk setup, ProVision CLI reference
wireless/
  aruba-ap515/      Instant AP setup, SSID → VLAN mapping, PoE notes
firewall/
  pfsense/          Interface → VLAN assignments, key firewall rules, DHCP scopes
dns/
  split-horizon.md  Technitium zones + pfSense Unbound overrides
```

## Related Repos

| Repo | Layer |
|------|-------|
| [wittycomp-lab](http://git.wittycomp.com/bearboss/wittycomp-lab) | Service layer — Docker compose, Caddyfile |
| [witty-workflow](http://git.wittycomp.com/bearboss/witty-workflow) | Operational runbooks |
| [witty-blueprints](http://git.wittycomp.com/bearboss/witty-blueprints) | Architecture diagrams |

## Change Protocol

Before any switch or AP config change:
1. Document the current state (screenshot or `show running-config` export)
2. Make the change
3. Test reachability: `ping` between VLANs you care about
4. Commit the new state to this repo with a meaningful message

Never make switch changes from memory — always verify port assignments here first.
