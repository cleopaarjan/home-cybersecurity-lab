# Cambium cnPilot E410 — Configuration Reference

## Setup

| Parameter | Value |
|---|---|
| Device | Cambium cnPilot E410 |
| Mode | Access Point (Bridge) |
| IP | 192.168.88.253 (static) |
| Admin Panel | http://192.168.88.253 |
| Gateway | 192.168.88.1 (MikroTik) |
| DNS | Handled by MikroTik — Pi-hole |
| PoE | 802.3af injector |

---

## WLAN 1 — 2.4GHz

| Setting | Value |
|---|---|
| SSID | Arjan's Lab |
| Band | 2.4GHz |
| Security | WPA2 Pre-shared Keys |
| Client Isolation | Local |
| Drop Multicast | Enabled |
| Hide SSID | No |
| Max Clients | 127 |

---

## WLAN 2 — 5GHz

| Setting | Value |
|---|---|
| SSID | Arjan's Lab 5G |
| Band | 5GHz |
| Channel Width | 40MHz |
| Security | WPA2 Pre-shared Keys |
| Client Isolation | Local |
| Drop Multicast | Enabled |

---

## Why Bridge/AP Mode Matters

The E410 operates purely as a wireless antenna extension of the MikroTik LAN. It performs no routing, no DHCP, no DNS. All of that is handled by MikroTik, meaning:

- WiFi clients get IPs from MikroTik's DHCP (192.168.88.x range)
- DNS goes to Pi-hole automatically
- All firewall rules on MikroTik apply equally to wired and wireless devices
- Client isolation prevents WiFi devices from attacking each other directly

---

## Network Role

```
MikroTik RB450G (router/DHCP/firewall)
        |
   [LAN port]
        |
  PoE Injector
        |
  cnPilot E410 (pure AP — bridge mode)
        |
  )))  WiFi  (((
  Arjan's Lab / Arjan's Lab 5G
```
