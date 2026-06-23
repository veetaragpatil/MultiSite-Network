# Testing & Verification

This document maps each verification activity to the captured CLI screenshots. The screenshots in the project (device `.png` files) are the command outputs collected from each device.

## Screenshot Inventory

| File                  | Device      | Captures                                  |
| --------------------- | ----------- | ----------------------------------------- |
| DC-R1.png             | DC-R1       | CLI verification output                    |
| DC-R1 (2).png         | DC-R1       | Additional CLI verification output         |
| HQ-EDGE-R1.png        | HQ-EDGE-R1  | CLI verification output                    |
| HQ-EDGE-R1 (2).png    | HQ-EDGE-R1  | Additional CLI verification output         |
| HQ-EDGE-R1 (3).png    | HQ-EDGE-R1  | Additional CLI verification output         |
| HQ-EDGE-R1 (4).png    | HQ-EDGE-R1  | Additional CLI verification output         |
| HQ-SW1.png            | HQ-SW1      | CLI verification output                    |
| HQ-SW1 (2).png        | HQ-SW1      | Additional CLI verification output         |
| HQ-SW2.png            | HQ-SW2      | CLI verification output                    |
| HQ-SW2 (2).png        | HQ-SW2      | Additional CLI verification output         |
| topology_image.png    | —           | Full network topology                      |

## Recommended Verification Commands

Run these per device to confirm the design is operating correctly.

### Routers (HQ-EDGE-R1, DC-R1, ISP-R1)
```
show ip interface brief
show ip route
show ip eigrp neighbors
show ip protocols
```

### NAT (HQ-EDGE-R1)
```
show ip nat translations
show ip nat statistics
show access-lists
```

### DHCP (HQ-EDGE-R1)
```
show ip dhcp binding
show ip dhcp pool
```

### Switches (HQ-SW1, HQ-SW2, DC-SW1)
```
show vlan brief
show interfaces trunk
show ip interface brief
```

## Test Cases

### Test 1 — Inter-VLAN Routing
**Action:** Ping a host in VLAN 3 from a host in VLAN 2.
**Expected:** PASS — traffic routes through HQ-EDGE-R1 sub-interfaces.

---

### Test 2 — DHCP Address Assignment
**Action:** Set HQ PCs to DHCP; confirm they receive addresses from the correct pool with gateway and DNS `1.1.1.1`.
**Expected:** PASS — addresses come from `192.168.1.x` ranges, excluding the reserved blocks.

---

### Test 3 — EIGRP Adjacency (HQ ↔ DC)
**Action:** `show ip eigrp neighbors` on HQ-EDGE-R1 and DC-R1.
**Expected:** PASS — neighbor over `192.168.2.0/24`; HQ learns `192.168.3.0/24`, DC learns the VLAN subnets.

---

### Test 4 — Internet Access via PAT
**Action:** Ping `8.8.8.8` from an HQ PC.
**Expected:** PASS — `show ip nat translations` shows the inside-local → `216.0.5.2` overload entries.

---

### Test 5 — Static NAT to DC Server
**Action:** From ISP-R1 (or a public host), reach `216.0.5.10`.
**Expected:** PASS — translates to `192.168.3.10`.

---

### Test 6 — DC Site Internet Path
**Action:** Ping `8.8.8.8` from a DC device.
**Expected:** PASS — default route forwards via WAN to HQ-EDGE-R1, then NAT to ISP.
