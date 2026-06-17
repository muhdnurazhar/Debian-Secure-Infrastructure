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

## System Verification & Results

### Verification 1: Secure Network Infrastructure & Core Addressing
* **What & Why:** Verifying that IP allocations automatically sync with internal name resolution, and site to site traffic is fully encrypted across the public transport layer.
* **Chaining:** Confirms that Objective 1 is met by validating active OSPF routing, GRE over IPsec VPN tunnel stability, and active DHCP-DDNS updates.
* **Verification & Expected Logs:**

#### *Check OSPF neighbor status and routing table on HQ-EDGE* ####
```bash
ip route show protocol ospf
sudo vtysh -c "show ip ospf neighbor"

Neighbor ID     Pri State           Up Time         Dead Time Address         Interface                   
210.187.97.126    1 Full/DR         1h19m48s          31.717s 103.17.78.6     ens37:103.17.78.1       
```

#### *Verify GRE over IPsec VPN tunnel interface state* ####
```bash
ip link show tun1
swanctl --list-sas 

root@HQ-EDGE:~# ip link show tun1
7: tun1@NONE: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1476 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/gre 103.17.78.1 peer 203.80.16.1

root@HQ-EDGE:~# swanctl --list-sas
conn: #1, ESTABLISHED, IKEv2, d29b8df9e76c2de9_i* b34e1173cf5c6461_r
  local  'C=MY, CN=HQ-EDGE.wsmb2026.my' @ 103.17.78.1[4500]
  remote 'C=MY, CN=DC-EDGE.itnsa.my' @ 203.80.16.1[4500]
  AES_CBC-128/HMAC_SHA2_256_128/PRF_HMAC_SHA2_256/ECP_256
  established 877s ago, rekeying in 12343s
  child: #2, reqid 1, INSTALLED, TUNNEL, ESP:AES_GCM_16-128
    installed 405s ago, rekeying in 2919s, expires in 3555s
    in  c4e76654,  69764 bytes,  1041 packets,    22s ago
    out cd9240a8,  69764 bytes,  1041 packets,    22s ago
    local  192.168.10.0/24 192.168.200.1/32
    remote 192.168.20.0/24 192.168.200.2/32
```

#### *Verifying isc-dhcp-server and ddns* ####
``` bash
journalctl -u isc-dhcp-server -u namned --no-pager -n 20

Jun 14 21:46:45 HQ-EDGE dhcpd[1675]: DHCPDISCOVER from 00:0c:29:bd:f8:8e via ens33
Jun 14 21:46:46 HQ-EDGE dhcpd[1675]: DHCPOFFER on 192.168.10.158 to 00:0c:29:bd:f8:8e (CLIENT) via ens33
Jun 14 21:46:46 HQ-EDGE dhcpd[1675]: DHCPREQUEST for 192.168.10.158 (192.168.10.254) from 00:0c:29:bd:f8:8e (CLIENT) via ens33
Jun 14 21:46:46 HQ-EDGE dhcpd[1675]: DHCPACK on 192.168.10.158 to 00:0c:29:bd:f8:8e (CLIENT) via ens33
Jun 14 21:46:46 HQ-EDGE dhcpd[1675]: Added new forward map from CLIENT.wsmb2026.my to 192.168.10.158
Jun 14 21:46:46 HQ-EDGE dhcpd[1675]: Added reverse map from 158.10.168.192.in-addr.arpa. to CLIENT.wsmb2026.my
```

#### *Validation of DDNS on Client:* ####
```bash
root@CLIENT:~# nslookup client.wsmb2026.my
Server:         192.168.10.10
Address:        192.168.10.10#53

Name:   CLIENT.wsmb2026.my
Address: 192.168.10.158
```

#### *Nftables script on edge router* ####

> DC-EDGE
```bash
# DC-EDGE nftables
nft add table nat
nft flush table nat

#nat chain
nft add chain ip nat 'prerouting { type nat hook prerouting priority -100; policy accept; }'
nft add chain ip nat 'postrouting { type nat hook postrouting priority 100; policy accept; }'

#nat rules
nft add 'rule ip nat postrouting ip saddr 192.168.20.0/24 ip daddr 192.168.10.0/24 counter accept comment "disable nat on vpn"'
nft add 'rule ip nat postrouting oifname ens37 counter masquerade comment "enable pat connections"'
nft add 'rule ip nat prerouting iif ens37 udp dport 53 dnat to 192.168.20.12 comment "enable dns forwarding"'
nft add 'rule ip nat prerouting iif ens37 tcp dport 25 dnat to 192.168.20.11 comment "enable smtp forwarding"'
```


