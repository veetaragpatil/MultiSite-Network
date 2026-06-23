# IP Addressing Plan

This network uses the private block `192.168.0.0/16` internally (HQ LAN, WAN, and DC LAN) and a simulated public block `216.0.5.0/24` for the ISP-facing connection. The HQ LAN (`192.168.1.0/24`) is subnetted into three equal `/26` networks (62 usable hosts each) for VLAN segmentation.

## Subnet Summary

| Network          | CIDR | Mask                | Usable Range                     | Purpose                  |
| ---------------- | ---- | ------------------- | -------------------------------- | ------------------------ |
| 192.168.1.0      | /26  | 255.255.255.192     | 192.168.1.1 – 192.168.1.62       | VLAN 2 (HQ)              |
| 192.168.1.64     | /26  | 255.255.255.192     | 192.168.1.65 – 192.168.1.126     | VLAN 3 (HQ)              |
| 192.168.1.128    | /26  | 255.255.255.192     | 192.168.1.129 – 192.168.1.190    | VLAN 4 (HQ)              |
| 192.168.2.0      | /24  | 255.255.255.0       | 192.168.2.1 – 192.168.2.254      | WAN link (HQ ↔ DC)       |
| 192.168.3.0      | /24  | 255.255.255.0       | 192.168.3.1 – 192.168.3.254      | Data Center LAN          |
| 216.0.5.0        | /24  | 255.255.255.0       | 216.0.5.1 – 216.0.5.254          | Public / ISP link        |

## Per-Network Gateways

| Network          | Purpose         | Gateway        |
| ---------------- | --------------- | -------------- |
| 192.168.1.0/26   | VLAN 2          | 192.168.1.1    |
| 192.168.1.64/26  | VLAN 3          | 192.168.1.65   |
| 192.168.1.128/26 | VLAN 4          | 192.168.1.129  |
| 192.168.2.0/24   | WAN HQ↔DC       | 192.168.2.1    |
| 192.168.3.0/24   | Data Center LAN | 192.168.3.1    |
| 216.0.5.0/24     | Public Internet | 216.0.5.1      |

## Full Device Interface Assignments

| Device      | Interface          | IP Address       | Subnet Mask         | Function                       |
| ----------- | ------------------ | ---------------- | ------------------- | ------------------------------ |
| ISP-R1      | G0/0               | 216.0.5.1        | 255.255.255.0       | Public gateway                 |
| ISP-R1      | Loopback1          | 1.1.1.1          | 255.255.255.255     | Simulated DNS resolver         |
| ISP-R1      | Loopback2          | 8.8.8.8          | 255.255.255.255     | Simulated public host          |
| HQ-EDGE-R1  | G0/1               | 216.0.5.2        | 255.255.255.0       | NAT outside (to ISP)           |
| HQ-EDGE-R1  | G0/0.20 (dot1Q 2)  | 192.168.1.1      | 255.255.255.192     | VLAN 2 gateway, NAT inside     |
| HQ-EDGE-R1  | G0/0.30 (dot1Q 3)  | 192.168.1.65     | 255.255.255.192     | VLAN 3 gateway, NAT inside     |
| HQ-EDGE-R1  | G0/0.40 (dot1Q 4)  | 192.168.1.129    | 255.255.255.192     | VLAN 4 gateway, NAT inside     |
| HQ-EDGE-R1  | G0/0/0             | 192.168.2.1      | 255.255.255.0       | WAN to DC, NAT inside          |
| HQ-SW1      | VLAN 2 (SVI)       | 192.168.1.2      | 255.255.255.0       | Management interface           |
| HQ-SW2      | VLAN 1 (SVI)       | 192.168.1.66     | 255.255.255.192     | Management (duplicate w/ VLAN3)|
| HQ-SW2      | VLAN 3 (SVI)       | 192.168.1.66     | 255.255.255.192     | Management (duplicate w/ VLAN1)|
| DC-R1       | G0/0               | 192.168.3.1      | 255.255.255.0       | DC LAN gateway                 |
| DC-R1       | G0/0/0             | 192.168.2.2      | 255.255.255.0       | WAN to HQ                       |
| DC-SW1      | VLAN 1 (SVI)       | 192.168.3.2      | 255.255.255.0       | Management interface           |
| DC Server   | NIC                | 192.168.3.10     | 255.255.255.0       | Static NAT target              |

## Static NAT Public Mappings

| Inside Local  | Inside Global | Type        |
| ------------- | ------------- | ----------- |
| 192.168.3.10  | 216.0.5.10    | Static NAT  |
| 192.168.3.11  | 216.0.5.11    | Static NAT  |
| 192.168.3.12  | 216.0.5.12    | Static NAT  |
