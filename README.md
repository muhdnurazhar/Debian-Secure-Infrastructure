# Simulate Secure Multi-Site Infrastructure

## Introduction & Project Overview
> This project simulates a real-world enterprise Linux-based (Debian 13) which focusses on security, centralization, load-balancing which is merge scenario where to geographically separated for coporate domains: (Headquarters) wsmb2026.my, (Server Farm) itnsa.my to establish reliable communication channels.

## Project Objectives

To architect and validate a production enterprise network connectivity, multi site Linux enterprise environment across the `wsmb2026.my` and `itnsa.my` domains by achieving the following milestones:

1. **Secure Network Infrastructure & Core Addressing Automation**
   To establish a secure, routed network backbone using dynamic routing (OSPF), encrypted site-to-site communication (GRE over IPsec VPN), perimetric firewalls (NFTables), and automated client addressing with real-time record updates (DHCP with Dynamic DNS).

2. **Centralized Identity Access Control & Trusted Data Governance**
   To deploy a unified user directory (OpenLDAP) and a custom Certificate Authority (CA) to enforce role-based network file sharing (Samba) and secure transport-layer corporate messaging (SMTPS/IMAPS Email) across the enterprise perimeters.

3. **High-Availability Application Delivery & Infrastructure as Code (IaC)**
   To engineer a resilient server environment utilizing reverse proxy load-balancing with TLS termination (HAProxy), automated multi-node web provisioning (Apache), fault-tolerant primary/secondary zone replication (BIND9 DNS Master-Slave), and zero-touch deployment pipelines (Ansible Automation).

## Technical Architecture & Implementation

<img width="921" height="799" alt="Simulated Infrastructure Topology" src="https://github.com/user-attachments/assets/db8c91d8-e542-414d-bea7-10870ab21cec" />

This multi-site network uses Debian 13 to run a resilient enterprise production infrastructure. All services are integrated together to handle security, automation, and high availability.

### 1. Headquarters Perimeter (`wsmb2026.my`)
Focuses on internal LAN operations, user identity management, and secure data storage:
* **ISC-DHCP & BIND9 (DDNS):** Automatically allocates client IPs and updates DNS records instantly.
* **OpenLDAP:** Acts as the single central server to manage and authenticate all user logins.
* **Samba File Server:** Secures corporate data by separating public read-only folders from private internal data.

### 2. Server Farm & DMZ Zone (`itnsa.my`)
Hosts core business applications and handles internal and external traffic demands:
* **HAProxy reverse proxy:** Enforces HTTP-to-HTTPS redirect and load-balances web traffic using Round-Robin.
* **Apache Web & BIND9 Cluster:** Runs active web endpoints backed by a Master-Slave DNS zone replication to prevent failure.
* **Postfix & Dovecot Email:** Provides secure corporate messaging using SSL/TLS certificates and LDAP integration.
* **Ansible Automation:** Uses automated playbooks for zero-touch configuration to set up secondary servers instantly.

### 3. Public ISP Transit Backbone (`internet.com`)
Simulates the public internet environment to connect all corporate sites together securely:
* **FRRouting (OSPF):** Runs dynamic routing across edge boundaries strictly for public subnets.
* **GRE over IPsec VPN:** Builds secure, encrypted site-to-site tunnels authenticated by a custom Certificate Authority (CA).
* **NFTables Firewall:** Blocks untrusted incoming public traffic by default while allowing internal hosts to use PAT.