> HQ-EDGE
```bash
#HQ-EDGE nftables
nft add table nat
nft add table filter 

nft flush table nat 
nft flush table filter

#nat chain
nft add chain ip nat 'prerouting { type nat hook prerouting priority -100; policy accept; }'
nft add chain ip nat 'postrouting { type nat hook postrouting priority 100; policy accept; }'

#filter chain
nft add chain ip filter 'input { type filter hook input priority 0; policy drop; }'
nft add chain ip filter 'output { type filter hook output priority 0; policy accept; }'
nft add chain ip filter 'forward { type filter hook forward priority 0; policy drop; }'

#nat rules
nft add 'rule ip nat postrouting ip daddr 192.168.20.0/24 counter accept comment "disable nat on vpn"'
nft add 'rule ip nat postrouting oifname ens37 counter masquerade comment "enable pat connections"'

#filter rules
nft  add 'rule ip filter input ct state related,established counter accept comment "enable ct state rules"'
nft  add 'rule ip filter output ct state related,established counter accept comment "enable ct state rules"'
nft  add 'rule ip filter forward ct state related,established counter accept comment "enable ct state rules"' 

nft  add 'rule ip filter input ip protocol icmp counter accept comment "enable icmp rules"'
nft  add 'rule ip filter forward ip protocol icmp counter accept comment "enable icmp rules"'

nft  add 'rule ip filter input ip protocol ospf counter accept comment "enable ospf rules"'
nft  add 'rule ip filter forward ip protocol ospf counter accept comment "enable ospf rules"'

nft  add 'rule ip filter input ip protocol gre counter accept comment "enable gre rules"'
nft  add 'rule ip filter forward ip protocol gre counter accept comment "enable gre rules"'

nft  add 'rule ip filter input ip protocol esp counter accept comment "enable esp rules"'
nft  add 'rule ip filter forward ip protocol esp counter accept comment "enable esp rules"'

nft add 'rule ip filter input tcp dport 389 counter accept comment "enable ldap port"'
nft add 'rule ip filter forward tcp dport 389 counter accept comment "enable ldap port"'

nft add 'rule ip filter input tcp dport 22 counter accept comment "enable scp port"'
nft add 'rule ip filter forward tcp dport 22 counter accept comment "enable scp port"'

nft add 'rule ip filter input udp dport 53 counter accept comment "enable scp port"'
nft add 'rule ip filter forward udp dport 53 counter accept comment "enable scp port"'

nft add 'rule ip filter input tcp dport { 500, 4500 } counter accept comment "open vpn port connections" '
nft add 'rule ip filter forward tcp dport { 500, 4500 } counter accept comment "open vpn port connections" '

nft add 'rule ip filter input ip saddr 192.168.10.0/24 counter accept comment "explicitly allowed internals"'
nft add 'rule ip filter forward ip saddr 192.168.10.0/24 counter accept comment "explicitly allowed internals"'
```

### *Open router and execute script each* ###
> HQ-EDGE
```bash
root@HQ-EDGE:~# ./fw.sh 
```
> DC-EDGE
```bash
root@DC-EDGE:~# ./fw.sh 
```

### Verification 2: Centralized Identity & Trusted Data Governance
What & Why: Verifying that network endpoints securely authorize domain users via a central directory and enforce safe, role-based file and email communications.

Chaining: Confirms that Objective 2 is met by validating client-side PAM OpenLDAP login bounds, conditional Samba share restrictions, and TLS-hardened corporate messaging.

### *Query LDAP Validation* ###
```bash
root@CLIENT:~# ldapsearch -x -H ldap://192.168.10.10 -b "cn=users,ou=groups,dc=wsmb2026,dc=my"
# extended LDIF
#
# LDAPv3
# base <cn=users,ou=groups,dc=wsmb2026,dc=my> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# users, groups, wsmb2026.my
dn: cn=users,ou=groups,dc=wsmb2026,dc=my
objectClass: posixGroup
gidNumber: 10000
cn: users
memberUid: samad
memberUid: krishnan
memberUid: jimmy

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 
```

### *Test remote SSH authorization using an LDAP directory user from CLIENT* ###
```bash
root@CLIENT:~# ssh krishnan@client.wsmb2026.my id
krishnan@client.wsmb2026.my's password:
uid=10002(krishnan) gid=10000(groups) groups=10000(groups)
```

