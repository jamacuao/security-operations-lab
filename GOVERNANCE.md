# GOVERNANCE

GOVERNANCE.md sets which document owns each rule, which document takes
precedence when two disagree, and how a rule is changed. The standards
are GOVERNANCE.md, conventions.md, writeup-standard.md, REDACTION.md,
and HANDLING.md. Every other document in the repository, entries
included, must agree with them.

## Document map

Each document answers one question:

| Document | Answers | Never contains |
|---|---|---|
| [`README.md`](README.md) | What the repository is, what it demonstrates, and where to start | Rules; it describes the guarantees, and the standards implement them |
| [`GOVERNANCE.md`](GOVERNANCE.md) | Which document owns each rule, which wins in a conflict, and how a rule changes | Rules about files |
| [`conventions.md`](conventions.md) | What every file is named, where it lives, and how it is written | Rules about what an entry must show |
| [`writeup-standard.md`](writeup-standard.md) | What an entry must show, and when it is complete | Syntax or layout |
| [`REDACTION.md`](REDACTION.md) | What may be published, and how captured output is made safe | Entry structure |
| [`HANDLING.md`](HANDLING.md) | How work with live samples is isolated | Anything outside live-sample work |
| [`RATIONALE.md`](RATIONALE.md) | Why each rule is shaped as it is | Anything binding |
| `templates/` | Starting files that instantiate the standards | Rules not stated in a standard |
| `tools/` | Which scripts apply or check the standards, and how to install and run them | Rules not stated in a standard |
| Layer READMEs | Each layer's exit criteria, entry index, and open gaps | Rules; the index and the gap list are views of frontmatter |
| `00-foundation/network-register.md` | Current network state | Rules or history |
| `60-emulation/attack-ledger.md` | Which ATT&CK techniques were tested, and the latest result for each | Rules or history |
| `.gitignore` | Which files git ignores | Rules not stated in a standard |

## Ownership

- Each rule has exactly one owner: the document whose question the rule
  answers. Other documents link to the rule rather than restating it.
- conventions.md owns every rule about how a file is named, where it
  lives, and how it is written. This includes rules that take judgment
  to apply.
- writeup-standard.md owns every rule about what an entry must show. For
  example, the rule that every entry includes a GAPS section belongs to
  conventions.md, while the rule that GAPS must name what is still
  unknown belongs to writeup-standard.md.
- REDACTION.md owns every rule about what may be published, in any file.
  conventions.md defines the `redacted` field, and REDACTION.md decides
  its value.
- RATIONALE.md explains the reasoning behind the rules. A standard gives
  a short reason inline only where a rule would otherwise surprise the
  reader, and links to the matching RATIONALE.md section where the rule
  is stated.

## Precedence

1. A standard takes precedence over every other document. When another
   document disagrees with a standard, correct the other document.
2. When two standards disagree, the standard that owns the rule takes
   precedence, as the document map sets out.
3. Resolve the disagreement by amending whichever standard is wrong. No
   entry or other document can grant itself an exception to a standard.
4. When two of the README's guarantees conflict, Disclosure control
   takes precedence ([why](RATIONALE.md#disclosure-precedence)).

## Amendment

An amendment is any change to what a standard means: it permits
something that was forbidden, or forbids something that was permitted.
Changing the document map is also an amendment. A change that keeps the
meaning, such as fixing a typo, rewording a rule, or reorganizing a
document, is editorial and needs no entry ([why](RATIONALE.md#amendment-bar)).

1. Record each amendment as an L0 entry, filed and committed as
   conventions.md sets out.
2. Only the standards are amended. When an entry changes a rule or the
   lab state that another document reflects, such as the README's
   current stack or the network register, update that document in the
   same commit as the entry. Editorial changes to other documents can be
   committed at any time.
3. The amendment process starts with the first committed entry. Before
   then, changes to the standards are drafting and need no entry.

## Extending the repository

New kinds of work, such as a migration or a configuration management
run, need no new rules. File, tier, and check them like any other entry.

Adding to the structure of the repository is an amendment. The table
lists what each kind of addition changes:

| To add | Amend | Also update |
|---|---|---|
| A layer | conventions.md, Layers | The README layer table, and a layer README from the template |
| A register | conventions.md, Registers, and the document map above | The new register file |
| A framework, or a new version of one | conventions.md, Frameworks and control tags | The README layer table and the exit criteria in the affected layer READMEs, when the targets change |
| A tag form | conventions.md, Frameworks and control tags, and writeup-standard.md, Control tags | none |
| An evidence row for a kind of work | writeup-standard.md, Evidence by kind of work | none |
| An entry section | conventions.md, Entry structure, and writeup-standard.md, Section contents | `templates/entry.md` |
| A segment enforcement value | conventions.md, Segment enforcement | none |

Three kinds of addition need no amendment:

- a new host role prefix, which is recorded in the network register;
- a new tool that implements an existing rule, which is named in
  `tools/README.md`;
- a new value in an entry's `tags` field, which is free text.

## Framework mapping

The table records what each standard documents. Only an entry can show
that a practice works. Practices that are not listed map to no framework
item.

| Standard | Practice | CIS v8.1 | CSF 2.0 | NIST SP 800-53 Rev. 5 | Coverage |
|---|---|---|---|---|---|
| GOVERNANCE.md | Document map, ownership, and precedence | none | GV.PO-01 | none | Documented; governs the repository's own standards |
| GOVERNANCE.md | Amendment by recorded decision | none | GV.PO-02 | none | Documented; each amendment is an L0 entry in `00-foundation/` |
| conventions.md | Segments, enforcement, and the isolated segment | 12.2 Establish and Maintain a Secure Network Architecture | PR.IR-01 | SC-7 Boundary Protection | Documented; current values are in the network register |
| conventions.md | Host naming and the network register | 1.1 Establish and Maintain Detailed Enterprise Asset Inventory; 12.4 Establish and Maintain Architecture Diagram(s) | ID.AM-01, ID.AM-03 | CM-8 System Component Inventory | Partial: records hosts, roles, segments, addresses, and a diagram; no owner fields |
| writeup-standard.md | Pre-checks, baseline, procedure, and rollback | 4.1 Establish and Maintain a Secure Configuration Process | PR.PS-01 | CM-3 Configuration Change Control; CM-4 Impact Analyses | Documented; shown per entry at L2 and above |
| writeup-standard.md | Verification read against the baseline | none | PR.PS-01 | CM-4(2) Verification of Controls | Documented; shown per entry at L2 and above |
| writeup-standard.md | Post-implementation review and gaps | none | ID.IM-03 | none | Documented; the review is required at L3 only |
| REDACTION.md | Classes A, B, and C, and the host plane | 3.7 Establish and Maintain a Data Classification Scheme | ID.AM-07 | AC-22 Publicly Accessible Content | Documented; covers published material only |
| REDACTION.md | Capture-side controls, raw logs, the redaction utility, the release check, and the push gate | none | PR.DS-01 | AC-22 Publicly Accessible Content | Documented; applied to every entry before it is committed and pushed |
| REDACTION.md | On exposure | 17.4 Establish and Maintain an Incident Response Process | RS.MA-01 | IA-5 Authenticator Management; IR-4 Incident Handling | Documented; unexercised |
