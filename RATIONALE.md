# RATIONALE

RATIONALE.md explains why the rules in the standards and the statements
in the README are what they are. Nothing in it is binding. Each rule
links here from the place it is stated.

## README

### Guarantees

Each guarantee limits the others.

Fidelity requires output to be published as captured. On its own, that
would publish every identifier in a capture. Disclosure control removes
those identifiers, and Fidelity keeps the removal from turning into
tidying or rewriting. Substitution under REDACTION.md satisfies both:
output changes only through declared, mechanical rules that a reader can
look up.

Falsifiability shows the basis of each claim, so a reader can decide how
much weight to give it.

Continuity keeps mistakes in the record. A visible correction is part of
what the repository demonstrates.

### Framework alignment

CSF 2.0 describes outcomes at the level an organization reports against.
CIS v8.1 describes concrete safeguards that a lab can show working. CIS
publishes a mapping between the two, which is why entries tag only CIS
safeguards: CSF coverage can be derived from the tags.

The other frameworks each cover one part of the lab:

- NIST SP 800-61r3 is a CSF 2.0 community profile for incident response.
- MITRE ATT&CK names the adversary techniques that detection and
  emulation work is described in.
- NIST SP 800-137 covers continuous monitoring, which the telemetry
  layer works toward.
- NIST SP 800-82r3 covers the security of operational technology.
- NIST SP 800-53 supplies the control names in the mapping table in
  GOVERNANCE.md.

Each framework is pinned to a version. A safeguard ID only has meaning
within its version.

The alignment is self-assessed, and a tag records only what one entry
showed.

## conventions.md

### Wrap exemptions

Entries borrow the look of IETF RFCs and UNIX man pages without
following either format exactly. RFC 7994 limits a line to 72
characters, indentation included, and requires a form feed at least
every 58 lines. The repository keeps the line limit. It drops the
pagination, because a browser and a terminal both scroll.

Some lines cannot be wrapped without damage:

- A Markdown table stops rendering when a row is split.
- Frontmatter is data that a parser reads.
- Reflowing captured output would change it, which Fidelity forbids.
- A link target cannot be shortened. A relative link from a reference
  document to an entry runs to 60 to 85 characters, so each line is
  measured with its targets removed.

Plain text has no markup for an exempt line, so inside an entry a line
is marked by its position instead. A command sits four spaces deeper
than the prose it belongs to. That works at any depth, because section
prose starts at four spaces and subsection prose at ten.

The entry's fence does not exempt its prose. Markdown treats the fence
as a code block, but the prose inside it is still wrapped.

### Entry fence

Without a fence, GitHub renders an indented plain-text body as Markdown.
Numbered sections become an ordered list, indented subsection text
becomes code blocks, and angle brackets in prose vanish as HTML tags.

A `text` fence shows the body exactly as written, in a browser and in a
terminal. It has one cost. Links do not render inside it, so an entry
cites other entries and their artifacts by path.

### RAW heading

The `## RAW` marker is meant to look out of place. It uses Markdown
heading syntax inside a plain-text body so that nobody mistakes the
block for one of the entry's sections. The block collects unedited
capture during the lab and is deleted before the entry is complete.

### Placeholders

`[PENDING]` and `[PASTE]` use the same brackets, so one search finds
both. They stay separate so that missing output can be found apart from
unwritten prose. `[PASTE]` also marks unfilled commands, wherever they
appear.

Lab IDs use `LAB-XXXX` because it stays visible in a rendered table. An
angle-bracket slot there would be read as an HTML tag and hidden.

Captured output often contains angle brackets of its own: XML
configuration, redirection operators, usage text. A `<slot>` therefore
counts as unfinished work only in frontmatter and prose, and an entry
that quotes a configuration file can still pass the check.

The standards and templates quote placeholders on purpose, so they are
exempt.

### Naming tools in standards

A standard states what must be true. A script is only one way to make it
true, and a standard that named the script would go out of date as soon
as the script was renamed, rewritten, or replaced.

Git and the POSIX utilities are the exception. Replacing them would mean
rewriting the standards anyway.

### Lab IDs

Layer directories group entries by what they produce. Lab IDs record the
order in which work started. A layer folder that holds LAB-0009 next to
LAB-0044 is therefore expected.

IDs are assigned when an entry is created and never reserved in advance,
because the order of future work is not known until it starts.

An ID is never reused, even for work abandoned before its first commit.
It may already appear in raw logs outside the working tree, and reuse
would leave one ID pointing at two pieces of work.

### AD domain name

`.local` is reserved for multicast DNS. An AD domain under it causes
name resolution failures on macOS and Linux clients that honor the
reservation.

`30-identity/` also needs a domain the author owns, because Entra
requires one to verify a custom domain. Choosing the right name at the
start avoids a domain rename later.

### Segments

A segment's enforcement changes as the lab gains hardware. Its identity
should not. A segment enforced by a libvirt network today and by a
tagged switch port later keeps its ID, range, and register row, and
moving it takes a one-column change in the register plus an entry that
records the change.

