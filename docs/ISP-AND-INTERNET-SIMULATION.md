# ISP and Internet Simulation

## 1. Overview

This document describes the ISP connectivity and simulated internet environment implemented in the Enterprise Network Security Lab using Cisco Packet Tracer.

The design uses two ISP routers connected to separate perimeter firewalls. ISP1 provides the tested connection to the simulated internet through an Internet edge router (INTERNET-R1), which connects to a public web server.

ISP2 provides a secondary WAN connection to the second firewall. Its end-to-end connectivity to the simulated internet has not yet been verified.

## 2. Network Topology

The simulated internet path is:

```text
Corporate Network
       |
   Core Switches
       |
   ASA Firewalls
       |
  +----+----------------+
  |                     |
FW-PRIMARY          FW-SECONDARY
  |                     |
 ISP1                  ISP2
  |
INTERNET-R1
  |
PUBLIC-WEB-SERVER
  |
  8.8.8.8
```

**Note:** ISP2 is connected to FW-SECONDARY, but a complete onward path from ISP2 to the public web server has not been implemented or verified.

## 3. IP Addressing Plan

### ISP1

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| ISP1 | GigabitEthernet0/0 | 203.0.113.1/30 | Connection to FW-PRIMARY |
| ISP1 | GigabitEthernet0/1 | 203.0.113.5/30 | Connection to INTERNET-R1 |

### ISP2

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| ISP2 | GigabitEthernet0/0 | 198.51.100.1/30 | Connection to FW-SECONDARY |
| ISP2 | GigabitEthernet0/1 | Shutdown | Not configured for onward internet connectivity |
| ISP2 | GigabitEthernet0/2 | Shutdown | Not configured for onward internet connectivity |

### INTERNET-R1

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| INTERNET-R1 | GigabitEthernet0/0 | 203.0.113.6/30 | Connection to ISP1 |
| INTERNET-R1 | GigabitEthernet0/1 | 8.8.8.1/24 | Public server network |

### Public Web Server

| Setting | Configuration |
|---|---|
| Device | PUBLIC-WEB-SERVER |
| IP Address | 8.8.8.8 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 8.8.8.1 |
| DNS Server | 8.8.8.1 |
| HTTP | Enabled |
| HTTPS | Enabled |

## 4. ISP1 Configuration

ISP1 provides the primary simulated internet path.

Its GigabitEthernet0/0 interface connects to FW-PRIMARY, while GigabitEthernet0/1 connects to INTERNET-R1.

### Interface configuration

```cisco
interface GigabitEthernet0/0
 description CONNECTION-TO-FW-PRIMARY
 ip address 203.0.113.1 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/1
 description CONNECTION-TO-INTERNET-R1
 ip address 203.0.113.5 255.255.255.252
 no shutdown
```

### Static route

```cisco
ip route 8.8.8.0 255.255.255.0 203.0.113.6
```

This route directs traffic destined for the simulated public server network through INTERNET-R1.

## 5. ISP2 Configuration

ISP2 provides the secondary WAN connection to FW-SECONDARY.

### Interface configuration

```cisco
interface GigabitEthernet0/0
 description CONNECTION-TO-FW-SECONDARY
 ip address 198.51.100.1 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/1
 shutdown
!
interface GigabitEthernet0/2
 shutdown
```

ISP2 currently has no configured onward connection to INTERNET-R1 or the public web server. Therefore, its role in this lab is limited to the secondary firewall-facing WAN link.

## 6. INTERNET-R1 Configuration

INTERNET-R1 simulates an internet edge router connecting ISP1 to the public web server network.

### Interface configuration

```cisco
interface GigabitEthernet0/0
 description CONNECTION-TO-ISP1
 ip address 203.0.113.6 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/1
 description PUBLIC-SERVER-NETWORK
 ip address 8.8.8.1 255.255.255.0
 no shutdown
```

### Static route

```cisco
ip route 203.0.113.0 255.255.255.0 203.0.113.5
```

This route directs traffic for the 203.0.113.0/24 network through ISP1.

## 7. Firewall WAN Connectivity

The perimeter firewalls connect to separate ISP routers.

| Firewall | Outside Interface | Outside IP | ISP Gateway |
|---|---|---|---|
| FW-PRIMARY | GigabitEthernet1/2 | 203.0.113.2/30 | 203.0.113.1 |
| FW-SECONDARY | GigabitEthernet1/2 | 198.51.100.2/30 | 198.51.100.1 |

The primary firewall uses ISP1 as its default route. The secondary firewall uses ISP2 as its default route.

The two firewalls are independently configured. Automatic firewall or ISP failover has not been demonstrated in this lab.

## 8. Connectivity Verification

The following tests were performed during implementation.

| Source | Destination | Result | Observation |
|---|---|---|---|
| ISP1 | 8.8.8.8 | Successful | Public server reachable |
| INTERNET-R1 | 8.8.8.8 | Successful | Public server reachable through directly connected network |
| FW-PRIMARY | 203.0.113.1 | Failed | ISP1 gateway connectivity was not established from the firewall |
| FW-PRIMARY | 8.8.8.8 | Failed | End-to-end connectivity through FW-PRIMARY was not established |
| ISP2 | 8.8.8.8 | Not verified | No onward ISP2 internet path configured |

### Verification commands

The following commands can be used to inspect router connectivity and routing information:

```cisco
show ip interface brief
show ip route
show running-config
show ip arp
ping 8.8.8.8
traceroute 8.8.8.8
```

The firewall WAN interface was observed as up/up, but its interface counters showed no traffic during the failed connectivity tests. The firewall-to-ISP1 path therefore requires further troubleshooting.

## 9. Current Implementation Status

| Component | Status |
|---|---|
| ISP1 to INTERNET-R1 connection | Configured |
| INTERNET-R1 to public web server | Configured |
| Public web server HTTP/HTTPS | Enabled |
| ISP1 to public server connectivity | Tested successfully |
| FW-PRIMARY to ISP1 connectivity | Not working during testing |
| ISP2 to FW-SECONDARY connection | Configured |
| ISP2 onward internet connectivity | Not implemented/verified |
| Automatic ISP failover | Not verified |

## 10. Conclusion

The lab demonstrates how an enterprise network can connect to a simulated external network through an ISP router and an internet edge router.

The primary ISP path between ISP1, INTERNET-R1 and the public web server was successfully tested. The secondary ISP connection was configured between ISP2 and FW-SECONDARY, but its onward connectivity remains incomplete.

Further work is required to troubleshoot the FW-PRIMARY WAN path and implement and test end-to-end connectivity through ISP2 before claiming full dual-ISP failover.
