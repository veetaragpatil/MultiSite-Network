# Routing Design

Routing in this network combines **EIGRP (AS 1)** for internal site-to-site reachability with **static default routes** for Internet access and **NAT** for public translation.

## 1. EIGRP — Autonomous System 1

EIGRP runs between **HQ-EDGE-R1** and **DC-R1** across the `192.168.2.0/24` WAN link, distributing the internal subnets so both sites can reach each other.

### HQ-EDGE-R1

```
router eigrp 1
 network 192.168.2.0
 network 192.168.1.0 0.0.0.63
 network 192.168.1.64 0.0.0.63
 network 192.168.1.128 0.0.0.63
```

Advertises: the WAN link plus all three HQ VLAN `/26` subnets (using wildcard masks `0.0.0.63`).

### DC-R1

```
router eigrp 1
 network 192.168.3.0
 network 192.168.2.0
```

Advertises: the Data Center LAN (`192.168.3.0/24`) and the WAN link.

### Resulting Reachability

- DC-R1 learns the three HQ VLAN subnets via EIGRP.
- HQ-EDGE-R1 learns the Data Center LAN `192.168.3.0/24` via EIGRP.

## 2. Default / Static Routing

| Device      | Static Route                                  | Purpose                                  |
| ----------- | --------------------------------------------- | ---------------------------------------- |
| HQ-EDGE-R1  | `ip route 0.0.0.0 0.0.0.0 216.0.5.1`          | Send Internet-bound traffic to the ISP   |
| DC-R1       | `ip route 0.0.0.0 0.0.0.0 192.168.2.1`        | Send Internet/unknown traffic to HQ edge |

The DC site relies on the HQ edge router for Internet access: traffic leaves DC-R1 over the WAN to HQ-EDGE-R1, which then NATs it and forwards it to ISP-R1.

## 3. ISP-R1

ISP-R1 has no internal routes configured — it only owns the public link `216.0.5.0/24` (directly connected) plus two loopbacks:

- `Loopback1 = 1.1.1.1/32` — used as the DNS server address handed out by DHCP.
- `Loopback2 = 8.8.8.8/32` — a simulated public Internet host for connectivity testing.

Because internal addresses are translated by NAT before reaching the ISP, ISP-R1 does not need routes back to the private networks.

## Traffic Flow Examples

**HQ PC → Internet (8.8.8.8):**
```
PC (VLAN2) → HQ-SW1/2 (trunk) → HQ-EDGE-R1 G0/0.20 → PAT (216.0.5.2) → G0/1 → ISP-R1
```

**Internet → DC Server (216.0.5.10):**
```
ISP-R1 → 216.0.5.2 (HQ-EDGE-R1 G0/1) → Static NAT (→192.168.3.10) → G0/0/0 → DC-R1 → DC-SW1 → Server
```

**HQ PC → DC Server (internal):**
```
PC → HQ-EDGE-R1 → EIGRP route 192.168.3.0/24 → WAN → DC-R1 → DC-SW1 → Server
```
