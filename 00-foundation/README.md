# 00-foundation

This layer covers the host, the hypervisor, storage layout, segmentation
and addressing, the separation of the lab plane from the host plane,
host backups, the naming scheme every later layer assumes, and
amendments to the standards. Guest builds belong to `10-endpoint/`,
apart from the test guest that the first exit criterion calls for.
Directory services belong to `20-server/`, and CCNA-scope switching and
routing labs to the netops-notes repository.

## Exit criteria

- [ ] Virtualization stack running on the host, with a test guest built,
  snapshotted, and reverted, and the revert observed.
- [ ] Host storage layout settled against measured free space, with the
  concurrent-guest ceiling stated as a number.
- [ ] 12.2 Establish and Maintain a Secure Network Architecture: the
  segmentation and address scheme is set, and no guest can reach the
  home network, any of the author's machines, or any host service
  beyond the DHCP and DNS its own segment provides. A failed connection
  attempt from a guest confirms each boundary.
- [ ] Isolated segment: a guest on the isolated segment has no path to
  the WAN, confirmed by a failed connection attempt and a failed lookup
  of an external name.
- [ ] Guest-to-host channels: no guest has a shared folder, a shared
  clipboard, or a device passed through from a host, confirmed from each
  guest's configuration on the hypervisor.
- [ ] 1.1 Establish and Maintain Detailed Enterprise Asset Inventory:
  every host and segment is recorded in the [network
  register](network-register.md).
- [ ] 12.4 Establish and Maintain Architecture Diagram(s): the network
  register's diagram matches its tables.
- [ ] 11.1 Establish and Maintain a Data Recovery Process: the scope,
  frequency, and restore order of host backups are recorded in an entry.
- [ ] 11.2 Perform Automated Backups: host backups run on a schedule
  without manual action.
- [ ] 11.3 Protect Recovery Data: backup data has access controls
  equivalent to those on the data it holds, and is encrypted or kept
  separate.
- [ ] 11.4 Establish and Maintain an Isolated Instance of Recovery Data:
  one copy of recovery data is held where the host cannot write to it.
- [ ] 11.5 Test Data Recovery: a guest is restored from backup and the
  restore is read against a baseline.

## Entries

| ID | Entry | Tier | Status | Superseded by |
|---|---|---|---|---|
| LAB-0001 | [Lab address scheme and target topology](LAB-0001-address-scheme-and-topology/address-scheme-and-topology.md) | L0 | complete | |

## Open gaps

- LAB-0001/guest-ceiling-unmeasured: the ceiling of about five
  concurrent guests is an estimate.
- LAB-0001/segment-99-undefined: segment 99 has a range and no rules
  until HANDLING.md exists.
