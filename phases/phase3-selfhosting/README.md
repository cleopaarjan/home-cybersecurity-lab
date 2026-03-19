# Phase 3 — Servers & Self Hosting

## Objective
Run real services on the lab network using Docker — creating actual targets to later attack in Phase 4, while learning Linux administration, containerization, web servers, and VPN configuration.

---

## 1. Docker on Raspberry Pi

Docker runs applications in isolated containers — lightweight, portable, and production-standard. Learning Docker directly translates to cloud skills (AWS ECS, Azure Container Instances, GCP Cloud Run).

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

---

## 2. DVWA — Damn Vulnerable Web Application

An intentionally vulnerable PHP web app for practicing web application attacks legally on your own infrastructure.

```bash
docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa
```

**Access:** `http://192.168.88.249:8080`  
**Credentials:** `admin / password`

**Vulnerabilities practised:**

| Vulnerability | What it teaches |
|---|---|
| SQL Injection | Database extraction via malformed queries |
| XSS (Reflected) | Session hijacking via injected JavaScript |
| XSS (Stored) | Persistent payload injection |
| Command Injection | OS command execution through web inputs |
| File Upload | Bypassing extension filters to upload shells |
| CSRF | Forging authenticated requests |
| Brute Force | Credential stuffing against login forms |

---

## 3. WireGuard VPN

Encrypted tunnel for remote access into the lab from anywhere. Modern, fast, and far simpler than OpenVPN.

**Server config (`/etc/wireguard/wg0.conf`):**
```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <server_private_key>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client_public_key>
AllowedIPs = 10.0.0.2/32
```

**MikroTik — open WireGuard port:**
```bash
/ip firewall filter add chain=input protocol=udp dst-port=51820 action=accept
```

**Result:** Full access to `192.168.88.x` lab from any device, anywhere, over an encrypted tunnel.

---

## 4. Nginx Reverse Proxy

Routes traffic to the correct container based on URL rather than requiring separate ports for every service.

```bash
docker run -d -p 80:80 --name nginx-proxy nginx
```

**Before reverse proxy:**
```
http://192.168.88.249:8080  → DVWA
http://192.168.88.249:3000  → Grafana
http://192.168.88.249/admin → Pi-hole
```

**After reverse proxy:**
```
http://dvwa.lab     → DVWA
http://grafana.lab  → Grafana
http://pihole.lab   → Pi-hole
```

---

## Key Takeaways

- Docker makes spinning up and tearing down services trivial — perfect for a lab environment
- Hosting intentionally vulnerable apps creates realistic attack targets without legal risk
- WireGuard demonstrates how VPNs work at a config level — key exchange, tunneling, routing
- Reverse proxies are fundamental to how every real web infrastructure routes traffic
- Every service you host teaches you both how to build it AND how to think about securing it
