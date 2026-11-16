# security-operations-lab
 
This is a personal security operations homelab modeling a small
enterprise environment. Entries document labs, decisions made, and
evidence captured throughout.
 
## Document index
 
The following documents and templates govern the entries in this
repository:
 
- [`conventions.md`](conventions.md): File naming, directory layout,
  frontmatter schema, and entry structure.
- [`GOVERNANCE.md`](GOVERNANCE.md): Indexes the standards, defines
  precedence when two documents conflict, and sets the process for
  modifying a standard.
- `HANDLING.md`: Covers work with live samples and is unwritten as of
  October 2026.
- [`REDACTION.md`](REDACTION.md): Disclosure rules governing what may be
  published.
- [`writeup-standard.md`](writeup-standard.md): Section requirements,
  tier definitions, and completion criteria.
- [`tools/README.md`](tools/README.md): Documentation for the redaction
  utility and the workstation pre-push hook.
- [`templates/entry.md`](templates/entry.md): Base template for new
  entries.

## Layers
 
Layers are numbered in dependency order. A layer may rely on any layer
with a lower number. Each layer directory contains a README documenting
the layer's exit criteria, index of entries, and open gaps.
 
| Layer | Scope | CSF 2.0 functions | CIS v8.1 target safeguards | Other frameworks |
|---|---|---|---|---|
| `00-foundation/` | Host, hypervisor, storage layout, segmentation and addressing, plane separation, naming scheme, host backups, amendments to the standards | Identify, Protect | 1.1, 11.1, 11.2, 11.3, 11.4, 11.5, 12.2, 12.4 | |
| `05-service-desk/` | Ticketing system, incident and change records, a ticket for every later lab | Respond | 17.3 | |
| `10-endpoint/` | Windows 11 builds, unattend, imaging, reimage, patching | Protect | 2.1, 3.6, 4.1, 7.3, 7.4 | |
| `20-server/` | Windows Server, AD DS, DNS, DHCP, GPO, patching, AD backup and restore | Protect | 4.1, 5.1, 6.1, 6.2, 7.3, 8.4, 11.2, 11.5 | |
| `25-vulnerability/` | Vulnerability scanning, remediation tracking | Identify, Protect | 7.1, 7.2, 7.5, 7.7 | |
| `30-identity/` | Entra tenant, Connect, hybrid join, Intune | Protect | 4.1, 6.3, 6.5, 6.7, 15.1 | |
| `35-ot/` | OT segment, PLC simulation, Siemens and Allen-Bradley controllers, industrial protocols | Identify, Protect | 12.2 | NIST SP 800-82r3 |
| `40-telemetry/` | Sysmon, WEF, SIEM install, ingest, parsing, filtering | Protect, Detect | 3.10, 8.1, 8.2, 8.3, 8.5, 8.8, 8.9, 8.10, 8.12 | NIST SP 800-137 |
| `50-detection/` | Sigma, rule authoring, false-positive tuning | Detect | 8.11, 13.1 | MITRE ATT&CK |
| `60-emulation/` | Atomic tests, sample detonation, ATT&CK ledger | Identify | none | MITRE ATT&CK, NIST SP 800-53 SC-44 |
| `70-response/` | IR playbooks, triage, containment, eradication, recovery | Respond, Recover | 17.4, 17.7 | NIST SP 800-61r3 |
 
## Framework alignment
 
The layers map to NIST CSF 2.0 functions and CIS Controls v8.1
safeguards. Incident response follows NIST SP 800-61r3, and attack
techniques are tracked with MITRE ATT&CK. The layer table is the target
profile, and each entry's `controls` field records what that entry
demonstrated. This alignment is entirely self-assessed and uncertified.
 
## Current stack
 
As of November 2026:

| Component | Choice | Chosen in |
|---|---|---|
| Lab host | Fedora 45 Workstation, KVM managed through libvirt | In place before the first entry |
| Lab addressing | `10.20.0.0/16`, one `/24` per segment | [LAB-0001](00-foundation/LAB-0001-address-scheme-and-topology/address-scheme-and-topology.md) |
 
## Finding an entry
 
