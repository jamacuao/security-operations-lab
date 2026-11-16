# conventions

These conventions keep every file in the repository named, placed, and
written the same way. Follow them in every commit.

## Repository layout

The repository is laid out as follows, with entry folders left out:

```
security-operations-lab/
├── README.md
├── GOVERNANCE.md
├── conventions.md
├── REDACTION.md
├── RATIONALE.md
├── HANDLING.md
├── writeup-standard.md
├── LICENSE                    CC BY 4.0, documentation
├── LICENSE-CODE               MIT, source and configuration
├── .gitignore
├── templates/
│   ├── entry.md
│   └── layer-readme.md
├── tools/                     scripts, named in tools/README.md
│   ├── README.md
│   └── hooks/
└── 00-foundation/
    ├── README.md
    └── network-register.md
```

Layer directories, entry folders, and registers are added by the first
commit that needs them. [Layers](#layers) lists every layer, and [File
placement](#file-placement) sets where each file goes.

## Permanent rules

These rules apply for the life of the repository:

1. Each lab ID belongs to exactly one entry and is never reassigned.
2. An entry folder keeps the name it was first committed with.
3. The frontmatter, evidence, and conclusions of a complete entry are
   final. A correction is made in a new entry that supersedes it.
4. A complete entry that outgrows its tier is superseded by a new entry
   at the higher tier.
5. A control tag keeps the framework version it was written with.
6. An entry's frontmatter values appear only in its frontmatter. The one
   exception is a gap slug, which also heads its item in the GAPS
   section. The body may still cite other entries, as [Entry body
   syntax](#entry-body-syntax) describes.
7. Artifacts stay with the entry that produced them.

## Formats

These formats apply to entries and, where a subsection says so, to
layer READMEs and registers.

### Frontmatter

Every entry opens with YAML frontmatter that includes every field in
this example. A field with nothing to record is set to `""` or `[]`
([why](RATIONALE.md#frontmatter)):

```yaml
---
id: LAB-0031
title: Sysmon deployment and configuration tuning via GPO
author: <legal name>
layer: 40-telemetry
date: 2026-10-14
status: complete
tier: L2
redacted: true
prereqs: [LAB-0011, LAB-0022]
supersedes: ""
tags: [sysmon, gpo, telemetry, windows-server]
gaps: [config-reload-under-gpo-refresh]
closes_gaps: []
controls: [cis8.1:8.2]
---
```

The fields also follow these rules:

| Field | Rule |
|---|---|
| `id` | Matches the entry's folder name |
| `author` | Matches the name in `LICENSE-CODE` exactly |
| `layer` | Matches the layer directory the entry sits in |
| `date` | The lab's first day; for an entry with no lab, the day of the decision |
| `status` | `draft` until the entry passes the definition of done in writeup-standard.md, then `complete` |
| `tier` | `L0`, `L1`, `L2`, or `L3` |
| `redacted` | Set by the release check in REDACTION.md |
| `prereqs` | Entries whose results this one relies on |
| `supersedes` | One lab ID at most |
| `tags` | For search only |
| `gaps` | One slug per GAPS item, unique within the entry |
| `closes_gaps` | Each item is `LAB-NNNN/<gap-slug>` |
| `controls` | Always `[]` at L0 |

### Entry structure

An entry body is made up of numbered sections in the order below, with
each section name written exactly as shown:

| Section | Group | L0 | L1 | L2 | L3 |
|---|---|---|---|---|---|
| CONTEXT | Spine | ✓ | ✓ | ✓ | ✓ |
| OPTIONS CONSIDERED | Spine | ✓ | ✓ | ✓ | ✓ |
| DECISION | Spine | ✓ | ✓ | ✓ | ✓ |
| CONSEQUENCES | Spine | ✓ | ✓ | ✓ | ✓ |
| OBJECTIVE AND SCOPE | Procedure | | | ✓ | ✓ |
| PRE-CHECKS | Procedure | | | ✓ | ✓ |
| BASELINE | Procedure | | | ✓ | ✓ |
| PROCEDURE | Procedure | | ✓ | ✓ | ✓ |
| VERIFICATION | Procedure | | ✓ | ✓ | ✓ |
| RISK AND ROLLBACK | Procedure | | | ✓ | ✓ |
| POST-IMPLEMENTATION REVIEW | Procedure | | | | ✓ |
| IN PLAIN TERMS | Closing | ✓ | ✓ | ✓ | ✓ |
| GAPS | Closing | ✓ | ✓ | ✓ | ✓ |
| REFERENCES | Closing | ✓ | ✓ | ✓ | ✓ |

- Delete the sections the entry's tier does not include. Keep every
  section the tier includes, even when there is nothing to record in it.
- Number the remaining sections consecutively.
- Open a draft with an unnumbered `STATUS OF THIS ENTRY` block above
  section 1, and close it with the temporary `## RAW` block after the
  last section. Delete both before setting `status: complete` ([why](RATIONALE.md#raw-heading)).
- In REFERENCES, cite each framework item by its ID and title. Do not
  copy framework text into the repository.

[`writeup-standard.md`](writeup-standard.md#section-contents) sets what
each section must show.

### Entry body syntax

Entry bodies follow the plain-text style of IETF RFCs and UNIX man
pages, inside one code fence ([why](RATIONALE.md#entry-fence)):

| Element | Convention |
|---|---|
| Fence | The whole body sits inside one code fence: a `` ```text `` line after the frontmatter, and a `` ``` `` line at the end of the file |
| Sections | Numbered and uppercase, such as `1.  CONTEXT` |
| Subsections | Numbered under their section, such as `2.1.` |
| Prose | Indented four spaces under a section, and ten under a subsection |
| Commands and captured output | Indented four spaces deeper than the prose they belong to |
| Lists | Each item starts with `o` |
| Gaps | Each GAPS item starts with `o` and its gap slug, on a line of its own |
| Options | Each option in OPTIONS CONSIDERED has a `Case for:` line and a `Why not:` line, and the chosen option ends its heading line with `[CHOSEN]` |
| Decoration | None |
| Markdown headings | None, apart from the temporary `## RAW` block |
| Pagination | None |
| Cross-references | Another entry is cited by its lab ID and its path from the repository root, and an artifact by its path, because links do not render inside the fence |

### Placeholders

Use only these four placeholders to mark unfinished work:

| Placeholder | Marks |
|---|---|
| `[PENDING]` | Prose not yet written |
| `[PASTE]` | A command or output block not yet filled from a real session |
| `<slot>` | A named value in a template, replaced by the value itself |
| `LAB-XXXX` | A lab ID not yet assigned |

`[PENDING]`, `[PASTE]`, and `LAB-XXXX` mark unfinished work anywhere in
a file. A `<slot>` marks unfinished work only in frontmatter and prose,
since captured output often contains angle brackets of its own. Keep
each `<slot>` on a single line; the placeholder check reads one line at
a time. An entry that still contains a placeholder stays at
`status: draft`, and a layer README that contains one is unfinished.
Placeholders quoted in the standards and templates are exempt
([why](RATIONALE.md#placeholders)).

### Frameworks and control tags

The repository uses the frameworks below, at the versions listed
([why](RATIONALE.md#framework-alignment)). Adopting a framework, or moving to a new version of one, is
an amendment; see [`GOVERNANCE.md`](GOVERNANCE.md#extending-the-repository).

| Framework | Version | Tag prefix | Reference |
|---|---|---|---|
| NIST Cybersecurity Framework (CSF) | 2.0 | none | https://doi.org/10.6028/NIST.CSWP.29 |
| CIS Critical Security Controls | v8.1 | `cis8.1:` | https://cisecurity.org/controls/v8-1 |
| CIS Controls v8.1 mapping to NIST CSF 2.0 | v8.1 | none | https://cisecurity.org/insights/white-papers/cis-controls-v8-1-mapping-to-nist-csf-2-0 |
| CIS Controls Assessment Specification | v8.1 | none | https://cas.docs.cisecurity.org/en/latest/ |
| NIST SP 800-61 | Rev. 3 | none | https://doi.org/10.6028/NIST.SP.800-61r3 |
| NIST SP 800-53 | Rev. 5 | none | https://doi.org/10.6028/NIST.SP.800-53r5 |
| NIST SP 800-137 | 2011 | none | https://doi.org/10.6028/NIST.SP.800-137 |
| NIST SP 800-82 | Rev. 3 | none | https://doi.org/10.6028/NIST.SP.800-82r3 |
| MITRE ATT&CK, Enterprise | Major release, carried in each tag | `attack<major>:` | https://attack.mitre.org/resources/versions/ |

The `controls` field accepts two tag forms:

- `cis8.1:<safeguard>` names a CIS safeguard, such as `cis8.1:8.2`.
- `attack<major>:<technique>` names an ATT&CK technique and the major
  release the entry used, such as `attack19:T1059.001` ([why](RATIONALE.md#attck-versions)).

Tag only CIS safeguards and ATT&CK techniques. CSF 2.0 coverage can be
derived from the published CIS mapping, and other frameworks are cited
under REFERENCES.
[`writeup-standard.md`](writeup-standard.md#control-tags) sets the
evidence each tag requires.

### Segment enforcement

A segment's enforcement is recorded as one of these values:

| Value | Enforced by |
|---|---|
| `libvirt:<network>` | A libvirt network on the hypervisor, through the host bridge and its netfilter rules |
| `vlan:<id>` | An 802.1Q tag on a trunk or access port, through a switch that reads it |
| `physical` | A separate physical segment with no shared media |

## Terminology

Use these terms exactly as defined, without synonyms:

| Term | Meaning |
|---|---|
| Entry | The Markdown file that records one lab or one decision, inside its folder `LAB-NNNN-<slug>/` |
| Spine | The four sections every entry opens with: CONTEXT, OPTIONS CONSIDERED, DECISION, CONSEQUENCES |
| Procedure sections | The sections an entry includes according to its tier |
| Closing sections | The three sections every entry ends with: IN PLAIN TERMS, GAPS, REFERENCES |
| Tier | How far an entry's steps were validated, from L0 to L3, recorded in `tier` and defined in writeup-standard.md |
| Layer | A top-level directory `NN-<name>/` that groups entries by the dependency level of what they produce |
| Host plane | The physical host machines, local personal devices, and the primary home network |
| Lab plane | The guest VMs, virtual networks and switches, and any lab-exclusive hardware |
| Artifact | Any file in an entry's folder other than the entry itself, such as the session log or a screenshot in `assets/` |
| Script | Code committed to the repository, as an artifact or in `tools/` |
| Raw log | An unredacted capture, kept outside the working tree |
| Session log | The redacted copy of a raw log, committed in the entry folder as `<slug>-session.log` |
| Register | A file that records current lab state and nothing else, listed under Registers |
| Control tag | A `controls` value naming a framework item the entry demonstrates, such as `cis8.1:8.2` |
| Placeholder | A mark of unfinished work, one of the four listed under Placeholders |
| Permanent rule | A rule listed under Permanent rules |
| Standard | A binding document, as listed in GOVERNANCE.md |
| Gap | An item in an entry's GAPS section, with its slug in the entry's `gaps` field |
| Superseded | The state of an entry that a complete entry names in `supersedes` |

## Document style

- Hard-wrap all prose at 72 columns, with the line breaks written into
  the file. This includes prose inside an entry's fence. Frontmatter,
  tables, commands, captured output, URLs, and other code blocks are
  exempt, and a line that contains a link is measured with the link
  target removed ([why](RATIONALE.md#wrap-exemptions)).
- Every Markdown file outside the entry folders is a reference document,
  apart from `templates/entry.md`, which follows the [entry body
  syntax](#entry-body-syntax).
- Use Markdown headings and tables in reference documents, with
  sentence-case headings and table headers.
- In reference documents, link between repository files with relative
  Markdown links.
- Write template guidance as plain prose at the content indentation, and
  delete it as each section is filled. Guidance left in a finished
  document is a defect.

### Writing style

Reference documents follow these rules:

- Open each document and each section with what is true or what to do.
- Write instructions in the imperative, and state rules and facts
  positively. Keep "do not" for real prohibitions.
- Keep one idea to a sentence, and one person and voice throughout a
  document.
- Spell words in full, such as "do not" and "configuration".
- Write dates as absolute dates, such as "as of October 2026".
- End directory names with a slash, such as `assets/`.
- Introduce a command block with a sentence that ends in a colon. Prefix
  each command with its prompt: `$` for a regular user and `#` for root
  on Linux, and `PS>` for PowerShell.
- Use a GitHub alert, `> [!NOTE]` or `> [!WARNING]`, only for
  information a reader would otherwise miss, or for a step that is hard
  to undo.
- Leave out what an example or a table already shows.

### Naming tools in standards

A standard names what is being built and where sensitive data comes
from. It names a tool only when replacing that tool would mean rewriting
the standard anyway, as with git and the POSIX utilities. Refer to every
other tool or script by its role, such as "the redaction utility," and
name it in [`tools/README.md`](tools/README.md) ([why](RATIONALE.md#naming-tools-in-standards)).

## Entry identity

- Lab IDs take the form `LAB-NNNN`, zero-padded to four digits, in one
  sequence shared by every layer.
- Assign the ID when creating the entry from the template. For lab work,
  that is the first step of the capture protocol, before the lab begins,
  so the sequence records the order in which work started ([why](RATIONALE.md#lab-ids)).
- An ID assigned to work that was abandoned before its first commit is
  retired, and never reused.
- Name the slug for the action performed, such as
  `sysprep-generalize-capture`. Slugs are lowercase and
  hyphen-separated, and contain no dates or version numbers.
- Name the entry folder `LAB-NNNN-<slug>/`, and the entry file inside it
  `<slug>.md`:

```
40-telemetry/
└── LAB-0031-sysmon-gpo-deploy/
    ├── sysmon-gpo-deploy.md
    ├── sysmon-gpo-deploy-session.log
    └── assets/
```

## Lifecycle state

An entry's state is derived from frontmatter and stored nowhere else:

- An entry is a draft or complete according to its own `status`.
- An entry is superseded once a complete entry names it in `supersedes`.
  The superseded entry's `status` stays as it was ([why](RATIONALE.md#supersession)).
- A gap stays open until a complete entry lists it in `closes_gaps` as
  `LAB-NNNN/<gap-slug>`. The entry that recorded the gap keeps its slug.
- An abandoned draft is completed as an abandonment record at
  `tier: L0`, as writeup-standard.md sets out. It keeps its ID and
  folder name. A draft whose slug proves wrong is abandoned the same
  way, and the work continues in a new entry.

## Naming and addressing

Every entry names hosts, domains, segments, and layers the same way.

### Host names

Keep host names to 15 characters or fewer. That is the NetBIOS limit:
Windows truncates a longer name, and the truncated name no longer
matches its DNS record.

Each name is a role prefix followed by a two-digit instance number, such
as `DC01` or `SIEM01`. The entry that plans or adds a role's first host
records the role's prefix in the network register, and the prefix is
permanent from then on.

### AD domain name

Name the Active Directory domain as a subdomain of a domain the author
owns, such as `lab.example.com`. Avoid `.local`, which is reserved for
multicast DNS ([why](RATIONALE.md#ad-domain-name)).

### Segments

A segment is one broadcast domain with one address range and one
security posture. The repository names and addresses the network by
segment.

Segment IDs come from the 802.1Q range, 1 to 4094, even where no VLAN
tag exists. Record a segment's enforcement apart from its identity, as
one of the values under [Segment enforcement](#segment-enforcement), and record only
enforcement that is in effect. When enforcement changes, the segment
keeps its ID, range, and register row, and an entry records the change
([why](RATIONALE.md#segments)).

Choose the lab's RFC 1918 range so that it cannot overlap any range used
on the home network.

### Layers

Name layer directories `NN-<name>/`. Prefixes step by 10, so a new layer
can take a number between two others, as `25-vulnerability/` does. The
layers are:

- `00-foundation/`
- `05-service-desk/`
- `10-endpoint/`
- `20-server/`
- `25-vulnerability/`
- `30-identity/`
- `35-ot/`
- `40-telemetry/`
- `50-detection/`
- `60-emulation/`
- `70-response/`

## Registers

When an entry changes the state that a register records, the same entry
updates the register.

`00-foundation/network-register.md` records:

- the RFC 1918 range in use, and the rule that derives a segment's range
  from its ID;
- each segment's ID, name, range, and enforcement;
- which segment is isolated, with no route to the home network or the
  WAN;
- each role's host name prefix, and each host's name, role, segment, and
  address;
- a plain-text diagram of the segments and hosts.

`60-emulation/attack-ledger.md` has one row for each ATT&CK technique
tested. Each row shows the technique's control tag, the latest entry
that tested it, whether that test was detected, and the entry that built
the detection.

## File placement

Every entry is a folder, even before it has artifacts. Place files as
follows:

- the entry file and its session log at the root of the entry folder;
- screenshots, exports, and other artifacts in `assets/`;
- repository-wide scripts, such as the redaction utility, in `tools/`.

The following rules cover the remaining cases:

- To reuse another entry's artifact unchanged, cite it by its path. To
  change it, commit the changed copy in the new entry's folder.
- Move a script to its own repository once it is used outside this one,
  or sooner when entries keep changing it, as with a configuration
  management codebase. Entries then cite the commit they ran ([why](RATIONALE.md#script-graduation)).
- Identify each disk image or installer by file name, version, and
  SHA-256 hash, and keep the file itself out of the repository.
- A layer directory contains its README, its entry folders, and its
  register, if it has one. Commit the layer README when work on the
  layer begins.
- File each entry in the layer of what it produces, even when the work
  spans several layers. An amendment changes a standard, so it goes in
  `00-foundation/`. A sample detonation goes in `60-emulation/`, where
  its result is recorded in the ATT&CK ledger.
- When one piece of work produces both a test result and a detection
  rule, record each in its own entry, with the test in `60-emulation/`
  and the rule in `50-detection/`. The rule's entry cites the test entry
  ([why](RATIONALE.md#detonation-filing)).
- Write layer exit criteria as functional requirements. Where a CIS v8.1
  safeguard applies, write the criterion as that safeguard, such as "8.9
  Centralize Audit Logs."
- Give each layer README an index of its entries, newest first, and a
  list of their open gaps. Both are views of the entries' frontmatter,
  including which entries are superseded and by which entry. When a view
  and an entry disagree, correct the view.

## License identifiers

Start every source file with an SPDX identifier, written in the comment
syntax of its language:

```
# SPDX-License-Identifier: MIT
```

A source file is any file licensed under `LICENSE-CODE`, such as a
script, a configuration file, or a rule file. In an executable script,
put the interpreter line first and the identifier second, because a
shebang must start at byte zero. Markdown documentation falls under
`LICENSE` (CC BY 4.0) and has no identifier ([why](RATIONALE.md#license-identifiers)).

## Commits

The commit history is also the timeline of the lab.

### When to commit

Change at most one entry per commit ([why](RATIONALE.md#one-entry-per-commit)). A draft and its
completion may be separate commits. Commit an amendment entry together
with the standard it changes.

### Subject

Write commit subjects in the [Conventional Commits
1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) format:

```
<type>(<scope>): <what changed>
```

- Keep the subject to 50 characters or fewer, with no trailing period.
- Use the imperative mood: "add," "replace," "document."
- Write in lowercase after the colon. Lab IDs and file names keep their
  case.
- Describe the change itself, such as
  `replace README placeholder with entry format`. A subject like
  `improve README` describes only an intent.
- Use the layer directory as the scope of an entry commit. Leave the
  scope out of a commit that is not tied to one layer, such as an
  editorial change to a standard.
- Include the lab ID in an entry commit:

```
docs(20-server): add LAB-0014 forest promotion
```

Choose the type from this table:

| Type | Used for |
|---|---|
| `docs` | An entry, or a change in meaning to a document |
| `feat` | A new script in `tools/`, or an entry that delivers a new script, detection rule, or playbook |
| `fix` | Correcting something wrong in a document or a script |
| `refactor` | Restructuring or rewording that keeps the meaning |
| `chore` | `.gitignore`, licensing, and other repository housekeeping |

### Body

- Separate the body from the subject with a blank line, and wrap it at
  72 columns.
- Give each conceptual change its own paragraph, even when it spans
  several files.
- State what changed and why. The diff shows how.
- State known gaps as current fact, with no promise of future work:

```
yes:  conventions.md and HANDLING.md are linked but do not exist yet.
no:   conventions.md and HANDLING.md to follow.
```

### Worked example

The subject below is exactly 50 characters. The entry commits its GPO
startup script, so the type is `feat`:

```
feat(40-telemetry): add LAB-0031 sysmon gpo deploy

Document the decision to deploy by GPO startup script with a
scheduled task for configuration refresh, and the three
alternatives rejected.

Record the procedure at L2 with pre-checks, verification, and
risk and rollback.

Configuration reload behavior under GPO refresh is asserted
from vendor documentation and was not observed. The entry
records it as an open gap.
```
