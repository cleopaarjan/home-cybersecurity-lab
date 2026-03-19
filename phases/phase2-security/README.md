# Phase 2 — Firewall & Security Hardening

## Objective
Lock down the MikroTik router with a proper stateful firewall, protect admin access with port knocking, deploy a honeypot to observe real attack traffic, and segment the network with VLANs.

---

## 1. Stateful Firewall (MikroTik)

A stateful firewall tracks the *state* of each connection — distinguishing between traffic belonging to an established session vs unsolicited inbound packets. Far superior to simple packet filtering.

```bash
# Allow established and related connections
/ip firewall filter add chain=input connection-state=established,related action=accept
/ip firewall filter add chain=forward connection-state=established,related action=accept

# Drop invalid packets (malformed / out of state — signs of scanning or attacks)
/ip firewall filter add chain=input connection-state=invalid action=drop
/ip firewall filter add chain=forward connection-state=invalid action=drop

# Allow LAN devices to access the router itself
/ip firewall filter add chain=input src-address=192.168.88.0/24 action=accept

# Allow ICMP (ping) for diagnostics
/ip firewall filter add chain=input protocol=icmp action=accept

# Drop all unsolicited traffic from WAN (D-Link side)
/ip firewall filter add chain=input in-interface=WAN action=drop

# Critical isolation rule — prevent lab devices reaching primary D-Link network
/ip firewall filter add chain=forward dst-address=192.168.0.0/24 action=drop
```

**Why the last rule matters:** Without this, a compromised device on the lab network could pivot to devices on the primary D-Link network (192.168.0.x), completely defeating the isolation architecture.

---

## 2. Port Knocking (Protect Winbox Admin)

Port knocking requires a secret sequence of port connections before a service becomes accessible. Without the knock sequence, Winbox port 8291 appears completely closed to anyone scanning.

```bash
/ip firewall filter add chain=input protocol=tcp dst-port=1234 \
  action=add-src-to-address-list address-list=knock1 address-list-timeout=10s

/ip firewall filter add chain=input protocol=tcp dst-port=5678 \
  src-address-list=knock1 \
  action=add-src-to-address-list address-list=knock2 address-list-timeout=10s

/ip firewall filter add chain=input protocol=tcp dst-port=9012 \
  src-address-list=knock2 \
  action=add-src-to-address-list address-list=allowed-winbox address-list-timeout=1h

/ip firewall filter add chain=input protocol=tcp dst-port=8291 \
  src-address-list=allowed-winbox action=accept

/ip firewall filter add chain=input protocol=tcp dst-port=8291 action=drop
```

**Knock sequence:** TCP to port `1234` → `5678` → `9012` within 10 seconds each. Winbox then opens for 1 hour.

---

## 3. Cowrie SSH Honeypot

A fake SSH server that logs everything attackers attempt. Runs on Raspberry Pi, mimics a real Linux system to trick automated scanners and attackers into revealing their tactics.

**Installation:**
```bash
sudo adduser --disabled-password cowrie
sudo su - cowrie
git clone https://github.com/cowrie/cowrie
cd cowrie
python3 -m venv cowrie-env
source cowrie-env/bin/activate
pip install -r requirements/requirements.txt
cp etc/cowrie.cfg.dist etc/cowrie.cfg
bin/cowrie start
```

**Logs location:** `~/cowrie/var/log/cowrie/cowrie.log`

**What gets captured:** Every login attempt (username/password combinations), every command typed by the attacker in the fake shell, file download attempts, and attacker IP addresses.

**Key learning:** Within hours of exposure, automated scanners appear trying common credentials like `root/root`, `admin/admin`, `pi/raspberry`. This illustrates why default credentials are dangerous and why obscurity alone is not security.

---

## 4. VLAN Segmentation

Split the lab network into separate logical networks on the same physical hardware. Pentesting tools (Kali Linux) isolated from main lab devices (Pi-hole, Grafana).

```bash
# Create VLAN for pentesting
/interface vlan add name=VLAN-pentest vlan-id=10 interface=bridge-LAN
/ip address add address=192.168.90.1/24 interface=VLAN-pentest

# DHCP for pentest VLAN
/ip pool add name=pentest-pool ranges=192.168.90.10-192.168.90.254
/ip dhcp-server add name=pentest-dhcp interface=VLAN-pentest address-pool=pentest-pool disabled=no
/ip dhcp-server network add address=192.168.90.0/24 gateway=192.168.90.1 dns-server=192.168.88.249

# Block pentest VLAN from reaching main lab
/ip firewall filter add chain=forward \
  src-address=192.168.90.0/24 dst-address=192.168.88.0/24 action=drop
```

**Network map after segmentation:**
```
MikroTik RB450G (192.168.88.1)
 ├── Main Lab VLAN    — 192.168.88.x (Pi-hole, AP, trusted devices)
 └── Pentest VLAN     — 192.168.90.x (Kali Linux, attack tools, vulnerable VMs)
```

---

## Key Takeaways

- Stateful firewalls are vastly more effective than simple port blocking
- Port knocking eliminates brute force risk on admin interfaces
- Honeypots reveal real-world attack patterns — automated scanners are constant and aggressive
- VLANs enforce the principle of least privilege at the network level
- The isolation rule blocking `192.168.0.0/24` is the single most important security rule in this setup
