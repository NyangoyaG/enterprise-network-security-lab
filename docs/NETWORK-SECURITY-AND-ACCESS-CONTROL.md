# Network Security and Access Control

## 1. Overview

This document describes the network security controls implemented in the Enterprise Network Security Lab using Cisco Packet Tracer.

The security design uses VLAN segmentation, Access Control Lists (ACLs), firewall policies and restricted management access to control communication between departments, agents, internal servers and the DMZ.

The objective is to limit unauthorized access while allowing users to reach the applications and services required for their work.

## 2. VLAN Segmentation

The enterprise network uses separate VLANs to logically segment departments and services.

| VLAN | Name / Department | Subnet |
|---|---|---|
| 10 | BUSINESS_A | 10.10.10.0/24 |
| 20 | BUSINESS_B | 10.10.20.0/24 |
| 30 | BUSINESS_C | 10.10.30.0/24 |
| 40 | AGENTS | 10.10.40.0/24 |
| 50 | BUSINESS_E | 10.10.50.0/24 |
| 60 | BUSINESS_F | 10.10.60.0/24 |
| 70 | VOICE_QOS | Reserved for voice traffic |
| 80 | WFM_QA | 10.10.80.0/24 |
| 90 | TEAM_MANAGERS | 10.10.90.0/24 |
| 100 | IT_ADMIN | 10.10.100.0/24 |
| 110 | FINANCE_HR | 10.10.110.0/24 |
| 120 | DMZ | 192.168.4.0/25 |
| 130 | INTERNAL_SERVERS | 10.10.130.0/24 |
| 140 | NETWORK_MGMT | 10.10.140.0/24 |

VLAN segmentation separates network traffic by department and function. Inter-VLAN communication is controlled through Layer 3 routing and security policies.

## 3. Agent VLAN Security

Agent workstations are assigned to VLAN 40.

**Agent network details**

| Setting | Value |
|---|---|
| VLAN | 40 |
| Network | 10.10.40.0/24 |
| Default gateway | 10.10.40.1 |
| Example agent PC | 10.10.40.10 |
| CRM application | 192.168.4.10 |
| DNS server | 192.168.4.11 |

The intended security policy is to allow agents to access the company's CRM and DNS services while restricting unnecessary access to other internal networks.

### Intended access policy

| Destination | Intended access |
|---|---|
| CRM – 192.168.4.10 | HTTP and HTTPS permitted |
| DNS – 192.168.4.11 | DNS permitted |
| Other corporate subnets | Restricted |
| Other DMZ resources | Restricted |
| External destinations | Subject to firewall and network policies |

## 4. Access Control Lists

ACLs are used to filter traffic based on source addresses, destination addresses and protocols.

### 4.1 SSH-MANAGEMENT

This standard ACL restricts SSH management access to the designated IT management workstation.

```cisco
access-list SSH-MANAGEMENT permit host 10.10.100.10
access-list SSH-MANAGEMENT deny any
```

Only the workstation at `10.10.100.10` is explicitly permitted by this ACL.

### 4.2 AGENT-RESTRICTION

This ACL contains rules intended to allow agents to access CRM and DNS services before denying access to other internal networks.

```cisco
10 permit tcp any host 192.168.4.10 eq www
20 permit tcp any host 192.168.4.10 eq 443
30 permit udp any host 192.168.4.11 eq domain
40 permit tcp any host 192.168.4.11 eq domain
50 deny ip any 10.10.0.0 0.0.255.255
60 deny ip any 192.168.4.0 0.0.0.127
70 permit ip any any
```

### 4.3 DMZ-AGENT-FILTER

This ACL defines permitted CRM and DNS traffic from VLAN 10 before denying other access from that subnet to the DMZ.

```cisco
10 permit tcp 10.10.10.0 0.0.0.255 host 192.168.4.10 eq www
20 permit tcp 10.10.10.0 0.0.0.255 host 192.168.4.10 eq 443
30 permit udp 10.10.10.0 0.0.0.255 host 192.168.4.11 eq domain
40 permit tcp 10.10.10.0 0.0.0.255 host 192.168.4.11 eq domain
50 deny ip 10.10.10.0 0.0.0.255 192.168.4.0 0.0.0.127
60 permit ip any any
```

### 4.4 ACL 141

ACL 141 contains the rules intended to restrict Agent VLAN 40 traffic.

