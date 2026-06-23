# Security Features

This network applies baseline device-hardening and address-hiding measures. The table below reflects exactly what is present in the running configurations.

## Implemented Controls

| Control                       | Device(s)              | Detail                                              |
| ----------------------------- | ---------------------- | --------------------------------------------------- |
| Enable Secret (hashed)        | HQ-SW1                 | `enable secret 5 $1$mERr$...` (MD5 hash stored)     |
| Console password              | All routers & switches | `line con 0` → `password cisco` / `login`           |
| VTY password (remote)         | Most devices           | `line vty 0 4` → `password cisco` / `login`         |
| NAT address hiding            | HQ-EDGE-R1             | Internal `192.168.x.x` hidden behind `216.0.5.2`    |
| Standard ACL scoping NAT      | HQ-EDGE-R1             | ACL 1 limits which subnets are translated           |

## Standard ACL (ACL 1)

ACL 1 on HQ-EDGE-R1 defines the inside source addresses eligible for PAT translation:

```
access-list 1 permit 192.168.1.64 0.0.0.63
access-list 1 permit 192.168.1.128 0.0.0.63
access-list 1 permit 192.168.1.0 0.0.0.63
```

This permits all three HQ VLAN `/26` subnets and is referenced by:

```
ip nat inside source list 1 interface GigabitEthernet0/1 overload
```

## Observations & Gaps (Hardening Opportunities)

These are not configured today and are recommended in the README's *Future Improvements*:

- **No SSH** — remote management uses plaintext VTY passwords only (Telnet-style). No `transport input ssh`, no crypto key, no domain name.
- **No `enable secret` on routers** — only HQ-SW1 has one.
- **No service password-encryption** — line passwords are stored in clear text (`no service password-encryption`).
- **No port security** on access switches.
- **Unused switch ports left enabled** — Fa0/19–24 and the Gig uplinks are not shut down.
- **ISP-R1 VTY** has `login` with no password set, which blocks remote login but is not an intentional control.
- **Identical line passwords** (`cisco`) reused across all devices.
