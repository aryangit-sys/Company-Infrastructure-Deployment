# Azure Mini Company Infrastructure with Centralized Logging (Wazuh SIEM)

## 📌 Project Overview

This project simulates a **small enterprise cloud infrastructure** deployed on **Microsoft Azure (Student Account)**.  
The environment is intentionally **insecure by design** to enable future security testing, attacks, and SOC analysis.

The focus of this phase is:
- Infrastructure deployment
- Network segmentation (Internal + DMZ)
- Basic system and application logging
- Centralized log collection using **Wazuh SIEM**

⚠️ **No hardening is applied intentionally**.

---

## 🏗️ Architecture Overview

### Virtual Machines (3 Total)

| VM Name | Role | Subnet |
|------|-----|-------|
| Internal-Server | FreeIPA (LDAP + Kerberos), File Server | Internal |
| Web-Server | Apache Web Server (DMZ) | DMZ |
| SIEM-Server | Wazuh SIEM + Analyst Workstation | Internal |

---

## 🌐 Network Design

### Virtual Network
- Address space: `10.0.0.0/16`

### Subnets
- **Internal-Subnet**: `10.0.1.0/24`
- **DMZ-Subnet**: `10.0.2.0/24`

---

## 🔐 Network Security Groups (NSGs)

### NSG-Internal (attached to Internal-Subnet)

**Inbound Rules**

| Priority | Rule | Port | Source |
|-------|-----|-----|------|
| 100 | Allow-SSH | 22 | Any |
| 110 | Allow-Internal-TCP | * | VirtualNetwork |
| 120 | Allow-ICMP | ICMP | Any |
| 130 | Allow-Wazuh-Dashboard | 5601 | Any |

---

### NSG-DMZ (attached to DMZ-Subnet)

**Inbound Rules**

| Priority | Rule | Port | Source |
|-------|-----|-----|------|
| 100 | Allow-HTTP | 80 | Any |
| 110 | Allow-HTTPS | 443 | Any |
| 120 | Allow-SSH | 22 | Any |
| 130 | Allow-All-TCP | * | Any |

---

## 🖥️ VM Configuration (Common)

- OS: **Ubuntu Server 22.04 LTS**
- Authentication: **Username + Password**
- Public IP: Enabled
- Size:
  - Internal & Web: `Standard_B1s`
  - SIEM: **Standard_B2s (2 GB RAM required)**

---

## 🧩 VM 1 — Internal Server (FreeIPA + File Server)

### Install Packages
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install freeipa-server samba auditd rsyslog -y

```




### Set Hostname
sudo hostnamectl set-hostname internal-server.corp.local

### Edit the hosts file:
sudo nano /etc/hosts

### Add the following line:
127.0.0.1 internal-server.corp.local internal-server

### Install and Configure FreeIPA
sudo ipa-server-install

### Use the following values during installation:
Domain: corp.local
Realm: CORP.LOCAL
Integrated DNS: Yes
Kerberos Admin Server: internal-server.corp.local
Configure Samba File Server

### Create shared directory:
sudo mkdir /shared
sudo chmod 777 /shared

### Edit Samba configuration:
sudo nano /etc/samba/smb.conf
#### Append:
[Shared]
path = /shared
read only = no
guest ok = yes

### Restart Samba:
sudo systemctl restart smbd

### Enable Logging Services:
sudo systemctl enable auditd
sudo systemctl start auditd

### Verify logs:
ls /var/log/

### Important logs:
/var/log/auth.log
/var/log/syslog
/var/log/audit/audit.log

#### VM 2 — Web Server (DMZ)
Install Apache and Logging Tools
sudo apt update
sudo apt install apache2 auditd rsyslog -y

#### Deploy Test Web Page
echo "<h1>Welcome to the DMZ Web Server</h1>" | sudo tee /var/www/html/index.html

#### Verify Web Server
curl http://localhost

#### Logs Generated
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/auth.log
/var/log/audit/audit.log

#### VM 3 — SIEM Server (Wazuh)
System Requirements
VM Size: Standard_B2s
Minimum RAM: 2 GB
Install Wazuh (All-in-One)
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a -o

Verify Wazuh Services
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard

Verify Dashboard Port
sudo ss -tulnp | grep 5601

### Expected output:
LISTEN 0 511 0.0.0.0:5601
Local Dashboard Test
curl -k https://localhost:5601

### Access Wazuh Dashboard
https://<SIEM_PUBLIC_IP>:5601

Ignore browser certificate warning and log in using the credentials shown after installation.

Wazuh Agent Installation (VM1 & VM2)
Install Agent
curl -sO https://packages.wazuh.com/4.7/wazuh-agent.sh
sudo bash wazuh-agent.sh

Configure Agent
sudo nano /var/ossec/etc/ossec.conf

#### Update server address:
<server>
  <address>10.0.1.X</address>
  <port>1514</port>
</server>

Start and Enable Agent
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```
Logging Summary
Internal Server
Authentication logs
Syslog
FreeIPA logs
Samba logs
Auditd logs
Web Server
Apache access and error logs
Authentication logs
Auditd logs
SIEM Server
Centralized log ingestion
Agent status monitoring
Event visualization
Infrastructure Deployment Phase Completed
The infrastructure and logging setup is complete.
The environment is intentionally left insecure for attack simulation and security hardening in the next phase.