Segment IDs come from the 802.1Q range so that the same number can
become the VLAN tag once a switch reads tags. Until then the tag
enforces nothing, which is why the register records only enforcement
that is in effect.

### Script graduation

An artifact supports one entry. A script stays with that entry until
something outside this repository uses it, because moving scripts out
earlier would create repositories for code nobody reuses.

Code that entries keep changing is the exception, and a configuration
management codebase is the usual case. If every entry that changed it
committed its own copy, its history would be scattered across entry
folders. In its own repository the code keeps one history, and the
commit an entry cites pins exactly what that entry ran.

### Detonation filing

A detonation that also produces a detection rule becomes two entries,
because the test and the rule differ in almost everything the standards
track:

- The ATT&CK ledger already keeps the entry that tested a technique
  apart from the entry that built its detection.
- The two need different evidence under writeup-standard.md.
- They often reach different tiers. A detonation is usually a single-run
  L1 record, while a rule with a disable-and-keep rollback can be L2.
- Rules are tuned and replaced more often than tests. A separate entry
  lets a rule be superseded without superseding the test.

### Frontmatter

Every field is always present. A missing field is therefore always an
error, and every entry parses the same way.

An entry's own values appear only in frontmatter. A second copy in the
body would drift out of date the first time a field changed.

The `author` field gives a reader a name to compare with the key that
signed the entry's commit. The email address stays out of the files; the
public key already includes it.

### Supersession

Changing a superseded entry's `status` would mean editing the
frontmatter of a complete entry, which the permanent rules forbid. The
state is derived instead, from the newer entry's `supersedes` field.
That keeps the record append-only, and the layer index still shows
readers which entries are superseded.

Gap state works the same way, through `closes_gaps`.

### ATT&CK versions

ATT&CK publishes a major release about twice a year. A release can
deprecate or revoke techniques and replace them under new IDs, so a
technique ID recorded without its release may later point at a different
technique, or at nothing.

Including the major release in each tag means the tag can always be
looked up. The CIS version is part of the tag prefix for the same
reason.

### License identifiers

Every Markdown file in the repository is documentation under `LICENSE`.
A per-file identifier would repeat that in every document. The usual `#`
comment form would also render as a heading, and an HTML comment would
only hide the repetition.

### One entry per commit

The commit history doubles as the lab's timeline. A commit that batches
several entries hides the order in which they were done.

Drafts are committed with `status: draft`, which lets a draft and its
completion be separate commits.

## GOVERNANCE.md

### Disclosure precedence

Disclosure control takes precedence because it is the only guarantee
whose failure a later entry cannot repair.

A fabricated block can be corrected by an entry that supersedes it. A
published credential cannot be recalled. It stays reachable by object
SHA after the commit that removed it, and even after history is
rewritten.

### Amendment bar

Only a change in meaning needs an amendment entry. Recording every typo
fix as a decision would bury the decisions that matter.

The process starts with the first committed entry. Until then there is
no settled rule to change.

Amendment entries share the main lab ID sequence. An amendment that a
lab forced therefore sits right next to that lab in the timeline.

## writeup-standard.md

### Choosing a tier

A tier is a claim about how far an entry's steps were validated, and
readers act on it. They treat an L2 or L3 entry as a procedure they can
follow.

Pre-checks and a rollback written after the work describe what the
author thinks should have been checked. That is rarely what would
actually have stopped the change. An L1 entry that admits it records a
single run is more useful to a reader than an L2 entry rebuilt from
memory.

A tier set too low is easy to fix later: a fuller entry at a higher tier
supersedes it. A tier set too high cannot be fixed the same way, since
the entry has already published pre-checks and a rollback that never
ran.

L3 should stay rare. A few L3 entries show the tier was chosen with
care. Dozens would suggest it was applied by default.

### Capture protocol

Reasoning written up after the work comes out cleaner than the reasoning
the author had at the time. The spine is written during the lab so that
it records the real uncertainty, including the options the author was
unsure about.

The draft entry starts outside the working tree because its `## RAW`
block collects raw output. It enters the tree only after redaction. No
raw value ever sits where git or a review copy could pick it up.

The session logger records the whole session. Terminal scrollback loses
whatever the author forgot to paste.

## REDACTION.md

### Capture control

Scrapers index public repositories continuously. A pushed value can be
harvested within minutes, long before a later commit or a support
request removes it.

Controls that keep a value out of the capture in the first place are the
only ones that do not depend on acting faster than the scrapers.

### Redacted field

A frontmatter field appears at the top of the rendered entry and can be
found with one search. It also makes the author decide, once per entry,
whether anything was substituted.

The field matters most for addresses. A bracketed token is obviously a
substitution, but `203.0.113.5` looks like a real address.

Substitution is deterministic because replacing every value with the
same constant would hide which value is which.

### Host plane

The host machines hold the author's own data and sit on the home
network.

Substitution hides a value but keeps its shape. A documentation address
or a suffixed token can still be matched across entries to build a
picture of the author's setup. Host-plane values are therefore kept out
of the record entirely, and the published lab describes only the lab
plane.
