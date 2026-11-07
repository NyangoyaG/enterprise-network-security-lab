# Firewall and WAN Configurations

## 1. Overview

The enterprise network uses two Cisco ASA firewalls to provide separate security gateway paths through two ISP connections.

- FW-PRIMARY connects to ISP1.
- FW-SECONDARY connects to ISP2.

Both firewalls provide inside/outside security zones, NAT/PAT and static routing.

## 2. FW-PRIMARY

- ASA version: 9.6(1)
- Inside interface: GigabitEthernet1/1
- Inside IP: 10.10.254.1/30
- Outside interface: GigabitEthernet1/2
- Outside IP: 203.0.113.2/30
- Failover-link interface: GigabitEthernet1/3
- Failover-link IP: 172.16.255.1/30

### Routing

Internal network route:
10.10.0.0/16 via 10.10.254.2

Default route:
0.0.0.0/0 via 203.0.113.1

### NAT

Dynamic interface PAT is configured for the internal network:

10.10.0.0/16

### Outside access control

ACL: OUTSIDE-IN

- Denies inbound traffic destined for 10.10.0.0/16.
- Denies inbound traffic destined for 192.168.4.0/25.
- The outside ACL is applied inbound on the outside interface.

## 3. FW-SECONDARY

- ASA version: 9.6(1)
- Inside interface: GigabitEthernet1/1
- Inside IP: 10.10.254.5/30
- Outside interface: GigabitEthernet1/2
- Outside IP: 198.51.100.2/30
- Failover-link interface: GigabitEthernet1/3
- Failover-link IP: 172.16.255.2/30

### Routing

Internal network route:
10.10.0.0/16 via 10.10.254.6

Default route:
0.0.0.0/0 via 198.51.100.1

### NAT

Dynamic interface PAT is configured for the internal network:

10.10.0.0/16

### Outside access control

ACL: OUTSIDE-IN

- Denies inbound traffic destined for 10.10.0.0/16.
- Denies inbound traffic destined for 192.168.4.0/25.
- Includes an explicit deny ip any any entry.
- The outside ACL is applied inbound on the outside interface.

## 4. WAN Addressing

| Connection | Local IP | Remote IP |
|---|---|---|
| FW-PRIMARY to ISP1 | 203.0.113.2/30 | 203.0.113.1 |
| FW-SECONDARY to ISP2 | 198.51.100.2/30 | 198.51.100.1 |

## 5. Security Architecture

The ASA firewalls provide:

- Network Address Translation.
- Internal network protection.
- DMZ protection.
- Outside interface access control.
- DNS, FTP and TFTP inspection policies.

No credentials or shared secrets are included in this documentation.

## 6. Verification Commands

show interface ip brief
show route
show nat
show xlate
show access-list
show running-config

## 7. Implementation Limitations

- The two ASA devices are configured independently.
- Native ASA failover has not been verified.
- Automatic ISP failover has not been demonstrated.
- FW-PRIMARY internet connectivity remains under investigation.
- FW-SECONDARY ISP connectivity has been tested at the gateway level.
- End-to-end WAN connectivity requires further validation.