### *Test corporate data governance boundaries on Samba data shares* ###
```bash
root@CLIENT:~# smbclient //192.168.10.10/internal -U smbuser%Skills39
Try "help" to get a list of possible commands.
smb: \> mkdir testing from client
smb: \> ls
  .                                   D        0  Sun Jun 14 23:15:06 2026
  ..                                  D        0  Sun Jun 14 23:15:06 2026
  testing                             D        0  Sun Jun 14 23:15:06 2026

                19353424 blocks of size 1024. 16989856 blocks available
```

### *Verify secure, encrypted SMTPS/IMAPS communication with Mail Server using custom CA* ###
```bash
root@CLIENT:~# openssl s_client -connect mail.itnsa.my:465 -brief
Connecting to 192.168.20.11
CONNECTION ESTABLISHED
Protocol version: TLSv1.3
Ciphersuite: TLS_AES_256_GCM_SHA384
Peer certificate: C=MY, CN=*.itnsa.my
Hash used: SHA256
Signature type: rsa_pss_rsae_sha256
Verification: OK
Peer Temp Key: X25519, 253 bits
220 DC-SRV1.itnsa.my ESMTP Postfix (Debian)
```


### Verification 3: High-Availability Delivery & Infrastructure as Code (IaC)
* **What & Why:** Verifying that web traffic is successfully load-balanced, internal endpoints switch roles seamlessly, and server configurations are deployed without errors via automation.
* **Chaining:** Confirms that Objective 3 is met by validating HAProxy Round-Robin algorithm states and clean Ansible playbook execution metrics.*


### *Verify HAProxy Round-Robin distribution across backend pools* ###
```bash
root@itnsa:~# for i in {1..4}; do curl -s https://www.itnsa.my | grep -i "Welcome to website of"; done
<h1><center>Welcome to website of DC-SRV3 </center></h1>
<h1><center>Welcome to website of DC-SRV2 </center></h1>
<h1><center>Welcome to website of DC-SRV3 </center></h1>
<h1><center>Welcome to website of DC-SRV2 </center></h1>
```

### *Verify automated task alignment and status check from Ansible host* ###
```bash
root@DC-SRV1:/opt/ansible# ansible-playbook configure-dns-srv3.yml configure-web-srv3.yml 

PLAY [configure-dns-srv3.yml] **********************************************************************************************************

TASK [Install bind] ********************************************************************************************************************
[WARNING]: Host 'DC-SRV3' is using the discovered Python interpreter at '/usr/bin/python3.13', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.19/reference_appendices/interpreter_discovery.html for more information.
ok: [DC-SRV3]

TASK [Push config secondary bind9 /etc/bind] *******************************************************************************************
ok: [DC-SRV3]

TASK [restart bind9] *******************************************************************************************************************
changed: [DC-SRV3]

PLAY RECAP *****************************************************************************************************************************
DC-SRV3                    : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


PLAY [configure-web-srv3.yml] **********************************************************************************************************

TASK [Install apache] ******************************************************************************************************************
ok: [DC-SRV3]

TASK [create folder] *******************************************************************************************************************
ok: [DC-SRV3] => (item=www)
ok: [DC-SRV3] => (item=intra)

TASK [copy apache config file] *********************************************************************************************************
ok: [DC-SRV3]

TASK [ansible.builtin.shell] ***********************************************************************************************************
changed: [DC-SRV3]

TASK [ansible.builtin.shell] ***********************************************************************************************************
changed: [DC-SRV3]

TASK [ansible.builtin.shell] ***********************************************************************************************************
changed: [DC-SRV3]

TASK [ansible.builtin.shell] ***********************************************************************************************************
changed: [DC-SRV3]

PLAY RECAP *****************************************************************************************************************************
DC-SRV3                    : ok=10   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Conclusion & Key Accomplishments
This project successfully creates a reliable Linux network setup that combines strong security with non-stop system uptime. Through this project, I achieved these core technical milestones:

* **Automated Server Setup:** Used Ansible to configure multiple servers instantly. This removes the need for manual setup and completely avoids human mistakes.
* **Network Defense & Encryption:** Protected the network using custom firewalls (NFTables) and set up highly secure, encrypted VPN tunnels (GRE over IPsec) between different offices.
* **Backup & High Availability:** Kept services running smoothly without down-time by using a traffic loader (HAProxy) and automatic DNS backup replication (BIND9 Master-Slave).
* **Central User & File Control:** Maintained a secure and organized workplace by using a central login system (OpenLDAP) and setting up strict file-sharing permissions (Samba) for different user groups.

