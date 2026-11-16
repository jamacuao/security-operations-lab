---
id: LAB-XXXX
title: <title>
author: <legal name>
layer: <NN-name>
date: <YYYY-MM-DD>
status: draft
tier: <L0|L1|L2|L3>
redacted: <true|false>
prereqs: []
supersedes: ""
tags: []
gaps: []
closes_gaps: []
controls: []
---

```text
STATUS OF THIS ENTRY

    Draft. The blocks marked [PASTE] still wait for real output, the
    prose marked [PENDING] is unwritten, and every claim below is
    unverified.


TEMPLATE NOTES

    Delete this section, and delete the guidance paragraph in each
    section below as you fill that section.

    Sections 5 through 11 are procedure sections. Keep the ones your
    tier includes, as the entry structure in conventions.md shows,
    delete the rest, and renumber the remaining sections
    consecutively.

    Delete the STATUS OF THIS ENTRY block and the RAW block as the
    last edit, then check the entry against the definition of done in
    writeup-standard.md.


1.  CONTEXT

    [PENDING]

    The state of the environment before this lab, the problem this
    entry addresses, and the prior entries it builds on.


2.  OPTIONS CONSIDERED

    2.1.  <option>

          Case for:  <why this was a candidate>
          Why not:   <what ruled it out>

    2.2.  <option>                                        [CHOSEN]

          Case for:  <what it buys>
          Why not:   <what it costs, accepted>

    Include any option you rejected only because you knew too little
    about it at the time, and say so. Where the work had no fork,
    write "none" and say why it was mechanical.


3.  DECISION

    [PENDING]

    What was done and the actual reason, with the configuration or
    command that makes the change.


4.  CONSEQUENCES

    o  <what this now commits to>

    o  <what this forecloses>

    o  <what it leaves exposed, and where that is handled>

    Name at least one cost, and state the residual risk. A known
    mitigation not applied is a decision: its reason goes in section
    2 and its risk goes here.


5.  OBJECTIVE AND SCOPE

    5.1.  Objective

          [PENDING]

    5.2.  Applies to

          o  <condition>

    5.3.  Does not apply to

          o  <condition, and where it is handled instead>


6.  PRE-CHECKS

    6.1.  <question the check answers>

              [PASTE]

          Abort if <condition>. <why that matters>


7.  BASELINE

        [PASTE]

    Output establishing the state described in section 1, cut from
    the redacted copy.


8.  PROCEDURE

    8.1.  <step>

    8.2.  <step>

              [PASTE]

    At L3, give every step its abort criterion, its expected output,
    and the output it actually produced.


9.  VERIFICATION

    9.1.  <question the check answers>

              [PASTE]

          <expected result, and what a wrong result indicates>

    At L1, record what broke here.


10. RISK AND ROLLBACK

    10.1. <reversal step>

    10.2. Confirm reversal.

              [PASTE]

    Where the change is irreversible, write "irreversible" and state
    what that costs.


11. POST-IMPLEMENTATION REVIEW

    [PENDING]

    Steps that did not behave as written, and whether the rollback
    would have held.


12. IN PLAIN TERMS

    [PENDING]

    The mechanism without jargon, for a reader who has not touched it
    in a year.


13. GAPS

    o  <gap-slug>

       <unverified claim or deferred test, and what would resolve it>

    Copy each slug into the frontmatter gaps list. Where there is
    nothing to record, write "none" and why.


14. REFERENCES

    o  <vendor documentation>

    o  <framework item, cited by ID and title>

    o  <prior entry, by lab ID and path from the repository root>

    Where there is nothing to cite, write "none" and why.


## RAW

Paste every command and its output here as you go, unformatted and
unedited, per the capture protocol in writeup-standard.md. Note
surprises inline as you hit them.
```
