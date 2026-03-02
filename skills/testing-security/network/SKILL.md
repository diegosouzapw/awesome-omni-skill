---
name: network
description: Network penetration testing skills for service exploitation and protocol attacks.
metadata:
  version: "1.0"
  category: network
---

# Network Security Skills

Comprehensive network penetration testing skills organized by service category.

## Categories

| Category | Description | Skills |
|----------|-------------|--------|
| [Reconnaissance](reconnaissance/SKILL.md) | Network discovery | 5 |
| [Layer 2 Attacks](layer2-attacks/SKILL.md) | MITM, credential capture | 5 |
| [Auth Services](auth-services/SKILL.md) | SSH, RDP, Kerberos | 8 |
| [File Services](file-services/SKILL.md) | SMB, NFS, FTP | 7 |
| [Databases](databases/SKILL.md) | SQL, NoSQL, cache | 11 |
| [Email](email/SKILL.md) | SMTP, POP3, IMAP | 3 |
| [Infrastructure](infrastructure/SKILL.md) | DNS, SNMP, RPC | 6 |
| [Containers](containers/SKILL.md) | Docker, Kubernetes | 3 |
| [Message Queues](message-queues/SKILL.md) | MQTT, RabbitMQ, Kafka | 3 |
| [Industrial IoT](industrial-iot/SKILL.md) | ICS/SCADA protocols | 4 |
| [VPN/Tunneling](vpn-tunneling/SKILL.md) | IPSec, pivoting | 2 |
| [Security Analysis](security-analysis/SKILL.md) | SSL/TLS, evasion | 3 |
| [Wireless](wireless/SKILL.md) | WiFi attacks | 1 |
| [Other Services](other-services/SKILL.md) | Java RMI, X11, VoIP | 5 |

## Quick Reference

| Service | Port | Quick Check |
|---------|------|-------------|
| SMB | 445 | `smbclient -L` |
| SSH | 22 | `ssh -v` |
| MySQL | 3306 | `mysql -u root` |
| Redis | 6379 | `redis-cli info` |
| Docker | 2375 | `docker -H` |
