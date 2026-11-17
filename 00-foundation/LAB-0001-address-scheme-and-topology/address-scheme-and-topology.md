---
id: LAB-0001
title: Lab address scheme and target topology
author: Kenneth Brizuela
layer: 00-foundation
date: 2026-11-15
status: complete
tier: L0
redacted: false
prereqs: []
supersedes: ""
tags: [addressing, segmentation, topology, libvirt]
gaps: [guest-ceiling-unmeasured, segment-99-undefined]
closes_gaps: []
controls: []
---

```text
1.  CONTEXT

    The host is one laptop running Fedora 45 Workstation, with the
    virtualization stack installed and KVM managed through libvirt.
    Earlier test guests were created and deleted while trying the
    stack out. As of this entry, no guest exists, and the only
    libvirt connection is the default QEMU/KVM one, with its `default`
    network. The lab has no segments and no address plan.

    Every later entry needs to name hosts and addresses. Choosing them
    one lab at a time would leave ranges that collide and names that
    follow no pattern. This entry fixes the address scheme, the
    segments, and the planned guests before the first build.

    No prior entries. This is the first.

2.  OPTIONS CONSIDERED

    2.1.  Keep the libvirt default network, 192.168.122.0/24

          Case for:  Requires no further setup post-install.
          Why not:   One flat network cannot separate servers,
                     workstations, and an attack host. 192.168.0.0/16 is
                     also the range most residential routers use, so an
                     overlap with the host plane is likely.

    2.2.  One /24 per segment inside 172.16.0.0/12

          Case for:  Rarely used in residential settings, so overlap is
                     unlikely.
          Why not:   The /12 boundary is easy to get wrong by hand, and
                     nothing ties a segment's ID to its range.

    2.3.  Decide the guest list later, one entry at a time

          Case for:  Less to commit to now.
          Why not:   Role prefixes are permanent once used, and the
                     host cannot run every guest at once. Both need
                     the full list to plan against.

    2.4.  10.20.0.0/16, with segment n at 10.20.n.0/24     [CHOSEN]

          Case for:  The third octet is the segment ID, so an address
                     shows its segment at a glance. The range is
                     outside what residential routers lease by default.
          Why not:   The rule only works for segment IDs 1 to 254,
                     fewer than the 1 to 4094 that conventions.md
                     allows. Accepted.

3.  DECISION

    The lab plane uses 10.20.0.0/16. Segment n uses 10.20.n.0/24.
    Inside each segment:

        .1              gateway, DHCP, and DNS
        .10 to .99      static addresses
        .100 to .199    DHCP pool

    Four segments are planned:

        ID   Name           Range
        10   servers        10.20.10.0/24
        20   workstations   10.20.20.0/24
        30   attack         10.20.30.0/24
        99   isolated       10.20.99.0/24

    Six guests are planned, with these role prefixes:

        Host     Role                       Segment
        DC01     domain controller          10
        SRV01    member server, WEF         10
        SIEM01   SIEM, all-in-one           10
        WS01     workstation                20
        WS02     workstation                20
        ATK01    attack host                30

    About five guests can run at once. That figure is an estimate,
    recorded in section 6.

    The reason for this scheme is that an address should tell the
    reader where a host sits without opening the register.

    This entry builds nothing. It fills the Address scheme and Role
    prefixes tables in 00-foundation/network-register.md and the
    Current stack section of README.md. Segment and host rows are
    added by the entries that build them.

4.  CONSEQUENCES

    o  Every later entry uses these ranges and prefixes. The prefixes
       DC, SRV, SIEM, WS, and ATK are permanent.

    o  Segment IDs above 254 are closed off unless a later entry changes
       the rule.

    o  Cost: six guests are planned and about five fit at once, so some
       labs will need a guest shut down first.

    o  Residual risk: nothing enforces this plan yet. Until an entry
       builds segment 10 and its boundary, the scheme is a document and
       not a control.

5.  IN PLAIN TERMS

    Before building anything, the lab gets a street map. Each
    neighborhood (segment) has a number, and that number appears in
    every address in it. Each kind of machine gets a short name it keeps
    for good.

6.  GAPS

    o  guest-ceiling-unmeasured

       The ceiling of about five guests is an estimate. Measuring free
       memory and storage on the host would resolve it.

    o  segment-99-undefined

       Segment 99 has a range and no rules. HANDLING.md is unwritten,
       and the segment is not built until it exists.

7.  REFERENCES

    o  RFC 1918, Address Allocation for Private Internets
       https://www.rfc-editor.org/rfc/rfc1918

    o  CIS Controls v8.1, 12.2 Establish and Maintain a Secure Network
       Architecture
       https://cas.docs.cisecurity.org/en/latest/source/Controls12/#122-establish-and-maintain-a-secure-network-architecture

    o  conventions.md, Naming and addressing
```
