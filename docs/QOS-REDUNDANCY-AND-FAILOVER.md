# QoS, Network Redundancy and Failover

## 1. Overview

This document describes the redundancy and Quality of Service (QoS) mechanisms implemented in the Enterprise Network Security Lab using Cisco Packet Tracer.

The network design incorporates:

- First Hop Redundancy Protocol (HSRP) between the two core switches.
- Redundant Layer 3 paths between the core switches and perimeter firewalls.
- Two ISP connections through separate firewall devices.
- QoS classification and prioritization for voice traffic.

The objective is to improve network availability, reduce the impact of individual device or link failures, and prioritize latency-sensitive traffic.

## 2. Core Switch Redundancy

The network uses two multilayer core switches:

- CORE-SW1
- CORE-SW2

The switches use HSRP to provide virtual default gateways for departmental VLANs.

### HSRP addressing

For the configured departmental VLANs:

| Device | HSRP role | SVI address pattern | Priority |
|---|---|---|---|
| CORE-SW1 | Active | 10.10.x.2 | 110 |
| CORE-SW2 | Standby | 10.10.x.3 | 100 |
| Virtual gateway | Shared gateway | 10.10.x.1 | — |

The virtual gateway remains the default gateway used by connected devices. CORE-SW1 is preferred as the active gateway because it has the higher HSRP priority.

### Example HSRP configuration

The following illustrates the HSRP configuration pattern used for VLAN 10.

**CORE-SW1**

```cisco
interface Vlan10
 ip address 10.10.10.2 255.255.255.0
 standby 10 ip 10.10.10.1
 standby 10 priority 110
 standby 10 preempt
```

**CORE-SW2**

```cisco
interface Vlan10
 ip address 10.10.10.3 255.255.255.0
 standby 10 ip 10.10.10.1
 standby 10 priority 100
 standby 10 preempt
```

The same addressing and priority pattern is used for the other configured departmental VLANs, with the corresponding subnet addresses and HSRP group numbers.

## 3. Core Switch Interconnections

The two core switches are interconnected through a trunk link.

| Device | Interface | Connection |
|---|---|---|
| CORE-SW1 | Fa0/2 | CORE-SW2 |
| CORE-SW2 | Fa0/2 | CORE-SW1 |

The trunk carries the configured VLANs:

```cisco
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,70,80,90,100,110,120,130,140
```

This allows VLAN traffic to traverse the inter-core connection.

## 4. Core-to-Firewall Redundant Paths

Each core switch has a routed connection to a separate firewall.

| Core switch | Interface | Firewall | Firewall interface | Network |
|---|---|---|---|---|
| CORE-SW1 | Fa0/1 | FW-PRIMARY | Gi1/1 | 10.10.254.0/30 |
| CORE-SW2 | Fa0/1 | FW-SECONDARY | Gi1/1 | 10.10.254.4/30 |

### IP addressing

| Device | IP address |
|---|---|
| FW-PRIMARY inside | 10.10.254.1/30 |
| CORE-SW1 Fa0/1 | 10.10.254.2/30 |
| FW-SECONDARY inside | 10.10.254.5/30 |
| CORE-SW2 Fa0/1 | 10.10.254.6/30 |

### Static default routes

CORE-SW1 uses FW-PRIMARY as its preferred default route and FW-SECONDARY as a higher-distance alternative.

```cisco
ip route 0.0.0.0 0.0.0.0 10.10.254.1
ip route 0.0.0.0 0.0.0.0 10.10.254.5 200
```

CORE-SW2 uses FW-SECONDARY as its preferred default route and FW-PRIMARY as a higher-distance alternative.

```cisco
ip route 0.0.0.0 0.0.0.0 10.10.254.5
ip route 0.0.0.0 0.0.0.0 10.10.254.1 200
```

The higher administrative distance makes the alternative static route less preferred while the primary route is available.

**Important:** Floating static routes provide an alternative route when the primary route is removed from the routing table. Without suitable tracking or route monitoring, a failed upstream path may not automatically remove a still-reachable next-hop route.

## 5. Dual ISP Design

The network has two separate ISP-facing firewall connections.

| Firewall | ISP | Outside IP | ISP gateway |
|---|---|---|---|
| FW-PRIMARY | ISP1 | 203.0.113.2/30 | 203.0.113.1 |
| FW-SECONDARY | ISP2 | 198.51.100.2/30 | 198.51.100.1 |

The primary firewall connects to ISP1, while the secondary firewall connects to ISP2.

The design provides two WAN paths at the firewall level. However, the two firewalls operate independently, and automatic firewall failover or complete end-to-end ISP failover has not been demonstrated.

ISP1 was successfully tested through INTERNET-R1 to the simulated public server at 8.8.8.8. ISP2's onward internet connectivity remains unverified.

## 6. Quality of Service (QoS)

QoS is used to identify and prioritize latency-sensitive voice traffic.

The lab uses a class map to identify voice traffic marked with DSCP Expedited Forwarding (EF).

### QoS classification

```cisco
class-map match-any VOICE-TRAFFIC
 match ip dscp ef
```

### QoS policy

```cisco
policy-map VOICE-POLICY
 class VOICE-TRAFFIC
  priority 10
```

The policy assigns a priority queue to the classified voice traffic.

### QoS application

The VOICE-POLICY was applied in the output direction on selected core-to-access switch uplinks.

The intention is to prioritize voice traffic during periods of congestion and improve the quality of real-time communication.

**Note:** QoS effectiveness depends on traffic being correctly marked and on the policy being applied to the relevant interfaces. Full congestion and voice-quality testing has not been documented.

## 7. Redundancy and Failover Verification

The lab includes the following configured redundancy mechanisms:

| Mechanism | Implementation | Verification status |
|---|---|---|
| HSRP gateway redundancy | CORE-SW1 and CORE-SW2 | Active/standby roles observed |
| Core switch interconnection | Trunk link | Configured |
| Core-to-firewall paths | Separate firewall connections | Configured |
| Floating static routes | Higher-distance backup routes | Configured |
| Dual ISP connections | ISP1 and ISP2 | Configured at firewall WAN interfaces |
| Automatic ISP failover | Not fully implemented or verified | Not confirmed |
| Firewall high availability | Independent firewall configurations | Not confirmed |
| Voice QoS | DSCP EF classification and priority policy | Configured; performance not fully tested |

### HSRP verification commands

```cisco
show standby brief
show standby
show ip interface brief
```

### Routing verification commands

```cisco
show ip route
show ip interface brief
show running-config
```

### QoS verification commands

```cisco
show policy-map interface
show class-map
show policy-map
```

Command availability may vary depending on the Packet Tracer device model and simulated IOS feature support.

## 8. Known Limitations

The following limitations should be considered when evaluating the lab:

1. HSRP active and standby roles were observed, but a complete failover test with all affected VLANs and applications has not been documented.
2. Floating static routes are configured, but reliable automatic upstream failure detection has not been demonstrated.
3. The two ASA firewalls operate independently; a fully functioning firewall high-availability pair has not been verified.
4. ISP1 has a tested path to the simulated public web server, while ISP2's onward connectivity remains incomplete.
5. QoS classification and prioritization are configured, but performance under congestion has not been measured.

## 9. Conclusion

The enterprise network incorporates important availability and traffic-management mechanisms, including HSRP, alternate static routes, separate ISP-facing firewall connections and voice QoS.

These configurations demonstrate how redundancy and traffic prioritization can be incorporated into an enterprise network design.

Further testing is required to demonstrate complete gateway failover, upstream route recovery, firewall high availability, dual-ISP failover and QoS performance under load.
