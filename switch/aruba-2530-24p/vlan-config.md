# Aruba 2530-24P — VLAN Configuration

Pulled from `show running-config` on 2026-06-06. This is the live state, not a template.

## VLANs Defined

| VLAN ID | Name (ProVision) | Purpose |
|---------|-----------------|---------|
| 1 | DEFAULT_VLAN | Unassigned/default — unpatched ports land here; limit use |
| 10 | Witty-Mgmt-10 | Management — servers, AP, trusted WiFi (BearFamily SSID) |
| 20 | Witty-Web-20 | Web-facing tier — wired WEB devices |
| 30 | Witty-Data-30 | Service containers (wn-caddy-01, wn-vault-01, all wn-*) |
| 40 | IoT-Simple | IoT devices — BearThings SSID, Samsung-F43 via AP enet1 |

## Running Config (extracted, 2026-06-06)

```
hostname "wn-sw-tt-01"
ip default-gateway 10.10.10.1
ip dns domain-name "wittycomp.com"
ip source-interface tftp vlan 10

vlan 1
   name "DEFAULT_VLAN"
   no untagged 1-3,5,7,10,22,24
   untagged 4,6,8-9,11-21,25-28
   tagged 23
   no ip address

vlan 10
   name "Witty-Mgmt-10"
   untagged 2,5,7,22-24
   tagged 1,20
   ip address 10.10.10.2 255.255.255.0
   ip igmp

vlan 20
   name "Witty-Web-20"
   untagged 10
   tagged 1-2,22-23
   no ip address

vlan 30
   name "Witty-Data-30"
   untagged 3
   tagged 1-2,22-23
   no ip address
   ip igmp

vlan 40
   name "IoT-Simple"
   tagged 1-2,22-23
   no ip address
```

> VLAN 40 has no untagged (access) ports on the switch itself — all IoT devices reach VLAN 40 through the AP's SSIDs or the AP's enet1 hardwired port, which tag the traffic before it hits port 22.

## Port VLAN Membership Summary

| Port | Native (untagged) | Tagged | Device |
|------|-------------------|--------|--------|
| 1 | — | 10, 20, 30, 40 | pfSense trunk uplink |
| 2 | 10 | 20, 30, 40 | wn-docker-01 |
| 3 | 30 | — | Samsung-F43-Hardwire (DATA) |
| 4 | 1 | — | Unassigned |
| 5 | 10 | — | Unknown MGMT device |
| 6 | 1 | — | Unassigned |
| 7 | 10 | — | Robin-Office-Hardline |
| 8-9 | 1 | — | Unassigned |
| 10 | 20 | — | Unknown WEB device |
| 11-21 | 1 | — | Unassigned |
| 22 | 10 | 20, 30, 40 | AP-515 (BearAir) trunk — PoE critical |
| 23 | 10 | 1, 20, 30, 40 | WN-MOB-TT-002 (LLDP) |
| 24 | 10 | — | Switch management access port |
| 25-28 | 1 | — | Uplinks/SFPs — unassigned |

## Security

```
ip authorized-managers 10.10.10.0 255.255.255.0 access manager
ip authorized-managers 10.10.20.0 255.255.255.0 access manager
ip authorized-managers 10.10.30.0 255.255.255.0 access operator
aaa authentication login privilege-mode
```

Management access is limited to MGMT and WEB VLANs; DATA VLAN gets operator (read-only) access.

## Adding a New VLAN

```bash
ssh admin@10.10.10.2
# (interactive login required)

config

# 1. Create VLAN
vlan 50
   name "Guest-50"
   exit

# 2. Tag it on trunk ports
vlan 50
   tagged 1       # pfSense uplink — creates subinterface there
   tagged 2       # docker host (if needed)
   tagged 22      # AP (if AP will serve this VLAN via SSID)
   exit

# 3. Access port example
vlan 50
   untagged 15    # device on port 15 gets VLAN 50 native
   exit

write memory
```

## Verify

```bash
show vlan                          # list all VLANs and member ports
show vlan 30                       # detail for DATA VLAN
show interfaces brief              # port link states
show lldp info remote-device all   # neighbor discovery
show spanning-tree                 # RSTP state
```
