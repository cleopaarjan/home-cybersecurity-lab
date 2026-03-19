# Phase 3 — Servers & Self Hosting

## Objective
Run real production-grade services across the lab network using Docker — building practical skills in containerization, Linux/Windows server administration, and self-hosted infrastructure. Every service hosted here also becomes a target for Phase 4 security testing.

---

## Infrastructure Overview

```
Lab Network (192.168.88.x)
 │
 ├── Raspberry Pi (192.168.88.249)
 │    ├── Pi-hole DNS
 │    └── WireGuard VPN
 │
 └── Windows PC (Docker Desktop)
      ├── Immich        — Self-hosted Google Photos (port 2283)
      ├── Ollama        — Local AI model runtime (port 11434)
      ├── Open WebUI    — Chat interface for local AI (port 3000)
      └── DVWA          — Vulnerable web app for pentesting (port 8080)
```

---

## 1. Docker on Windows PC

Docker Desktop for Windows runs Linux containers natively via WSL2. Foundation for all PC-hosted services.

### Installation
1. Download Docker Desktop from `https://docker.com`
2. Enable WSL2 when prompted
3. Restart PC
4. Verify:
```bash
docker --version
docker ps
```

### Why Docker
Each service is fully isolated — if one crashes or gets compromised it doesn't affect others. Containers rebuild in seconds, perfect for a lab environment where things get broken intentionally.

---

## 2. Immich — Self-Hosted Google Photos

### What It Is
A full Google Photos replacement running entirely on local hardware. Automatic photo/video backup from phone, AI-powered face recognition, object detection, timeline view, album sharing — all without any data leaving the network.

### Why Self-Host Photos
- Complete privacy — no cloud company accesses your photos
- No storage limits beyond your own drive
- Demonstrates multi-container app architecture, persistent storage, and database management

### Installation

```bash
mkdir C:\immich
cd C:\immich

curl -L https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml -o docker-compose.yml
curl -L https://github.com/immich-app/immich/releases/latest/download/example.env -o .env
```

Edit `.env`:
```env
UPLOAD_LOCATION=C:\immich\photos
DB_DATA_LOCATION=C:\immich\postgres
```

Start all containers:
```bash
docker compose up -d
```

### Containers Immich Runs

| Container | Role |
|---|---|
| immich-server | Main API and web interface |
| immich-microservices | Background jobs (thumbnails, encoding) |
| immich-machine-learning | AI face/object recognition |
| postgres | Metadata database |
| redis | Cache and job queue |

### Access
- Web UI: `http://192.168.88.x:2283`
- Mobile app: iOS/Android — point to your PC's IP for automatic photo backup over WiFi

### What I Learned
- Multi-container application architecture
- Persistent Docker volumes
- How ML models run as microservices
- Local network service accessibility across devices

---

## 3. Ollama — Local AI Model Runtime

### What It Is
Runs large language models (LLMs) entirely on local hardware — no internet required, no API costs, complete privacy. A local alternative to ChatGPT that never sends data anywhere.

### Installation
Download from `https://ollama.com` and run the installer. Ollama runs as a background service on port `11434`.

```bash
ollama --version
```

### Models Installed

```bash
ollama pull llama3
ollama pull mistral
ollama pull gemma
```

| Model | Size | Best For |
|---|---|---|
| Llama 3 | ~4.7GB | General purpose, strong reasoning |
| Mistral | ~4.1GB | Fast responses, instruction following |
| Gemma | ~5GB | Lightweight, Google's open model |

### CLI Usage
```bash
# Interactive chat in terminal
ollama run llama3

# List installed models
ollama list

# Check currently running models
ollama ps
```

### Ollama REST API
Ollama exposes a REST API — any app on the network can query it directly:
```bash
curl http://192.168.88.x:11434/api/generate -d '{
  "model": "llama3",
  "prompt": "Explain SQL injection simply",
  "stream": false
}'
```

This is how Open WebUI connects to it.

---

## 4. Open WebUI — Chat Interface for Local AI

### What It Is
A full-featured self-hosted chat UI for Ollama — essentially a private ChatGPT that talks exclusively to your local models. Supports model switching, conversation history, system prompts, and document uploads.

### Installation
```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  -v open-webui:/app/backend/data \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

`host.docker.internal` bridges the Docker container to Ollama running directly on Windows — without this the container can't reach a service outside of Docker.

### Access
`http://192.168.88.x:3000` — create admin account on first visit, select model from dropdown, start chatting. Everything runs locally on the PC.

### What I Learned
- How Docker containers communicate with host machine services
- REST API consumption (Open WebUI → Ollama API)
- How LLMs are served — request/response cycle, token streaming
- Practical differences between model sizes, speed, and quality
- Container networking — port mapping and `host.docker.internal`

---

## 5. DVWA — Damn Vulnerable Web Application

Intentionally vulnerable web app for practising attacks legally on self-owned infrastructure. Primary target for Phase 4 web application pentesting.

```bash
docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa
```

**Access:** `http://192.168.88.x:8080`  
**Login:** `admin / password` → click Create/Reset Database on first visit  
**Security levels:** Start at Low, progress to Medium then High as skills develop

---

## 6. WireGuard VPN (Raspberry Pi)

Encrypted tunnel for remote access into the entire lab from anywhere.

```bash
sudo apt install wireguard -y
wg genkey | tee server_private.key | wg pubkey > server_public.key
```

`/etc/wireguard/wg0.conf`:
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

MikroTik — open WireGuard port:
```bash
/ip firewall filter add chain=input protocol=udp dst-port=51820 action=accept comment="WireGuard VPN"
```

Once connected via VPN — Immich, Open WebUI, Pi-hole admin, DVWA all accessible remotely over an encrypted tunnel.

---

## Running Services Summary

| Service | Host | Port | Status |
|---|---|---|---|
| Pi-hole | Raspberry Pi | :80/admin | ✅ Running |
| WireGuard VPN | Raspberry Pi | :51820 UDP | ✅ Running |
| Immich | Windows PC | :2283 | ✅ Running |
| Ollama | Windows PC | :11434 | ✅ Running |
| Open WebUI | Windows PC (Docker) | :3000 | ✅ Running |
| DVWA | Windows PC (Docker) | :8080 | ✅ Running |
| Grafana + InfluxDB | Raspberry Pi | :3000 | 🔜 Planned |
| Nginx Reverse Proxy | Windows PC | :80/:443 | 🔜 Planned |

---

## Key Takeaways

- Docker makes deploying complex multi-container applications reproducible and manageable
- Self-hosting teaches every layer of an application — storage, networking, APIs, and UI — that cloud services normally hide
- Running AI locally proves powerful ML models don't require cloud infrastructure or subscriptions
- Every service hosted is also a potential attack surface — deploying it is the first step to understanding how to secure it
- WireGuard shows how clean modern VPN configuration is compared to legacy solutions like OpenVPN
