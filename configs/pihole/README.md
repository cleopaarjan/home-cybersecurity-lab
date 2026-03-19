# Pi-hole — Configuration Reference

## Setup

| Parameter | Value |
|---|---|
| Device | Raspberry Pi |
| IP | 192.168.88.249 (static) |
| Admin Panel | http://192.168.88.249/admin |
| DNS Role | Primary DNS for entire lab network |
| Fallback DNS | 8.8.8.8, 8.8.4.4 (Google) |

---

## Blocklists

| List | URL | Category |
|---|---|---|
| StevenBlack Hosts | `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts` | Core |
| OISD Big | `https://big.oisd.nl/` | Comprehensive |
| Hagezi Multi | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/multi.txt` | Ads + tracking |
| Hagezi TIF | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/tif.txt` | Threats |
| Phishing Army | `https://phishing.army/download/phishing_army_blocklist_extended.txt` | Phishing |
| Hagezi Pro | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.txt` | Privacy |
| Easyprivacy | `https://v.firebog.net/hosts/Easyprivacy.txt` | Tracking |
| WindowsSpyBlocker | `https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/spy.txt` | Windows telemetry |
| SmartTV | `https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/SmartTV.txt` | IoT |
| Easylist | `https://v.firebog.net/hosts/Easylist.txt` | Ads |
| AdguardDNS | `https://v.firebog.net/hosts/AdguardDNS.txt` | Ads |
| Hagezi Ultimate | `https://gitlab.com/hagezi/mirror/-/raw/main/dns-blocklists/adblock/ultimate.txt` | Nuclear option |

---

## Update Commands

```bash
# Full system + Pi-hole update
sudo apt update && sudo apt upgrade -y && sudo apt dist-upgrade -y && \
sudo apt autoremove -y && sudo apt autoclean && pihole -up && pihole -g
```

## Auto Update Cron (Weekly, Sunday 3AM)

```bash
0 3 * * 0 apt update && apt upgrade -y && apt autoremove -y && pihole -up && pihole -g
```

---

## MikroTik DNS Settings

```bash
# Point all lab DNS to Pi-hole with Google fallback
/ip dns set servers=192.168.88.249,8.8.8.8,8.8.4.4 allow-remote-requests=yes
/ip dhcp-server network set 0 dns-server=192.168.88.249

# Allow D-Link to push fallback DNS dynamically
/ip dhcp-client set [find interface=WAN] use-peer-dns=yes
```
