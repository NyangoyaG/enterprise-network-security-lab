# Enterprise Network Security Lab – Network Architecture

## 1. Project Overview

This project simulates an enterprise network connecting a Head Office (HQ) and a Branch Office using Cisco Packet Tracer.

The branch network is designed to support approximately 1,500 agents and corporate departments while providing network segmentation, controlled access to internal resources, redundancy and security.

## 2. Network Architecture

The network consists of:

- Head Office and Branch Office environments.
- Two core switches configured for redundancy using HSRP.
- Eight access switches connecting departmental users.
- Two ASA firewalls representing primary and secondary security gateways.
- Two ISP connections for WAN connectivity.
- A DMZ hosting internal business applications.
- Internal infrastructure servers.
- Dedicated VLANs for departments and network management.

## 3. Network Segmentation

The network uses VLANs to separate departments and services.

| VLAN | Department / Purpose | Subnet |
|---|---|---|
| 10 | BUSINESS_A | 10.10.10.0/24 |
| 20 | BUSINESS_B | 10.10.20.0/24 |
| 30 | BUSINESS_C | 10.10.30.0/24 |
| 40 | AGENTS | 10.10.40.0/24 |
| 50 | BUSINESS_E | 10.10.50.0/24 |
| 60 | BUSINESS_F | 10.10.60.0/24 |
| 70 | VOICE_QOS | Reserved |
| 80 | WFM_QA | 10.10.80.0/24 |
| 90 | TEAM_MANAGERS | 10.10.90.0/24 |
| 100 | IT_ADMIN | 10.10.100.0/24 |
| 110 | FINANCE_HR | 10.10.110.0/24 |
| 120 | DMZ | 192.168.4.0/25 |
| 130 | INTERNAL_SERVERS | 10.10.130.0/24 |
| 140 | NETWORK_MGMT | 10.10.140.0/24 |

## 4. Core Network Redundancy

Two core switches provide gateway redundancy using Hot Standby Router Protocol (HSRP).

- CORE-SW1: Active router, priority 110.
- CORE-SW2: Standby router, priority 100.
- Virtual gateway addresses: Typically .1 in each VLAN subnet.

The core switches use trunk links to carry departmental VLAN traffic.

## 5. Firewall and WAN Architecture

Two Cisco ASA firewalls provide separate security gateway paths:

- FW-PRIMARY connects to ISP1.
- FW-SECONDARY connects to ISP2.

Both firewalls implement NAT and outside access-control policies.

The Packet Tracer implementation uses independent ASA devices. Native ASA failover and fully automatic dual-ISP failover have not been verified.

## 6. DMZ and Internal Services

The DMZ contains:

| Service | IP Address |
|---|---|
| CRM Application | 192.168.4.10 |
| DNS Server | 192.168.4.11 |
| Web / Mail Test Server | 192.168.4.12 |

The CRM application is accessible through:

`crm.company.local`

The DNS server resolves the hostname to 192.168.4.10.

## 7. Quality of Service

A QoS policy named VOICE-POLICY was configured to prioritize traffic marked with DSCP EF.

The policy uses a priority queue and is applied to selected core-to-access uplinks.

## 8. Security Considerations

The project demonstrates:

- Departmental VLAN segmentation.
- Firewall-based traffic filtering.
- Network Address Translation.
- DMZ separation.
- Access-control lists.
- Dedicated IT administration VLAN.

Agent access restrictions remain under validation. Full isolation from other internal networks has not yet been confirmed.

## 9. Testing and Verification

Tests performed include:

- HSRP active and standby verification.
- VLAN and trunk verification.
- Inter-VLAN connectivity testing.
- DNS resolution.
- CRM application access.
- Internal web server access.
- Firewall and ISP connectivity checks.
- QoS configuration verification.

Some WAN and failover scenarios remain incomplete and are documented as limitations.

## 10. Project Files

The main Packet Tracer simulation is:

`Corporate_Network_HQ_Branch.pkt`

This file contains the enterprise network topology and device configurations.
