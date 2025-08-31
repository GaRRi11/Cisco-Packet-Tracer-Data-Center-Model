# Cisco-Packet-Tracer-Data-Center-Model

## Device Roles and Justification

### Edge & Security Layer

Edge Router (Cisco ISR): Manages external routes, dual-homed to ISPs. Provides WAN intelligence not possible on a switch.

SW-OUTSIDE-1: A simple Catalyst L2 switch that isolates the edge router and distributes links to ASA1/ASA2.

ASA1/ASA2: Stateful firewalls in active/standby failover. Perform NAT, VPN, and DMZ isolation. ASA chosen for firewall depth rather than routing scale.

### Core & Distribution Layer

Core1/Core2: High-availability pair providing L3 services, HSRP, and OSPF. They are the traffic directors.

DSW1/DSW2: Policy enforcers at the aggregation layer. Route between VLANs, run ACLs, and balance uplinks from access switches.

### ASW_DB (Database Zone)

DB1 and DB2: They store persistent data for backend applications.

only backend servers can reach them.

### ASW_Backend (Application Zone)

Internal HAProxy: Load-balances requests between App1 and App2 before they reach databases.

App1 and App2: Application servers that process business logic and talk to the database tier.

The backend is only reachable by DMZ web servers and internal clients.

### ASW_DMZ (Demilitarized Zone)

Web1 and Web2: Public-facing servers hosting website.

Cache: Improves web responsiveness by reducing direct load on backend and DB.

Public HAProxy + VIP: Front-end load-balancer that provides a single entry point for internet users.

The DMZ accepts public requests but can only forward them to backend servers under strict control.

### ASW_Monitoring (Monitoring Zone)

Monitoring Server: Collects syslog, SNMP traps, flow records, and server health data.

Placed in a dedicated zone so monitoring continues even if production segments are compromised.

### ASW_Management (Management Zone)

Jump Server: Bastion host — the only permitted way for admins to reach critical devices.

AAA Server: Central authority for authentication, authorization, and accounting. Controls who logs in and what they can do.

Server Discovery: Manages asset inventory and server onboarding.

NTP Server: Provides time synchronization across the entire data center.

Infrastructure DevOps Server: Automates configuration and provisioning tasks.

DNS/DHCP: Provides name resolution and dynamic IP allocation.

### ASW_Client (Client Zone)

Client PC: Represents internal user workstations. They access apps through proper paths, never directly hitting sensitive tiers.

## Traffic Flows

Example: Internet User Accessing a Web Server in DMZ

A user sends an HTTP request to the data center’s public IP.

The Edge Router receives the packet and forwards it to ASA1.

ASA1 applies NAT and ACL rules — only allowing TCP/80 and TCP/443 into the DMZ.

ASA1 forwards the packet to Core1/Core2 → DSW1/DSW2 → ASW_DMZ.

Public HAProxy receives the packet and forwards it to Web1 or Web2.

If needed, Web1 queries Cache or forwards to Internal HAProxy in the Backend zone.

Internal HAProxy may query DB1/DB2 for persistent data.

Response travels back the exact path, with ASA performing stateful inspection to ensure legitimacy.

## Security Architecture

ASA Firewalls: First barrier; perform NAT, VPN, and inspection of inbound/outbound traffic.

DMZ Isolation: Public servers are fenced into their own VLAN, with only the bare minimum allowed into backend tiers.

AAA Enforcement: All admin logins controlled by AAA, with full auditing.

Jump Server: A single chokepoint for administrative access — prevents direct device login from arbitrary clients.

ACLs and VLAN Segmentation: Even inside the core, no VLAN can talk to another without explicit permission.

Out-of-Band Management: Admin traffic kept on its own VLAN, ensuring troubleshooting access during outages.

Monitoring & Logging: Centralized monitoring server collects data from every device, enabling quick detection of anomalies.

Least Privilege Everywhere: Every device, from DB to cache, only communicates with what it needs. Nothing more.
