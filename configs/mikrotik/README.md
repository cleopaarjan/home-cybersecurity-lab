# MikroTik RB450G — Configuration Reference

## Network Overview

| Parameter | Value |
|---|---|
| Device | MikroTik RB450G |
| RouterOS | 6.49.5 |
| WAN Interface | ether1 (connected to D-Link LAN) |
| LAN Bridge | bridge-LAN (ether2–ether5) |
| LAN Subnet | 192.168.88.0/24 |
| Gateway IP | 192.168.88.1 |
| DHCP Range | 192.168.88.10–192.168.88.254 |
| DNS | 192.168.88.249 (Pi-hole), 8.8.8.8, 8.8.4.4 (fallback) |

---

## Interface Setup

```bash
# Rename interfaces
/interface set ether1 name=WAN
/interface set ether2 name=LAN2
/interface set ether3 name=LAN3

# Create LAN bridge
/interface bridge add name=bridge-LAN
/interface bridge port add bridge=bridge-LAN interface=LAN2
/interface bridge port add bridge=bridge-LAN interface=LAN3
/interface bridge port add bridge=bridge-LAN interface=ether4
/interface bridge port add bridge=bridge-LAN interface=ether5

# Assign LAN IP
/ip address add address=192.168.88.1/24 interface=bridge-LAN

# WAN via DHCP from D-Link
/ip dhcp-client add interface=WAN disabled=no use-peer-dns=yes
```

---

## DHCP Server

```bash
/ip pool add name=LAN-pool ranges=192.168.88.10-192.168.88.254
/ip dhcp-server add name=LAN-dhcp interface=bridge-LAN address-pool=LAN-pool disabled=no
/ip dhcp-server network add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=192.168.88.249
```

---

## NAT

```bash
/ip firewall nat add chain=srcnat out-interface=WAN action=masquerade
```

---

## DNS

```bash
/ip dns set servers=192.168.88.249,8.8.8.8,8.8.4.4 allow-remote-requests=yes
```

---

## Firewall Rules

```bash
# Input chain
/ip firewall filter add chain=input connection-state=established,related action=accept
/ip firewall filter add chain=input connection-state=invalid action=drop
/ip firewall filter add chain=input src-address=192.168.88.0/24 action=accept
/ip firewall filter add chain=input protocol=icmp action=accept
/ip firewall filter add chain=input in-interface=WAN action=drop

# Forward chain
/ip firewall filter add chain=forward connection-state=established,related action=accept
/ip firewall filter add chain=forward connection-state=invalid action=drop
/ip firewall filter add chain=forward dst-address=192.168.0.0/24 action=drop

# Port knocking for Winbox
/ip firewall filter add chain=input protocol=tcp dst-port=1234 action=add-src-to-address-list address-list=knock1 address-list-timeout=10s
/ip firewall filter add chain=input protocol=tcp dst-port=5678 src-address-list=knock1 action=add-src-to-address-list address-list=knock2 address-list-timeout=10s
/ip firewall filter add chain=input protocol=tcp dst-port=9012 src-address-list=knock2 action=add-src-to-address-list address-list=allowed-winbox address-list-timeout=1h
/ip firewall filter add chain=input protocol=tcp dst-port=8291 src-address-list=allowed-winbox action=accept
/ip firewall filter add chain=input protocol=tcp dst-port=8291 action=drop

# WireGuard VPN
/ip firewall filter add chain=input protocol=udp dst-port=51820 action=accept
```

---

## Bandwidth Limit

```bash
/queue simple add name=lab-limit target=192.168.88.0/24 max-limit=20M/20M
```

---

## VLAN — Pentest Segment

```bash
/interface vlan add name=VLAN-pentest vlan-id=10 interface=bridge-LAN
/ip address add address=192.168.90.1/24 interface=VLAN-pentest
/ip pool add name=pentest-pool ranges=192.168.90.10-192.168.90.254
/ip dhcp-server add name=pentest-dhcp interface=VLAN-pentest address-pool=pentest-pool disabled=no
/ip dhcp-server network add address=192.168.90.0/24 gateway=192.168.90.1 dns-server=192.168.88.249
/ip firewall filter add chain=forward src-address=192.168.90.0/24 dst-address=192.168.88.0/24 action=drop
```

---

## Static DHCP Leases

```bash
# Pi-hole
/ip dhcp-server lease make-static [find mac-address="XX:XX:XX:XX:XX:XX"]
/ip dhcp-server lease set [find mac-address="XX:XX:XX:XX:XX:XX"] comment="Pi-hole"

# cnPilot E410
/ip dhcp-server lease set [find address="192.168.88.253"] comment="cnPilot E410"
```

---

## Router Identity

```bash
/system identity set name="MikroTik-Lab"
```
