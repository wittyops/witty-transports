# pfSense — Firewall Rules

Key rules that govern inter-VLAN and WAN traffic. Full rules live in pfSense UI — this documents intent and verifies nothing breaks when making changes.

## Rule Summary by Interface

### LAN (VLAN 1)
| Action | Source | Dest | Port | Purpose |
|--------|--------|------|------|---------|
| Allow | LAN | DATA (10.10.30.0/24) | 80, 443 | Access services via Caddy |
| Allow | LAN | Any | Any | Default LAN allow |

### MGMT (VLAN 10) — opt1
| Action | Source | Dest | Port | Purpose |
|--------|--------|------|------|---------|
| Allow | MGMT | Any | Any | Full access — trusted admin VLAN |

### WEB (VLAN 20) — opt2
| Action | Source | Dest | Port | Purpose |
|--------|--------|------|------|---------|
| Allow | WEB | DATA (10.10.30.0/24) | Any | WEB-to-DATA shim |
| Allow | WEB | WAN | Any | Internet access |

### DATA (VLAN 30) — opt3
| Action | Source | Dest | Port | Purpose |
|--------|--------|------|------|---------|
| Allow | DATA | DATA | Any | Intra-service (limited — macvlan containers use bridge) |
| Allow | DATA | WAN | Any | Containers pulling updates, external APIs |
| Block | DATA | MGMT | Any | Containers cannot reach management hosts |

### IoT (VLAN 40) — opt4
| Action | Source | Dest | Port | Purpose |
|--------|--------|------|------|---------|
| Allow | IoT | WAN | Any | Internet access for smart devices |
| Block | IoT | LAN | Any | Cannot reach home devices |
| Block | IoT | MGMT | Any | Cannot reach servers |
| Block | IoT | DATA | Any | Cannot reach services |
| Block | IoT | WEB | Any | Cannot reach web tier |

> IoT isolation is the most important rule set. A compromised smart device or IP camera cannot pivot to the service layer.

## Unbound DNS Override

pfSense Unbound is configured with a domain override:
```
wittycomp.com → 10.10.10.10 (Technitium)
```

This means any device on any VLAN that queries pfSense's DNS (or Technitium directly) for `*.wittycomp.com` gets the internal Caddy IP (`10.10.30.5`) back, bypassing the Cloudflare tunnel.

Applied 2026-06-04. Verify: `nslookup home.wittycomp.com 10.10.10.1` should return `10.10.30.5`.

## Access pfSense Config (XMLRPC)

```python
import xmlrpc.client
proxy = xmlrpc.client.ServerProxy(
    "https://admin:7403B3ars!@10.10.10.1/xmlrpc.php",
    allow_none=True
)
# Backup a config section:
result = proxy.pfsense.backup_config_section(["filter"])
```

> Note: `exec_php` and `listMethods` are NOT available on this pfSense version.
> Available: `backup_config_section`, `restore_config_section`, `filter_configure`