- Match the layer: Find the directory covering the entry in the layer
  table (for example, endpoint telemetry and log forwarding live in
  `40-telemetry/`).
- Check the layer index: Open that layer's README.md to view its
  entries, newest first, with their tier, status, and whether a newer
  entry supersedes them.
- Filter for active runbooks: For repeatable configurations, use entries
  marked `status: complete` and `tier: L2` or `L3` that the layer index
  does not show as superseded.

## Reading an entry
 
Each entry combines an architecture decision record (context, options
considered, decision, and consequences) with the procedure, verification
output, and rollback steps recorded during implementation.
 
Entries open with YAML frontmatter tracking metadata (id, layer, date,
status, tier), relationships (prereqs, supersedes), open or resolved
gaps, redacted state, and mapped controls. conventions.md defines every
field.
 
The tier field (L0–L3) records how far an entry's steps were validated:
 
| Tier | Validation |
|---|---|
| L0 | Architectural decision only. No executable steps. |
| L1 | Commands and outcomes recorded from a single run on the named hosts. Kept as a historical log rather than a repeatable runbook. |
| L2 | Reproducible procedure with pre-checks, a captured baseline, verification against that baseline, and a tested rollback. |
| L3 | L2 procedure with explicit verification stops and captured evidence at every step, used for destructive changes where rollback requires more than restoring a snapshot. |
 
Each procedure was verified only on the lab systems its entry names. The
entry's OBJECTIVE AND SCOPE section states where the procedure applies
and where it does not.
 
Values in square brackets, such as `[USER]` or `[TENANT-ID a3f9]`,
replace values removed before publication. Addresses in `192.0.2.0/24`,
`198.51.100.0/24`, `203.0.113.0/24`, and `2001:db8::/32` replace real
addresses, while private addresses on lab-plane segments appear as
captured. A token with a four-character suffix stands for the same
value wherever it appears. `redacted: true` in an entry's frontmatter
means the entry or its artifacts contain substitutions, and
[`REDACTION.md`](REDACTION.md#substitution-grammar) lists every token.
 
## Host plane & lab plane
 
The labs run as virtual guests on hardware shared with personal data.
 
- Host plane: Physical host machines, local personal devices, and the
  primary home network.
- Lab plane: VMs, virtual networks/switches, and any lab-exclusive
  hardware.
 
The environment is designed to be logically isolated so that no guest
VM can communicate with the host plane beyond its own segment's DHCP
and DNS. The 12.2 exit criterion in
[`00-foundation/README.md`](00-foundation/README.md) sets this
requirement, and the entry that meets it records the test. Lab
documentation intentionally excludes host-plane details. Physical hosts
interact with the lab only through their hypervisor interfaces and
assigned virtual networks.
 
## Guarantees
 
All entries are held to the following four guarantees.
 
### Fidelity
 
Evidence is captured during the live session. Captured output appears
exactly as it was captured. Nothing is tidied, invented, or
reconstructed afterwards.
 
### Disclosure control
 
No published entry exposes credentials, identifiers, or any person other
than myself (included only for attribution). Commands are structured to
exclude sensitive information from their output. A custom redaction
utility substitutes whatever remains, and its output is manually
reviewed before being committed.
 
### Falsifiability
 
Every claim states its basis: direct lab observation, vendor
documentation, or unknown. Claims that rest on documentation alone
remain open gaps until they are observed.
 
### Continuity
 
Corrections are recorded as new entries that supersede earlier ones,
rather than overwriting or modifying existing entries. Prior entries,
including mistakes, remain in the repository.
 
In the event of a conflict, [`GOVERNANCE.md`](GOVERNANCE.md#precedence)
determines which guarantee takes precedence.
 
## License
 
Documentation is licensed under CC BY 4.0; see [`LICENSE`](LICENSE).
Source and configuration are licensed under the MIT License; see
[`LICENSE-CODE`](LICENSE-CODE).
 
## See also
 
- [`RATIONALE.md`, Guarantees](RATIONALE.md#guarantees): how the four
  guarantees fit together.
- [`RATIONALE.md`, Framework
  alignment](RATIONALE.md#framework-alignment): why these frameworks,
  and why each is pinned to a version.
 
