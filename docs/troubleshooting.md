# Troubleshooting & Known Configuration Issues

During documentation, the running configurations were reviewed against the topology. The items below are genuine inconsistencies found in the configs. They are recorded here both as a troubleshooting log and as a guide for anyone reproducing the lab.

---

## Issue 1 — Duplicate IP address on HQ-SW2 (VLAN1 and VLAN3)

### Problem
HQ-SW2 has the **same IP address assigned to two SVIs**:

```
interface Vlan1
 ip address 192.168.1.66 255.255.255.192
interface Vlan3
 ip address 192.168.1.66 255.255.255.192
```

### Impact
Only one SVI can be active with a given address at a time; this produces an overlapping/duplicate address condition and unpredictable management reachability. A switch should have a single management SVI.

### Recommended Fix
Keep management on one VLAN only (VLAN 3 matches its `192.168.1.65` gateway):

```
interface Vlan1
 no ip address
 shutdown
interface Vlan3
 ip address 192.168.1.66 255.255.255.192
 no shutdown
ip default-gateway 192.168.1.65
```

---

## Issue 2 — Orphaned / mis-numbered sub-interface on HQ-EDGE-R1

### Problem
HQ-EDGE-R1 has an extra sub-interface with no encapsulation and no address:

```
interface GigabitEthernet0/0.4
 no ip address
```

The other sub-interfaces are numbered `.20/.30/.40` but tagged `dot1Q 2/3/4`. The mismatch between sub-interface number and VLAN tag is legal in IOS but is a maintenance hazard.

### Impact
`G0/0.4` does nothing (no IP, no encapsulation). The number/tag mismatch makes the config harder to read and audit.

### Recommended Fix
Remove the orphan and align sub-interface numbers with VLAN IDs for clarity:

```
no interface GigabitEthernet0/0.4
```
And (optional, cosmetic) renumber `.20/.30/.40` to `.2/.3/.4` so the sub-interface number matches the dot1Q tag.

---

## Issue 3 — HQ-SW2 has only one trunk; missing uplink redundancy

### Problem
HQ-SW2 trunks only on `Fa0/2` (to HQ-SW1). It has no direct trunk to the edge router, so all of HQ-SW2's VLAN traffic depends on the single HQ-SW1 ↔ HQ-SW2 link.

### Impact
Single point of failure for HQ-SW2 connectivity. No redundancy / no EtherChannel.

### Recommended Fix
Add a redundant trunk (and consider EtherChannel + STP tuning) between the access switches, or dual-home HQ-SW2.

---

## Issue 4 — DHCP excluded ranges don't all align to pools

### Problem
HQ-EDGE-R1 excludes four ranges:

```
ip dhcp excluded-address 192.168.1.1 192.168.1.10      (vlan2 pool)
ip dhcp excluded-address 192.168.1.65 192.168.1.74     (vlan3 pool)
ip dhcp excluded-address 192.168.1.129 192.168.1.138   (vlan4 pool)
ip dhcp excluded-address 192.168.1.193 192.168.1.202   (no matching /26 pool)
```

The `192.168.1.193 – 192.168.1.202` range falls in the `192.168.1.192/26` network, for which **no DHCP pool or VLAN exists**.

### Impact
Harmless today (no pool serves that range) but indicates either a planned-but-missing 4th VLAN or a leftover from an earlier design.

### Recommended Fix
Remove the unused exclusion, or add the corresponding VLAN/pool if a fourth segment is intended.

---

## General Troubleshooting Methodology

When inter-VLAN or Internet connectivity fails in this topology, work bottom-up:

1. **Physical/Access:** `show ip interface brief`, `show interfaces status` — confirm ports up and in the right VLAN.
2. **Switching:** `show vlan brief`, `show interfaces trunk` — confirm VLANs exist and trunks carry them.
3. **Routing:** `show ip route`, `show ip eigrp neighbors` — confirm EIGRP adjacency and learned routes.
4. **Services:** `show ip dhcp binding`, `show ip nat translations` — confirm DHCP leases and NAT entries.
5. **End-to-end:** `ping` then `traceroute` to localize where the path breaks.
