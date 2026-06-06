# Aruba 2530-24P — VLAN Configuration

## VLANs Defined

| VLAN ID | Name | Purpose |
|---------|------|---------|
| 1 | DEFAULT_VLAN | Default (limit use — anything untagged lands here) |
| 10 | MGMT | Management, wired trusted devices, WiFi trusted clients |
| 20 | WEB | Web-facing tier |
| 30 | DATA | Service containers (wn-caddy-01, wn-vault-01, etc.) |
| 40 | IoT | IoT devices, smart home, IP cameras |

## ProVision Commands to Apply

```bash
# SSH into switch
ssh admin@10.10.10.2

config

# Create VLANs
vlan 10 name "MGMT"
vlan 20 name "WEB"
vlan 30 name "DATA"
vlan 40 name "IoT"

# Set switch management IP on VLAN 10
vlan 10
  ip address 10.10.10.2 255.255.255.0
exit

# Default gateway (pfSense MGMT interface)
ip default-gateway 10.10.10.1

write memory
```

## Trunk Port Setup

The uplink to pfSense must carry all VLANs tagged. pfSense terminates each VLAN as a subinterface.

```bash
config

# Uplink port to pfSense — example: port 24 (adjust to your actual uplink)
vlan 1
  untagged 24      # VLAN 1 native on trunk (or remove from access)
vlan 10
  tagged 24
vlan 20
  tagged 24
vlan 30
  tagged 24
vlan 40
  tagged 24

write memory
```

## AP-515 Trunk Port

The AP runs multiple SSIDs, each mapped to a different VLAN. The switch port connected to the AP must be a trunk carrying VLAN 10 (trusted WiFi) and VLAN 40 (IoT WiFi).

```bash
config

# AP-515 port — example: port 1 (adjust to your cable)
vlan 10
  tagged 1
vlan 40
  tagged 1
# Native VLAN for AP management traffic:
vlan 10
  untagged 1      # AP management IP (10.10.10.101) is on VLAN 10 untagged

write memory
```

## Docker Host Port (wn-docker-01)

The docker host needs VLAN 10 for its own management IP and VLAN 30 for macvlan container traffic.

```bash
config

# wn-docker-01 port — example: port 2
vlan 10
  untagged 2     # host management (10.10.10.10) — untagged = native
vlan 30
  tagged 2       # macvlan containers get tagged VLAN 30 frames

write memory
```

## Access Ports (End Devices)

Most wired devices get a single untagged VLAN — they never see VLAN tags.

```bash
config

# Example: IoT device (smart TV) on port 10
vlan 1
  no untagged 10   # remove from default VLAN first
vlan 40
  untagged 10      # native IoT VLAN

write memory
```

## Verify Config

```bash
show vlan                          # list all VLANs
show vlan 30                       # see which ports are tagged/untagged for VLAN 30
show interfaces brief              # port link states
show lldp info remote-device all   # see what's connected to each port
```
