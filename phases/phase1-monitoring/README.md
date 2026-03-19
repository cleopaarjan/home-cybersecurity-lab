# Phase 1 — Network Monitoring & Analysis

## Objective
Develop visibility into the lab network — understand what normal traffic looks like, identify devices, and build the habit of reading network data before attempting to secure or attack it.

---

## Tools Used

### 1. MikroTik Torch
Real-time traffic monitor built into RouterOS. Shows per-device bandwidth, protocols, source/destination IPs.

```bash
/tool torch interface=bridge-LAN
```

**What I learned:** Every device constantly generates background traffic — DNS queries, NTP syncs, telemetry pings. Understanding "normal" baseline traffic is essential for spotting anomalies.

---

### 2. Wireshark
Deep packet capture and analysis on the LAN. Set up port mirroring on MikroTik to capture all LAN traffic, not just traffic from my own PC.

**Port mirroring command:**
```bash
/interface ethernet switch rule add switch=switch1 \
  src-mac-address=00:00:00:00:00:00/00:00:00:00:00:00 \
  mirror=yes mirror-target=ether2
```

**Useful Wireshark filters:**
```
dns                          # All DNS queries
ip.src == 192.168.88.x       # Traffic from specific device
http                         # Unencrypted web traffic
tcp.flags.syn == 1           # TCP handshakes only
```

**Key observation:** Running Wireshark while browsing on a phone connected to the WiFi reveals dozens of background domain lookups to telemetry, analytics, and ad servers — most of which Pi-hole blocks.

---

### 3. Pi-hole Query Log
Every DNS query from every device logged and categorised. Accessible at `http://192.168.88.249/admin`.

**Blocklists configured:**

| List | Category |
|---|---|
| StevenBlack Hosts | Core ads/malware |
| OISD Big | Comprehensive |
| Hagezi Multi | Ads + tracking |
| Hagezi TIF | Threats/phishing |
| Phishing Army Extended | Phishing domains |
| Easyprivacy | Tracking |
| WindowsSpyBlocker | Windows telemetry |
| SmartTV Blocklist | IoT telemetry |

---

### 4. Grafana + InfluxDB Dashboard
Visual monitoring of Pi-hole stats and network metrics over time. Hosted on Raspberry Pi, accessible at `http://192.168.88.249:3000`.

- Grafana dashboard ID used: `14525`
- Shows query volume, block rate, top blocked domains, top clients

---

## Key Takeaways

- Packet analysis reveals what devices are *actually* doing vs what you think they're doing
- DNS is the most information-rich protocol for network visibility — every connection starts with a DNS query
- Pi-hole blocks ~20-30% of DNS queries on a typical home network — most of it telemetry and ads
- Establishing a traffic baseline makes anomaly detection possible in later phases
