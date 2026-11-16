# writeup-standard

This standard sets what an entry must show, and when it is complete. An
entry is complete once it passes the [definition of
done](#definition-of-done).

## Tiers

An entry's tier records how far its steps were validated:

| Tier | Claims | Chosen when |
|---|---|---|
| L0 | A decision, with nothing repeatable | The output is a decision, a comparison, or a plan |
| L1 | The steps worked once, on the systems named | The steps ran and were verified, but a second person could not follow them without the author |
| L2 | A second person can run the procedure | Scope, pre-checks with abort conditions, a baseline, and a verified rollback were captured during the work |
| L3 | Every step has a verification stop and captured evidence | The change is destructive, its rollback is harder than restoring a snapshot, and every step's stop and output were recorded during the work |

To choose a tier, answer these questions in order, and stop at the first
one that decides it:

1. Did anything repeatable come out of the work? If not, the entry is
   L0.
2. Could a second person run the steps without the author, using the
   scope, pre-checks, baseline, and verified rollback the entry
   captured? If not, the entry is L1.
3. Is the change destructive, with a rollback harder than restoring a
   snapshot, and does every step have an abort criterion, an expected
   output, and its captured output? If so, the entry is L3. Otherwise,
   it is L2.

A change that a snapshot can revert is L2 at most. Raising a forest
functional level can be L3, because undoing it takes real work.

Capture the pre-checks, the baseline, the rollback, and the per-step
evidence for L3 during the work. Do not write any of them afterward to
reach a higher tier ([why](RATIONALE.md#choosing-a-tier)).

## Section contents

Each section must show the following:

| Section | Must show |
|---|---|
| CONTEXT | The state of the environment before the work, the problem the entry addresses, and the prior entries it builds on |
| OPTIONS CONSIDERED | The alternatives weighed, each with the reason it was chosen or rejected, including any option rejected only because the author knew too little about it at the time; at least one rejected option, or "none" and why the work was mechanical |
| DECISION | What was done and the actual reason, with the configuration or command that makes the change |
| CONSEQUENCES | What the change commits to, forecloses, and leaves exposed, including the residual risk and at least one cost |
| OBJECTIVE AND SCOPE | What the procedure applies to, and what it excludes |
| PRE-CHECKS | Each command run to find a reason to stop, with the condition in its output that aborts |
| BASELINE | Captured state before the change |
| PROCEDURE | The steps as they ran; at L3, each step's abort criterion, expected output, and captured output |
| VERIFICATION | Captured state after the change, read against the baseline where the tier includes one; at L1, also what broke |
| RISK AND ROLLBACK | How the change is reversed, with captured output confirming the reversal, or "irreversible" with its cost |
| POST-IMPLEMENTATION REVIEW | Which steps behaved differently than written, and whether the rollback would have held |
| IN PLAIN TERMS | The mechanism, restated without jargon |
| GAPS | Each claim still unverified and each deferred test, with the test that would resolve it |
| REFERENCES | The vendor documentation, framework items, and prior entries the entry depends on |

When a section has nothing to record, write "none" and the reason, so a
reader can tell an empty section from a forgotten one.

## Claims and evidence

These rules implement the Fidelity and Falsifiability guarantees in the
README:

- State each claim's basis where the claim is made: observed in the lab,
  taken from vendor documentation, or unknown.
- Label a claim taken from vendor documentation in the sentence that
  makes it. Each such claim remains an open gap until it is observed.
- Paste captured output as it appeared, cut from the redacted copy.
- State the platform and version wherever syntax or behavior varies
  between them.

## The seven questions

An entry's evidence answers seven questions. The table names the section
that answers each:

| Q | Question | Section |
|---|---|---|
| Q1 | What was true before? | BASELINE |
| Q2 | Where does this apply, and where is it wrong? | OBJECTIVE AND SCOPE |
| Q3 | What would have made me stop? | PRE-CHECKS |
| Q4 | What did I choose, and what did I reject? | OPTIONS CONSIDERED |
| Q5 | What is true now? | VERIFICATION |
| Q6 | What does this expose, and can I undo it? | CONSEQUENCES, RISK AND ROLLBACK |
| Q7 | What do I still not know? | GAPS |

Each tier answers the questions whose sections it includes, as [Entry
structure](conventions.md#entry-structure) shows. An L0 entry answers
Q4 and Q7, plus the first half of Q6 (what the decision exposes) in
CONSEQUENCES.

Q4 records what was rejected and why, which is most of what an entry
teaches.

Answer Q5 with the same command as Q1 wherever the same thing exists
before and after the change. When the lab builds something new, the
baseline measures the environment around it, such as host storage
before a guest build. Either way, the two answers must be readable
against each other.

Q7 lists only what is still unknown. A known mitigation that was skipped
is a decision: record its reason in OPTIONS CONSIDERED and its risk in
CONSEQUENCES.

## Evidence by kind of work

Scope (Q2), options (Q4), and gaps (Q7) are answered the same way for
any kind of work. The baseline, pre-checks, verification, and rollback
depend on what the work changes, and the table below sets out what each
one needs for common kinds of work. Apply every row that matches the
work. When no row matches, answer the seven questions directly.

| Work | Abort check | Evidence | Rollback |
|---|---|---|---|
| Host, hypervisor, and storage | Free storage against the threshold the entry states; whether the change cuts the author's own access to the host; whether the change gives a guest a path to the host plane | Storage use and service state, before and after | A snapshot taken before the change, verified by reading storage against the baseline |
| Group Policy | Existing links and inheritance at the target OU; security filtering; whether the scope includes a domain controller. Abort if the scope is wider than intended | The resultant set of policy for a test account, before and after, plus registry state for policy-backed settings | A GPO backup taken before the edit, and an unlink in preference to a delete, verified by a forced refresh and the same resultant-set query |
| Vulnerability scanning and remediation | Scan targets against the intended scope, and the account a credentialed scan uses. Abort if any target sits outside the lab segments | Findings from the same scan profile, before and after remediation | The remediation's reversal or a snapshot, verified by a rescan with the same profile |
| Directory sync and Conditional Access | Which objects are in sync scope; whether the change affects accounts still needed to sign in | Sync status and object state, before and after, plus a sign-in from a test account | For Conditional Access, report-only mode before enforcement, so rollback is a mode change |
| Detection rules | The rule logic run as a historical query first. Abort if hit volume over the lookback window is unmanageable | Hit count and sample events, before and after, plus one deliberately generated true positive | The rule disabled and kept, so its tuning history survives, verified by its silence on the generated event |
| Attack emulation and response exercises | The isolation that [`HANDLING.md`](HANDLING.md) requires is in place before the test runs | Telemetry the test produced and the detection result | Revert to the pre-test snapshot, and state anything the test left behind |
| Linux host services | Listening services, running services, and whether the change cuts the author's session; before enabling a host firewall, the remote-access rule exists | Listening sockets, service state, and a connection from another host | A snapshot, or deletion of the specific rule added |

## Control tags

Tag an entry only with items it demonstrates:

- A `cis8.1:` tag requires captured output in VERIFICATION showing the
  safeguard's practice in effect on a lab system. Vendor documentation
  alone is not enough for a tag.
- An `attack<major>:` tag requires captured telemetry or a detection
  result for that technique in VERIFICATION.
- A tag covers only the systems the entry names.

## Capture protocol

Build every entry that records work on a lab system from captured
material ([why](RATIONALE.md#capture-protocol)):

1. Before touching anything, copy `templates/entry.md` into the
   directory where the raw log will be written, outside the working
   tree, as [`REDACTION.md`](REDACTION.md#raw-logs) requires. Assign the entry its lab ID.
2. Start the session logger, which writes the raw log, and keep it
   running for the whole session.
3. Paste each command and its output into the entry's `## RAW` block
   during the session, unformatted and unedited. When in doubt, paste
   it: output that was never captured cannot be recovered.
4. Note surprises inline as they happen: unexpected output, warnings,
   and anything that failed the first time. These notes feed Q4, and
   they are the details most easily forgotten.
5. Write the spine during the lab.
6. After the lab, run the redaction utility on the draft entry and the
   raw log, as REDACTION.md sets out. Move the redacted entry into its
   entry folder, and add the redacted log beside it as the session log.
7. Edit `## RAW` down into the entry's sections, working from the
   redacted copy.

## Gaps and supersession

An entry that closes an earlier entry's gap names that entry in its
CONTEXT and REFERENCES. When the finding changes what the earlier entry
concluded, the new entry supersedes it.

## Abandonment record

Complete an abandoned draft as an L0 record of the decision to stop. Its
sections show the following:

| Section | Shows |
|---|---|
| CONTEXT | What the draft set out to do, and how far the work got |
| OPTIONS CONSIDERED | Continuing, restarting in a new entry, and stopping, each with its reason |
| DECISION | The decision to stop and its actual reason |
| CONSEQUENCES | What the work left in the lab, whether it was reverted, and where the session log shows it |
| GAPS | What remains unknown because the work stopped |

When the work continues in a new entry, the new entry names the
abandoned one in its CONTEXT.

## Definition of done

Set `status: complete` once every item below is true for the entry's
tier:

- [ ] The entry conforms to conventions.md.
- [ ] Every output block is pasted output.
- [ ] At L2 and above, the baseline and the post-change evidence are
  readable against each other.
- [ ] At L2 and above, each pre-check names the condition that would
  have aborted the change.
- [ ] OPTIONS CONSIDERED includes a rejected option with its reason, or
  "none" with the reason the work was mechanical.
- [ ] At L2 and above, RISK AND ROLLBACK shows the rollback verified by
  observation, or "irreversible" with its cost.
- [ ] At L3, every procedure step has its abort criterion, expected
  output, and captured output, and the post-implementation review is
  written.
- [ ] Every claim states its basis.
- [ ] At least one consequence is a cost.
- [ ] GAPS lists an open item, or "none" with the reason.
- [ ] Every control tag meets the [control tag](#control-tags) rules.
- [ ] The core mechanism is explained once outside IN PLAIN TERMS. A
  search for its key phrase finds at most one hit outside that section.
- [ ] The entry and its artifacts pass the release check in
  REDACTION.md.
- [ ] When the work produced a raw log, its redacted copy is committed
  as the session log, and the raw log stays outside the working tree.
