# 🏠 Home Cybersecurity Lab

A fully isolated home network lab built for learning networking, security hardening, self-hosting, and penetration testing — all on real hardware.

---

## 📡 Network Architecture

```
ISP
 └── D-Link DIR-615 (Primary Network — 192.168.0.x)
          └── MikroTik RB450G (Isolated Lab — 192.168.88.x)
                   ├── Cambium cnPilot E410 (WiFi AP — "Arjan's Lab")
                   └── Raspberry Pi
                            ├── Pi-hole DNS (192.168.88.249)
                            └── Docker Services
```

---

## 🧰 Hardware

| Device | Role | IP |
|---|---|---|
| D-Link DIR-615 | Primary router / ISP gateway | 192.168.0.1 |
| MikroTik RB450G | Secondary router / Lab gateway | 192.168.88.1 |
| Cambium cnPilot E410 | Wireless Access Point | 192.168.88.253 |
| Raspberry Pi | DNS + Docker host | 192.168.88.249 |

---

## 🔒 Key Features

- **Double NAT isolation** — lab network completely isolated from primary network
- **Pi-hole DNS filtering** — network-wide ad/tracker/malware blocking with Google DNS fallback
- **Stateful MikroTik firewall** — proper connection state tracking and WAN protection
- **Client isolation on WiFi** — wireless devices cannot communicate with each other directly
- **Port knocking** — Winbox access protected by secret knock sequence
- **Cowrie honeypot** — SSH honeypot logging real attack attempts
- **VLANs** — pentesting VLAN (192.168.90.x) isolated from main lab

---

## 📚 Phases

| Phase | Topic | Status |
|---|---|---|
| [Phase 1](./phases/phase1-monitoring/) | Network Monitoring & Analysis | ✅ Complete |
| [Phase 2](./phases/phase2-security/) | Firewall & Security Hardening | ✅ Complete |
| [Phase 3](./phases/phase3-selfhosting/) | Servers & Self Hosting | 🔄 In Progress |
| [Phase 4](./phases/phase4-pentesting/) | Hacking & Pentesting | 🔜 Upcoming |

---

## 🛠️ Configs

- [MikroTik RB450G](./configs/mikrotik/)
- [Pi-hole](./configs/pihole/)
- [Cambium cnPilot E410](./configs/cambium/)

---

## 🎯 Goals

- Build practical cybersecurity skills on real hardware
- Document a professional portfolio for university applications
- Work toward a career in cybersecurity / network security

---

> ⚠️ **Disclaimer:** All pentesting and security testing is performed exclusively on self-owned hardware within an isolated private network. Nothing here targets systems without explicit ownership/permission.
