# Split-Horizon DNS

The lab uses split-horizon DNS so internal clients resolve `*.wittycomp.com` directly to the internal Caddy IP (`10.10.30.5`) instead of routing through the Cloudflare tunnel. External clients still go through the CF tunnel normally.

## How It Works

```
Internal client
  │
  ├── DNS query: home.wittycomp.com
  ▼
Technitium DNS (10.10.10.10)
  │
  ├── Zone: wittycomp.com (authoritative, internal)
  ├── A record: home.wittycomp.com → 10.10.30.5  ← direct to Caddy
  │
  ▼
Caddy at 10.10.30.5:443
  │
  └── SNI: home.wittycomp.com → reverse proxy → 10.10.30.45:3000

External client
  │
  ├── DNS query: home.wittycomp.com
  ▼
Cloudflare DNS
  │
  ├── CNAME: home.wittycomp.com → 3e0a5181...cfargotunnel.com
  ▼
Cloudflare CDN
  │
  ▼
CF Tunnel → wn-tunnel-01 → Caddy (172.18.0.100:443)
```

## Technitium DNS Zone

All subdomains have explicit A records pointing to `10.10.30.5` (Caddy macvlan IP).

| Record | Type | Value | Purpose |
|--------|------|-------|---------|
| `wittycomp.com` | SOA/NS | — | Zone root |
| `*.wittycomp.com` | A | 10.10.30.5 | Wildcard — catch-all to Caddy |
| `home.wittycomp.com` | A | 10.10.30.5 | Explicit (preferred over wildcard) |
| *[all other subdomains]* | A | 10.10.30.5 | Explicit records per service |

## pfSense Unbound Override

pfSense Unbound has a domain override so any client querying pfSense directly (not Technitium) also gets the right result:
```
Domain: wittycomp.com
IP: 10.10.10.10   ← forward to Technitium
```

Applied 2026-06-04. Verify from any VLAN:
```bash
nslookup home.wittycomp.com 10.10.10.1   # should return 10.10.30.5
nslookup home.wittycomp.com 10.10.10.10  # Technitium direct, same result
```

## Technitium API — Add Record

```bash
TOKEN=$(curl -sk "https://10.10.10.10:53443/api/user/login?user=admin&pass=7403B3ars!" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['token'])")

curl -sk "https://10.10.10.10:53443/api/zones/records/add?\
token=$TOKEN&zone=wittycomp.com&domain=<name>.wittycomp.com\
&type=A&ipAddress=10.10.30.5&ttl=3600"
```

> Token expires between sessions — re-authenticate each time.
> Stored token in Vaultwarden item "Technitium DNS - API Token" is a reference copy only.

## Cloudflare DNS (External)

Each subdomain also has a CF DNS CNAME pointing to the tunnel:
- CNAME: `<name>.wittycomp.com` → `3e0a5181-c09b-49a6-bf3a-3a33af664e87.cfargotunnel.com`
- Proxied: true

This is handled automatically by the `cf-add-subdomain.sh` script in [witty-workflow/scripts](http://git.wittycomp.com/bearboss/witty-workflow/src/branch/main/scripts/cf-add-subdomain.sh).
