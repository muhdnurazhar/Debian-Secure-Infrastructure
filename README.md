# Simulate Secure Multi-Site Infrastructure

## Introduction & Project Overview
>This project simulates a real-world enterprise Linux-based (Debian 13) which focusses on security, centralization, load-balancing which is merge scenario where to geographically separated for coporate domains: (Headquarters) wsmb2026.my, (Server Farm) itnsa.my to establish reliable communication channels.

## Project Objective

> To architect and deploy a production-ready, multi-site enterprise infrastructure that securely integrates two distinct corporate domains using simulated internet using Public ISP. 

1. Utilizing critical thinking to solve core infrastructure challenges.

2. Enforcing a Zero-Trust security perimeter.

3. Mitigating single points of failure which have redundacy for multiple services also achieve rapid scaling for enterprises.

## Technical Architecture & Implementation

<img width="921" height="799" alt="Screenshot 2026-06-13 124132" src="https://github.com/user-attachments/assets/db8c91d8-e542-414d-bea7-10870ab21cec" />

> This topology outlines resilent and enterprise production multi-site using 3 multiple sites
> ### Headquarters: ** Act as the main corporate network hosting core identity services and internal user endpoint using Active Directory concepts. 
> 1. Network and IP Management
> 2. firewall?
> 3. Central User Login
> 4. File Sharing

> ### Server Farm/ DMZ
> 1. Smart Web Traffic to secure the network from redirecting all the way to the backend server
> 2. DNS servers that automatically sync with primary and secondary DNS to mitigate any outage happen
> 3. Setup and securing enterprise email using postfix dovecot package using SSL/TLS form digital certificates
> 4. Ansible was used automated code to setup and configure servers instantly while also reduced human errors.

> ### Public ISP Transit: Act as untrusted public internet connecting all corporate sites 
> 1. All the edge routers were using OSPF routing protocol only for public subnets to route public traffic across the network.
> 2. Implementing GRE over IPsec both edge routers using certifcate-based and placed on trusted root certicate authorities on each servers.
> 3. Also block all untrusted incoming internet traffic by default and only explicitly allowed internal traffic 

