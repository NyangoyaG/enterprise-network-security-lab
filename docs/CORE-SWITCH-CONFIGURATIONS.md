# Core Switch Configurations

## 1. Overview

The enterprise network uses two Layer 3 core switches:

- CORE-SW1: Active HSRP gateway.
- CORE-SW2: Standby HSRP gateway.

Both switches provide inter-VLAN routing, gateway redundancy and connectivity to access switches and their respective firewalls.

## 2. CORE-SW1

### Device details

- Hostname: CORE-SW1
- IOS: 12.2(37)SE1
- Routing: Layer 3 enabled
- HSRP priority: 110
- HSRP role: Active

### Routed firewall connection

| Interface | Connected device | IP address |
|---|---|---|
| Fa0/1 | FW-PRIMARY | 10.10.254.2/30 |

### Trunk connections

| Interface | Connected device |
|---|---|
| Fa0/2 | CORE-SW2 |
| Fa0/3 | ACCESS-SW1 |
| Fa0/4 | ACCESS-SW2 |
| Fa0/5 | ACCESS-SW3 |
| Fa0/6 | ACCESS-SW4 |

### HSRP configuration

CORE-SW1 uses priority 110 and preemption on the configured VLAN interfaces.

| VLAN | Switch IP | Virtual IP |
|---|---|---|
| 10 | 10.10.10.2 | 10.10.10.1 |
| 20 | 10.10.20.2 | 10.10.20.1 |
| 30 | 10.10.30.2 | 10.10.30.1 |
| 40 | 10.10.40.2 | 10.10.40.1 |
| 50 | 10.10.50.2 | 10.10.50.1 |
| 60 | 10.10.60.2 | 10.10.60.1 |
| 80 | 10.10.80.2 | 10.10.80.1 |
| 90 | 10.10.90.2 | 10.10.90.1 |
| 100 | 10.10.100.2 | 10.10.100.1 |
| 110 | 10.10.110.2 | 10.10.110.1 |
| 120 | 192.168.4.2 | 192.168.4.1 |
| 130 | 10.10.130.2 | 10.10.130.1 |
| 140 | 10.10.140.2 | 10.10.140.1 |

### Default routing

- Primary default route: 10.10.254.1
- Backup default route: 10.10.254.5, administrative distance 200

## 3. CORE-SW2

### Device details

- Hostname: CORE-SW2
- IOS: 12.2(37)SE1
- Routing: Layer 3 enabled
- HSRP priority: Default 100
- HSRP role: Standby

### Routed firewall connection

| Interface | Connected device | IP address |
|---|---|---|
| Fa0/1 | FW-SECONDARY | 10.10.254.6/30 |

### Trunk connections

| Interface | Connected device |
|---|---|
| Fa0/2 | CORE-SW1 |
| Fa0/3 | ACCESS-SW5 |
| Fa0/4 | ACCESS-SW6 |
| Fa0/5 | ACCESS-SW7 |
| Fa0/6 | ACCESS-SW8 / DMZ |

### HSRP configuration

CORE-SW2 uses the default HSRP priority of 100 and preemption on the configured VLAN interfaces.

| VLAN | Switch IP | Virtual IP |
|---|---|---|
| 10 | 10.10.10.3 | 10.10.10.1 |
| 20 | 10.10.20.3 | 10.10.20.1 |
| 30 | 10.10.30.3 | 10.10.30.1 |
| 40 | 10.10.40.3 | 10.10.40.1 |
| 50 | 10.10.50.3 | 10.10.50.1 |
| 60 | 10.10.60.3 | 10.10.60.1 |
| 80 | 10.10.80.3 | 10.10.80.1 |
| 90 | 10.10.90.3 | 10.10.90.1 |
| 100 | 10.10.100.3 | 10.10.100.1 |
| 110 | 10.10.110.3 | 10.10.110.1 |
| 120 | 192.168.4.3 | 192.168.4.1 |
| 130 | 10.10.130.3 | 10.10.130.1 |
| 140 | 10.10.140.3 | 10.10.140.1 |

### Default routing

- Primary default route: 10.10.254.5
- Backup default route: 10.10.254.1, administrative distance 200

## 4. Quality of Service

Both core switches contain:

- Class map: VOICE-TRAFFIC
- Match condition: DSCP EF
- Policy map: VOICE-POLICY
- Priority allocation: 10%

The policy is intended to prioritize voice traffic on selected core-to-access uplinks.

## 5. Security and Management

Both switches include:

- SSH version 2.
- Restricted SSH management access.
- Local administrative authentication.
- Access-control lists for agent traffic.
- RADIUS authentication configuration.

Credentials and shared secrets are intentionally excluded from this documentation.

## 6. Verification Commands

Use the following commands to verify the core network:

### HSRP

show standby brief

### VLANs

show vlan brief

### Trunk links

show interfaces trunk

### Interface status

show ip interface brief

### Routing table

show ip route

### QoS

show policy-map

### ACL configuration

show access-lists

## 7. Implementation Limitations

- HSRP gateway redundancy is configured, but complete failure scenarios should be tested.
- VLAN 70 is reserved for voice QoS and currently has no SVI.
- Agent isolation has not been fully verified.
- Automatic firewall and ISP failover has not been demonstrated.
- ACL placement and ordering require further review.
