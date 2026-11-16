# network-register

As of November 2026, the address scheme and role prefixes are recorded.
No segment or host is built yet.

## Address scheme

| Item | Value |
|---|---|
| RFC 1918 range in use | `10.20.0.0/16` |
| Rule deriving a segment's range from its ID | Segment n uses `10.20.n.0/24`, for IDs 1 to 254: `.1` gateway, DHCP, and DNS; `.10` to `.99` static; `.100` to `.199` DHCP pool |

## Segments

| ID | Name | Range | Enforced by | Isolated |
|---|---|---|---|---|

## Role prefixes

| Role | Prefix |
|---|---|
| Domain controller | `DC` |
| Member server | `SRV` |
| SIEM | `SIEM` |
| Workstation | `WS` |
| Attack host | `ATK` |

## Hosts

| Host | Role | Segment | Address |
|---|---|---|---|

## Diagram

```
```
