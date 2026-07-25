# NETSEC: Secure Multi-Site Network in Cisco Packet Tracer

I built this Cisco Packet Tracer project as one connected secure network, not as separate small labs for individual security features. My goal was to make the topology behave like a real organization network: multiple routed sites, protected management access, centralized authentication, logging, time synchronization, a DMZ for public services, and a VPN path between remote networks.

The project file is the main artifact in this repository. The PDF is included as a quick visual reference for the topology.

## Repository Contents

- `Project NETSEC.pkt` - Cisco Packet Tracer project file
- `Project NETSEC.pdf` - exported topology view for quick review
- `README.md` - project overview and design notes

## What I Built

The topology contains 43 devices:

- 9 routers
- 11 switches
- 7 servers
- 16 PCs

I separated the network into routed subnets so each area had its own role instead of placing all hosts in one flat LAN. Some examples from the project:

| Area | Subnet / Device |
| --- | --- |
| Remote branch LAN | `192.168.0.0/24` |
| Restricted user LAN | `192.168.1.0/24` |
| Internal private network | `192.168.2.0/24` |
| DMZ | `192.168.3.0/24` |
| VLAN 10 / VLAN 20 segments | `192.168.4.0/25` and `192.168.4.128/25` |
| Monitoring services | `192.168.5.0/24` |
| AAA services | `192.168.6.0/24` |
| Remote web/server network | `192.168.7.0/24` |
| Additional routed site | `192.168.8.0/24` |

I used OSPF as the dynamic routing protocol between the routers, with multiple `/30` point-to-point serial links from the `10.0.0.0` range. I also made the LAN-facing interfaces passive so routing updates stay on the router-to-router links instead of being sent toward end hosts.

## Security Features

### Centralized Device Login

I configured AAA on the routers using both RADIUS and TACACS+ servers:

- TACACS+ server: `192.168.6.101`
- RADIUS server: `192.168.6.102`

Routers use centralized authentication first and then fall back to local login. I added this because I did not want the project to stop at local passwords on each device. I wanted management access to be closer to how an organization would control router administration.

### Secure Management Plane

For router management and selected hardened switches, I configured:

- SSH version 2
- local admin user for fallback access
- login authentication through AAA
- console and VTY session timeouts
- MOTD warning banners
- syslog forwarding to `192.168.5.101`
- NTP synchronization with `192.168.5.100`

This makes the devices easier to audit and manage. The logs and time source also make troubleshooting more realistic, because events are not isolated on each router.

### OSPF Authentication

The routed backbone uses OSPF with MD5 authentication on the serial interfaces. I added this because routing security is easy to forget in labs. If someone can form a fake routing adjacency, the rest of the security design can be weakened, so I wanted the routing layer to have protection too.

### Zone-Based Firewall and DMZ

Router4 acts as the firewall point for three zones:

- `PRIVATE_ZONE` on `192.168.2.0/24`
- `DMZ` on `192.168.3.0/24`
- `PUBLIC_ZONE` on the routed WAN side

The firewall policy allows specific flows between zones. For example, public traffic is allowed to reach the DMZ web service only on HTTP, while private hosts can reach the DMZ and outside networks. The DMZ web server is `192.168.3.101`.

### Site-to-Site IPsec VPN

I configured an IPsec VPN between Router2 and Router8:

- Router2 LAN: `192.168.0.0/24`
- Router8 LAN: `192.168.7.0/24`
- Router2 peer: `10.0.0.1`
- Router8 peer: `10.0.0.30`

The tunnel uses ISAKMP, AES encryption, pre-shared key authentication, PFS group 5, and an ACL to define interesting traffic between the two LANs. This part of the project was my way of connecting two separate sites securely instead of just relying on normal routing.

### Access Control Lists

I used ACLs to control traffic in different parts of the network. One example is Router6, where I treated hosts differently depending on their role instead of giving the whole LAN the same access:

- SSH traffic from `192.168.1.100`
- HTTP access from `192.168.1.101` to selected web servers
- ICMP echo from `192.168.1.102`
- Syslog traffic from `192.168.1.103`
- ICMP echo replies

This was useful practice because the ACL was not just "permit everything"; it was based on what each host was supposed to do.

### Layer 2 Hardening

On the hardened access switches, I configured protections such as:

- port security
- DHCP snooping
- Dynamic ARP Inspection
- BPDU Guard
- PortFast on access ports
- native VLAN changed to `999`
- DTP disabled using `switchport nonegotiate`
- unused ports shut down

I included these controls because many simple Packet Tracer projects treat switches as passive cabling. I wanted the access layer to be part of the security design, not just a way to connect PCs.

## Main Services

| Service | Address |
| --- | --- |
| NTP server | `192.168.5.100` |
| Syslog server | `192.168.5.101` |
| TACACS+ server | `192.168.6.101` |
| RADIUS server | `192.168.6.102` |
| DMZ web server | `192.168.3.101` |
| Remote web server | `192.168.7.101` |
| DHCP server | `192.168.4.240` |

## What This Project Demonstrates

By building this project, I practiced connecting security controls across different layers instead of configuring them one by one without context:

- routing with OSPF and authenticated adjacencies
- centralized administrative access with RADIUS and TACACS+
- SSH-only remote management
- syslog and NTP infrastructure
- ACL-based traffic control
- zone-based firewall policy for private, public, and DMZ traffic
- site-to-site IPsec VPN
- VLAN segmentation and router-on-a-stick
- access switch hardening against common Layer 2 attacks

The part I like most is that the controls are connected. AAA protects device access. Syslog records what happens, and NTP makes those logs useful. OSPF authentication protects routing, ACLs and zones control traffic, and the VPN protects traffic between separated sites.

## How To Open

1. Install Cisco Packet Tracer.
2. Open the `.pkt` file from this repository.
3. Review the topology from the logical workspace.
4. Check the router and switch CLI configurations for the security controls listed above.
5. Use Simulation Mode to test allowed and blocked traffic paths.

## Notes

This is a lab project, so some secrets and keys are visible in the Packet Tracer configuration. In a real environment, those values should be rotated, stored securely, and never committed in plain text.
