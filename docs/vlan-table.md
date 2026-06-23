# VLAN Table

The HQ site uses three VLANs for departmental segmentation. Each VLAN maps to a `/26` subnet and is routed through a dedicated 802.1Q sub-interface on **HQ-EDGE-R1** (router-on-a-stick). Access switches **HQ-SW1** and **HQ-SW2** assign edge ports to the VLANs and trunk the tagged traffic toward the router.

## VLANs

| VLAN ID | dot1Q Tag | Sub-interface | Network            | Gateway        | Purpose (assumed) |
| ------- | --------- | ------------- | ------------------ | -------------- | ----------------- |
| 2       | 2         | G0/0.20       | 192.168.1.0/26     | 192.168.1.1    | Department A      |
| 3       | 3         | G0/0.30       | 192.168.1.64/26    | 192.168.1.65   | Department B      |
| 4       | 4         | G0/0.40       | 192.168.1.128/26   | 192.168.1.129  | Department C      |

## Access Port Assignments

### HQ-SW1

| Ports          | Mode    | VLAN |
| -------------- | ------- | ---- |
| Fa0/1          | Trunk   | —    |
| Fa0/2          | Trunk   | —    |
| Fa0/4 – Fa0/8  | Access  | 2    |
| Fa0/9 – Fa0/13 | Access  | 3    |
| Fa0/14 – Fa0/18| Access  | 4    |

### HQ-SW2

| Ports          | Mode    | VLAN |
| -------------- | ------- | ---- |
| Fa0/2          | Trunk   | —    |
| Fa0/4 – Fa0/8  | Access  | 2    |
| Fa0/9 – Fa0/13 | Access  | 3    |
| Fa0/14 – Fa0/18| Access  | 4    |

## Trunking Notes

- **HQ-SW1 Fa0/1** trunks to **HQ-EDGE-R1 G0/0** (carries the tagged VLAN 2/3/4 traffic for inter-VLAN routing).
- **HQ-SW1 Fa0/2 ↔ HQ-SW2 Fa0/2** trunk link interconnects the two access switches so devices on either switch share the same VLANs.

## Sub-interface vs. Tag Mapping (Important)

The sub-interface numbering on HQ-EDGE-R1 does **not** match its dot1Q tag, which can be confusing but is functionally valid because IOS uses the `encapsulation dot1Q` value (not the sub-interface number) for tagging:

| Sub-interface | Configured dot1Q tag | Matches VLAN |
| ------------- | -------------------- | ------------ |
| G0/0.20       | 2                    | VLAN 2       |
| G0/0.30       | 3                    | VLAN 3       |
| G0/0.40       | 4                    | VLAN 4       |

> Note: `G0/0.4` also exists with `no ip address` and no encapsulation. See [troubleshooting.md](troubleshooting.md).
