# Network Testing, Verification and Troubleshooting

## 1. Overview

This document consolidates the connectivity tests, verification procedures and troubleshooting findings from the Enterprise Network Security Lab implemented using Cisco Packet Tracer.

Testing focused on validating VLAN connectivity, routing, HSRP, internal applications, DNS, firewall connectivity, ISP connectivity and network security policies.

The results distinguish between successfully tested functionality, configured features and issues that remain unresolved.

## 2. VLAN and Network Connectivity

The enterprise network uses departmental VLANs to separate users and services.

| VLAN | Department / Function | Network |
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

### Verification commands

On the core and access switches:

```cisco
show vlan brief
show interfaces trunk
show ip interface brief
```

These commands help verify VLAN creation, trunk configuration and Layer 3 interface status.

## 3. HSRP Verification

HSRP provides virtual default gateways for the departmental VLANs.

The observed configuration uses:

- CORE-SW1 as the active gateway, priority 110.
- CORE-SW2 as the standby gateway, priority 100.
- Virtual gateway addresses ending in `.1`.

### Verification commands

```cisco
show standby brief
show standby
```

### Observed result

The HSRP status previously confirmed:

| Device | VLAN 10 role | Priority | IP address |
|---|---|---|---|
| CORE-SW1 | Active | 110 | 10.10.10.2 |
| CORE-SW2 | Standby | 100 | 10.10.10.3 |
| Virtual gateway | Shared | — | 10.10.10.1 |

The active and standby roles were observed successfully. A complete gateway failover test across all VLANs and applications has not been documented.

## 4. Internal Application and DNS Testing

The DMZ contains the internal CRM and DNS services.

| Service | IP address | Purpose |
|---|---|---|
| CRM server | 192.168.4.10 | Internal CRM application |
| DNS server | 192.168.4.11 | Internal name resolution |
| Web/mail test server | 192.168.4.12 | Internal web and mail testing |

### Agent workstation

The example Agent PC uses:

| Setting | Value |
|---|---|
| IP address | 10.10.40.10 |
| Subnet mask | 255.255.255.0 |
| Default gateway | 10.10.40.1 |
| DNS server | 192.168.4.11 |

### Test 1: CRM access

The Agent PC successfully accessed the internal CRM application.

### Test 2: DNS resolution

The following command was executed:

```text
nslookup crm.company.local
```

**Observed result:** The hostname resolved to `192.168.4.10` using DNS server `192.168.4.11`.

### Test 3: Internal web application

The Agent PC successfully accessed the internal web server at `192.168.4.12`.

## 5. Network Security Testing

Agent VLAN 40 is intended to provide access to approved business applications while restricting access to other internal resources.

### Observed results

| Test | Expected behaviour | Observed result |
|---|---|---|
| Agent access to CRM | Allow | Successful |
| Agent DNS lookup | Allow | Successful |
| Agent access to internal web server | Subject to policy | Successful |
| Agent ping to IT Admin `10.10.100.2` | Restrict | Successful |

The successful ping to IT Admin indicates that Agent VLAN isolation was not fully effective during testing.

ACL 141 and ACL 150 were found to contain broad permit statements that weaken the intended restrictions.

The ACL configurations and known security gaps are documented in:

`NETWORK-SECURITY-AND-ACCESS-CONTROL.md`

## 6. Firewall and WAN Testing

The network uses two ASA firewalls and two ISP routers.

### WAN addressing

| Device | Outside IP | Gateway |
|---|---|---|
| FW-PRIMARY | 203.0.113.2/30 | 203.0.113.1 |
| FW-SECONDARY | 198.51.100.2/30 | 198.51.100.1 |

### ISP1 and simulated internet

The simulated public web server uses:

- IP address: `8.8.8.8`
- Default gateway: `8.8.8.1`
- HTTP and HTTPS: Enabled.

The following tests were recorded:

| Source | Destination | Result |
|---|---|---|
| ISP1 | 8.8.8.8 | Successful |
| INTERNET-R1 | 8.8.8.8 | Successful |
| FW-PRIMARY | 203.0.113.1 | Failed |
| FW-PRIMARY | 8.8.8.8 | Failed |
| ISP2 | 8.8.8.8 | Not verified |

The primary firewall's outside interface was observed as up/up, but its counters showed no traffic during the failed tests. Its connectivity to ISP1 requires further troubleshooting.

ISP2 has a firewall-facing connection, but its onward internet path has not been implemented or verified.

## 7. Routing Verification

The core switches use static default routes with higher-distance alternatives.

### CORE-SW1

```cisco
show ip route
show ip interface brief
```

Preferred next hop: `10.10.254.1`

Alternative next hop: `10.10.254.5`, administrative distance 200.

### CORE-SW2

Preferred next hop: `10.10.254.5`

Alternative next hop: `10.10.254.1`, administrative distance 200.

These routes provide an alternative next hop if the preferred route is removed from the routing table. Automatic upstream failure detection has not been demonstrated.

## 8. Troubleshooting Commands

The following commands were used or identified as useful for diagnosing network issues.

### Switches

```cisco
show vlan brief
show interfaces trunk
show ip interface brief
show standby brief
show ip route
show access-lists
show running-config
```

### Routers

```cisco
show ip interface brief
show ip route
show ip arp
show running-config
ping 8.8.8.8
traceroute 8.8.8.8
```

### Cisco ASA firewalls

```cisco
show interface ip brief
show route
show arp
show access-list
show nat
show xlate
show running-config
ping 203.0.113.1
```

Command availability may vary depending on the simulated device and Packet Tracer version.

### End-user workstations

```text
ipconfig /all
ping 10.10.40.1
ping 192.168.4.10
nslookup crm.company.local
tracert 8.8.8.8
```

## 9. Known Issues and Outstanding Work

The following items remain outstanding:

1. FW-PRIMARY connectivity to ISP1 and the simulated public server requires troubleshooting.
2. ISP2 requires an onward connection and routing configuration before end-to-end internet connectivity can be tested.
3. Agent VLAN ACLs require correction to enforce the intended least-privilege policy.
4. SSH management ACL enforcement requires verification.
5. HSRP failover needs a documented end-to-end test.
6. Floating static route recovery needs testing under an upstream failure scenario.
7. Firewall high availability has not been demonstrated.
8. QoS performance under congestion has not been measured.

## 10. Overall Implementation Assessment

| Network component | Assessment |
|---|---|
| VLAN segmentation | Configured |
| HSRP | Active/standby observed |
| Core switch trunking | Configured |
| Static routing | Configured |
| Internal CRM access | Tested successfully |
| Internal DNS | Tested successfully |
| DMZ services | Configured and accessed |
| Agent VLAN restrictions | Partially implemented; isolation gap identified |
| ASA firewall policies | Configured |
| ISP1 simulated internet | Tested successfully from ISP1 and INTERNET-R1 |
| ISP2 internet connectivity | Not verified |
| QoS | Configured; performance not tested |
| Complete network failover | Not verified |

## 11. Conclusion

The Enterprise Network Security Lab demonstrates an enterprise network architecture incorporating departmental VLANs, redundant core switching, HSRP, ASA firewalls, dual ISP-facing connections, internal applications, DNS, ACLs and QoS.

Testing confirmed internal CRM access, DNS resolution and connectivity between ISP1, INTERNET-R1 and the simulated public web server.

The tests also identified areas requiring further improvement, particularly Agent VLAN isolation, primary firewall WAN connectivity, ISP2 internet connectivity and complete failover verification.

The documented results provide a transparent record of the implemented design, tested functionality and outstanding technical work.