```cisco
10 permit ip 10.10.40.0 0.0.0.255 host 10.10.40.1
20 permit tcp 10.10.40.0 0.0.0.255 host 192.168.4.10 eq www
30 permit tcp 10.10.40.0 0.0.0.255 host 192.168.4.10 eq 443
40 permit udp 10.10.40.0 0.0.0.255 host 192.168.4.11 eq domain
50 deny ip 10.10.40.0 0.0.0.255 10.10.0.0 0.0.255.255
60 deny ip 10.10.40.0 0.0.0.255 192.168.4.0 0.0.0.127
70 permit ip 10.10.40.0 0.0.0.255 any
80 permit icmp 10.10.40.0 0.0.0.255 host 192.168.4.11
90 permit icmp 10.10.40.0 0.0.0.255 host 192.168.4.10
```

**Important configuration observation:** ACL entries are processed from top to bottom. Rule 70 permits all remaining IP traffic from VLAN 40, making rules 80 and 90 unreachable. The broad permit also means that the ACL does not enforce the intended restriction on all remaining destinations.

### 4.5 ACL 150

ACL 150 contains a similar set of Agent VLAN 40 restrictions.

```cisco
10 permit tcp 10.10.40.0 0.0.0.255 host 192.168.4.10 eq www
20 permit tcp 10.10.40.0 0.0.0.255 host 192.168.4.10 eq 443
30 permit udp 10.10.40.0 0.0.0.255 host 192.168.4.11 eq domain
40 deny ip 10.10.40.0 0.0.0.255 10.10.0.0 0.0.255.255
50 deny ip 10.10.40.0 0.0.0.255 192.168.4.0 0.0.0.127
60 permit ip 10.10.40.0 0.0.0.255 any
```

Rule 60 permits all remaining IP traffic from VLAN 40 to other destinations. Consequently, ACL 150 also requires further review if the intention is to restrict agents to CRM and DNS only.

## 5. Firewall Security

The network uses two Cisco ASA firewalls:

- FW-PRIMARY connects to ISP1.
- FW-SECONDARY connects to ISP2.

The firewall configurations include:

- Inside and outside security zones.
- Network Address Translation (NAT) and Port Address Translation (PAT).
- Default routes towards the respective ISP gateways.
- Outside access policies restricting unsolicited access to internal networks and the DMZ.

The firewall configurations are documented separately in `FIREWALL-AND-WAN-CONFIGURATIONS.md`.

## 6. Security Verification Results

The following tests were previously performed during the lab implementation.

| Test | Result | Observation |
|---|---|---|
| Agent PC access to CRM | Successful | Internal CRM application accessible |
| Agent DNS lookup | Successful | `crm.company.local` resolved to `192.168.4.10` |
| Agent access to IT Admin address | Successful ping | ICMP traffic reached `10.10.100.2` |
| Agent isolation from all internal networks | Not fully verified | Successful IT Admin ping indicates an access-control gap |
| SSH management restriction | Configured | Enforcement not independently verified |
| DMZ access restrictions | Configured | Full enforcement not independently verified |

## 7. Known Security Gaps

The following issues were identified during implementation:

1. ACL 141 and ACL 150 contain broad permit rules that weaken the intended Agent VLAN restrictions.
2. ACL 141 includes ICMP permit entries after a broad permit statement, making those entries ineffective.
3. Agent-to-IT Admin ping was successful, indicating that agent isolation requires further review.
4. The presence of an ACL in the configuration does not establish that it is correctly applied to the intended interface or direction.
5. SSH management access has been configured in an ACL, but its effective enforcement still requires verification.

## 8. Recommended Improvements

The following improvements are recommended before considering the network security policy complete:

- Review and correct ACL rule ordering.
- Remove broad permit statements where they conflict with the intended security policy.
- Permit only required CRM and DNS traffic from Agent VLAN 40.
- Apply ACLs to the correct interfaces and directions.
- Verify that agents cannot access IT Admin, Finance, HR and other restricted networks.
- Test both permitted and denied traffic from representative workstations.
- Confirm SSH access is limited to the authorized management workstation.
- Record ACL hit counters and test results as implementation evidence.

## 9. Conclusion

The lab implements several enterprise network security mechanisms, including VLAN segmentation, ACL-based traffic filtering, SSH management restrictions and ASA firewall policies.

The configurations demonstrate the intended security design, but testing identified gaps in Agent VLAN isolation. Further ACL corrections and verification are required before the intended least-privilege access policy can be considered fully enforced.
