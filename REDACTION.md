# REDACTION

This standard sets what may be published, and how captured output is
made safe before it is published. It covers every file committed to the
repository.

## Order of controls

A value pushed to a public repository cannot be fully withdrawn. A later
commit that removes it still leaves it reachable by object SHA, in every
fork and clone, and in the hosting service's cached views ([why](RATIONALE.md#capture-control)).

Apply the controls in this order:

1. Shape each command so its output leaves the sensitive value out.
2. Write the raw log outside the working tree.
3. Run the redaction utility before anything enters the working tree.
4. Read the outgoing commits by eye before pushing.
5. Let the pre-push hook check the push.

Steps 3 to 5 are backstops for what the first two steps miss. Do not
rely on them alone: substitution cannot undo a Class A value that has
already reached the raw log, and a by-eye read misses more as a session
grows longer.

## Classes

### Class A: credentials

Class A values are credentials or credential material. Keep them out of
every capture:

| Value | Typical source |
|---|---|
| Passwords in plaintext | `net user`, `New-LocalUser -Password`, `--password` flags, `unattend.xml` |
| Private keys, PFX and PEM contents | Key files, certificate export |
| API keys and bearer tokens | SIEM setup, Graph calls, any `Authorization:` header |
| JWTs | Entra sign-in, Graph, any `eyJ`-prefixed blob |
| Agent enrollment tokens | Wazuh agent auth, Elastic enrollment, Splunk HEC |
| Client secrets | Entra app registration |
| NTLM hashes, Kerberos tickets, DPAPI blobs | Credential-access work in `50-detection/` through `70-response/` |
| Retail Windows product keys | `slmgr /dli`, `unattend.xml` |

> [!WARNING]
> Substitution cannot repair a Class A capture, because the value
> reached the raw log on disk before the redaction utility ran. If a
> Class A value reaches a raw log, rotate the secret. If it was also
> pushed, follow [On exposure](#on-exposure).

Lab hashes count as Class A, because a lab hash is still a working hash.
The `[NTLM-HASH]` token tells a reader everything the hash itself would.

Generic KMS client setup keys are published by Microsoft, so they are
kept. An entry that shows one notes inline that the key was kept on
purpose.

### Class B: identifiers

Substitute Class B values. Each one ties the repository to real
infrastructure or to a real person. When a value belongs to the host
plane, the [Host plane](#host-plane) rules below apply instead:

| Value | Note |
|---|---|
| Entra tenant, subscription, application, and object IDs | Enables targeted enumeration of a tenant whose configuration is documented here |
| Public IPv4 addresses | Any public address outside the host plane, such as a remote server a guest contacted |
| Global unicast IPv6 | Any global address outside the host plane; a SLAAC EUI-64 address also embeds the MAC |
| MAC addresses of lab-plane physical interfaces | Persistent hardware identifier |
| Wi-Fi SSID and BSSID | A BSSID resolves to a street address through public wardriving databases |
| Hardware and chassis serial numbers | `dmidecode`, `Win32_BIOS`, Cisco `show version`; Cisco serials carry licensing and support entitlement |
| Real OS username and hostname | Leaks through the shell prompt on every line |
| Personal domain names | The domain is already public; the exposure is binding it to a documented tenant |
| Physical address, phone, legal name, email address | Any source, apart from the author's name where Class C keeps it |

The username in the shell prompt is the most frequent leak in this list.
Generic secret scanners miss it, so the redaction utility's literal list
covers it.

### Class C: kept values

Keep Class C values as captured. They identify nothing outside the lab,
and substituting them would strip the entry of meaning:

- RFC 1918 addresses on lab-plane segments, IPv4 link-local, and
  loopback;
- IPv6 link-local (`fe80::/10`) and unique local (`fd00::/8`) addresses,
  unless the interface ID is built from a physical MAC, which is
  substituted;
- segment IDs, VLAN tags, and libvirt network names;
- lab host names, which are pseudonyms under the naming scheme in
  conventions.md;
- OU paths, GPO names, group names, and lab service accounts;
- virtual MAC addresses from a hypervisor OUI (`52:54:00` for QEMU and
  KVM, `00:15:5D` for Hyper-V, `00:50:56` and `00:0C:29` for VMware);
- sample SHA-256 hashes and family names;
- timestamps.

These carve-outs are also kept, because substituting them costs
readability and discloses nothing:

- reserved MACs: broadcast, all-zeros, IPv4 and IPv6 multicast, and STP;
- unspecified, multicast, and broadcast IP addresses;
- the public anycast resolvers listed in the
  [appendix](#appendix-pattern-rules), so a lookup against `1.1.1.1`
  still reads as one;
- GUIDs identical on every Windows install, listed in the appendix;
- any value on the redaction utility's allow list, which is how a false
  positive is cleared.

The author's legal name is kept in two places: `LICENSE-CODE`, and the
`author` field of each entry's frontmatter. The name is public by the
author's choice, and it confirms authorship. Everywhere else, it is
Class B.

The author's email appears in no committed file.

Vendor OUIs on lab-plane physical interfaces are also kept, as the
[substitution grammar](#substitution-grammar) shows. Every carve-out is
listed in this standard, so a reader can tell a deliberate keep from a
miss.

### Host plane

Keep the host plane out of every capture ([why](RATIONALE.md#host-plane)). Treat these values
as host-plane values:

- the home WAN address, any VPS address, every address or prefix the ISP
  assigns, and the addresses of ISP equipment, such as the hops past the
  home router in a traceroute;
- the home network's private addresses, which Class C would otherwise
  keep;
- the physical MAC addresses, serial numbers, and hardware UUIDs of the
  host machines;
- the real hostnames and usernames of those machines;
- the home SSID and BSSIDs.

Shape every command run on a host so it prints only lab-plane values,
such as a virtual bridge and the guests on it. As a backstop, put each
host's physical MAC addresses, serial numbers, and hardware UUID, the
home WAN address, any VPS address, and the home network's addresses on
the literal list, so a value that slips into a capture becomes
`[REDACTED]`.

A masked MAC, a suffixed token, or a documentation address that stands
in for a host-plane value counts as a leak. Handle it as a Class B
disclosure under [On exposure](#on-exposure). The network register records no
host-plane value.

## Substitution grammar

Substitute addresses with documentation ranges, which keep the shape of
the output:

| Real value | Substitute |
|---|---|
| Public IPv4 | RFC 5737: `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` |
| Global unicast IPv6 | RFC 3849: `2001:db8::/32` |
| Real domain | RFC 2606: `example.com` |
| Physical MAC on the lab plane | The OUI kept and the device half masked: `00:1b:21:xx:xx:xx` |

Substitute identity-shaped values with bracketed tokens, because a fake
GUID that parses as a GUID would pass for a real one:

```
[TENANT-ID a3f9]      [SERIAL b21c]      [USER]
[SUBSCRIPTION 7e02]   [SSID]             [HOSTNAME]
[OBJECT-ID 4c18]      [BSSID]            [NTLM-HASH]
[JWT]                 [NAME]             [PRODUCT-KEY]
[PRIVATE-KEY]         [REDACTED]         [EMAIL]
```

The four-character suffix is the start of an HMAC-SHA256 of the real
value, keyed with a secret salt. The same tenant ID therefore reads as
the same tenant across entries without being disclosed. A token without
a suffix stands for a value where cross-entry correlation adds nothing.

Substitution is deterministic: the same input produces the same token
everywhere in the repository ([why](RATIONALE.md#redacted-field)).

Keep the salt outside the working tree, and never commit it. The salt is
the HMAC key. If it leaked, anyone who suspects a value, such as a
public address or a tenant ID, could hash the guess and compare it with
the published token. Losing the salt costs only cross-entry consistency
for later substitutions.

Before any rule runs, the redaction utility removes terminal escape
sequences, such as color codes and window titles, because a value
printed right after a color code would otherwise pass every rule. The
session log is therefore the raw log without its escape sequences.

Input saved as UTF-16 with a byte-order mark is read as text, and the
utility's output is always UTF-8.

## Capture-side controls

Shape each command to print only what the entry needs; output that is
never produced needs no redaction. This matters most in `30-identity/`,
where the raw output of a Graph call or a portal export is mostly GUIDs.
Select only the properties the entry needs:

```
PS> Get-MgUser -All | Select-Object DisplayName, UserPrincipalName
```

Apply the same approach to `Get-ADUser -Properties *`, to `ip addr`
where `ip -4 addr` would do, and to `show running-config` where
`show run | section interface` would do.

Before each session:

1. Set a neutral prompt, so the username, hostname, and home directory
   stay out of the log. In a Linux shell, use `PS1='$ '`, or `PS1='# '`
   in a root shell. In PowerShell, define `function prompt { 'PS> ' }`.
2. On a host with a wireless interface, disable the interface unless the
   session needs it, and record its state at the top of the raw log.
   Wireless tooling and system logs print the home SSID and neighboring
   BSSIDs, so keep them out of any session that runs with the interface
   up.

## Raw logs

Keep raw logs, and draft entries that still contain raw output, outside
the working tree. A `.gitignore` entry can be overridden, and a raw file
inside the tree is one careless copy away from a commit. The
`.gitignore` also lists `raw/`, as a backstop for a capture made inside
the tree by habit.

The [capture protocol](writeup-standard.md#capture-protocol) follows
this rule.

## Redaction utility

The redaction utility reads a raw log, applies the literal list and the
[pattern rules](#appendix-pattern-rules), and writes a substituted copy. [`tools/README.md`](tools/README.md)
covers its setup and use.

These rules apply to it:

- Besides the file it redacts, it reads three inputs: the salt, the
  allow list, and the literal list. Keep all three outside the working
  tree, and keep their backups out of any review location that syncs
  (see [Review copies](#review-copies)).
- The literal list contains the values only the author can enumerate,
  such as the real username, real hostname, home SSID, personal domain,
  and the author's name and email. No pattern can find these values,
  and generic secret scanners miss them.
- It keeps the author's name in `LICENSE-CODE` and on an entry's
  `author` line, and substitutes it everywhere else.
- It reports each Class A rule that fires, so the secret can be found
  and rotated.
- Moving its output into the working tree is a separate, deliberate
  step. Entry excerpts are cut from that output.
- When the redaction utility and this section disagree, fix the
  redaction utility.

## Release check

Before committing an entry or its artifacts, confirm that:

1. the redaction utility has run on the draft entry and the raw log,
   producing the entry and its session log;
2. the redaction utility's output was read by eye against the raw log;
3. every file in `assets/` meets the rules in [Screenshots and binary
   files](#screenshots-and-binary-files);
4. `redacted` is `true` when the redaction utility, or a hand edit,
   substituted anything in the entry or its artifacts, and `false`
   otherwise.

## Push gate

A commit can still be undone with `git reset`, `git commit --amend`, or
an interactive rebase until it is pushed. The pre-push hook is the last
automated check before that point. It runs the redaction utility in
check mode over every file version in the commits being pushed,
including the changes a merge commit makes.

Check mode cannot read images: it names each image it skips, and
[Screenshots and binary files](#screenshots-and-binary-files) sets how images are reviewed. Check
mode refuses every other binary file, which blocks the push.

> [!WARNING]
> `git push --no-verify` skips the hook.

Push protection is enabled on the hosting service account that pushes to
this repository. It rejects a push that contains a recognized vendor
credential format, such as a cloud access key, and `--no-verify` does
not skip it. Push protection knows nothing of the literal list or
Class B, so the hook remains the control for those values.

Before each push, read the outgoing commits:

```
$ git log -p @{upstream}..
```

On a branch with no upstream, read `git log -p origin/main..`. For the
first push to an empty remote, read `git log -p`.

The output shows only what is new, which is where a leak would be,
including content that a Markdown renderer hides or reformats. It also
shows the commit messages, which the hook does not check.

## Review copies

A copy of a document made for proofreading outside the working tree can
leak what the repository later corrects. When a review finds an
identifier that should not ship, the repository gets the corrected
version, but the review copy keeps the original. If the copy syncs to a
service with version history, the original stays in that history even
after the copy is deleted.

Find out once whether the review location syncs, and to what. If it
does, stop syncing it, or review documents that contain captured output
inside the working tree instead. Prose with no captured output is
unaffected.

## Screenshots and binary files

The text rules above cannot inspect images or binary captures. Apply
these rules to them:

1. Strip all metadata from every image when it is added. Phone and
   camera images contain GPS coordinates, and screenshots record the
   capture tool and sometimes a username in a file path.
2. Before committing an image, read its whole frame: title bars, the
   taskbar, browser tabs and the address bar, notifications, the clock,
   and any second monitor.
3. Crop each image to the region the entry needs. Whatever the crop
   removes cannot leak, even when a review misses it.
4. Prefer text to an image. A pasted terminal block can be redacted,
   diffed, and searched, while a screenshot can only be inspected by
   eye.
5. Keep binary captures, such as packet captures, event log files, trace
   files, and memory dumps, out of the repository, because the redaction
   utility reads text only. Quote a text export of the records the entry
   needs.

## On exposure

Treat a disclosed Class A value as compromised. Run these steps in
order:

1. Rotate the secret before touching the repository. The value is
   already public, and the rewrite takes minutes.
2. Rewrite history to remove the value, then force push.
3. Ask the hosting service's support team to purge cached views and
   unreachable objects. Until it does, the original commit stays
   retrievable by SHA.
4. Check for forks, which the rewrite cannot reach.
5. Record the disclosure and its handling in an entry.

For a Class B disclosure, follow steps 2 to 5. Follow step 1 as well
when the value can be changed, such as a WAN address. A hardware serial
cannot be changed.

## Known limits

- Pattern rules catch structured values. A comment typed mid-session, a
  path containing a real name, or a hostname invented on the spot and
  missing from the literal list gets past them. The by-eye read in the
  release check covers that gap.
- Four Class A types have no pattern rule: plaintext passwords, API keys
  and bearer tokens that are not JWTs, agent enrollment tokens, and
  client secrets. A bare NT hash, without its LM half, is reported only
  as `review:bare32`, and the push still goes through. Capture-side
  shaping is the only reliable control for these values.
- Reverse DNS names can contain a public address in dashed form, such as
  an ISP's name for the WAN hop in a traceroute. The address rules miss
  that form, so add it to the literal list.
- Public IPv4 addresses map onto 762 documentation addresses, so two
  real addresses can share a substitute.
- The literal list matches exact strings, so it cannot cover a whole
  home subnet. A home-network address missing from the list, such as the
  broadcast address, passes as an RFC 1918 address. Shaping each command
  run on a host is the control that covers the rest.
- Home IPv6 addresses cannot be listed reliably either, because privacy
  addresses change and one address can be written several ways. The
  IPv6 rule would turn a missed one into a suffixed documentation
  address, which counts as a host-plane leak. Keep host commands to IPv4
  output, such as `ip -4 addr`, where the entry allows it.
- The redaction utility cannot read images. The pre-push hook names each
  image it skips and refuses every other binary file, including a UTF-16
  file without a byte-order mark.
- The redaction utility skips REDACTION.md, which holds example patterns
  and addresses by design, and the pre-push hook inherits the skip. Read
  REDACTION.md by eye whenever it changes: a real value pasted into it
  as an illustration would pass every check.
- Aggregation is out of scope. Timestamps across many entries describe a
  routine. The mitigation is judgment about what an entry needs, applied
  at capture time.
- This standard governs what leaves the lab. Hardening the lab is the
  work of the entries.

## Appendix: pattern rules

The redaction utility implements this specification. Address rules parse
each candidate, because no regex safely separates a public address from
a private one.

| Target | Pattern |
|---|---|
| GUID | `[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}` |
| MAC, colon or hyphen | `[0-9A-Fa-f]{2}([:-])([0-9A-Fa-f]{2}\1){4}[0-9A-Fa-f]{2}`, with one separator throughout |
| MAC, Cisco dotted | `([0-9A-Fa-f]{4}\.){2}[0-9A-Fa-f]{4}` |
| JWT | `eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+` |
| Private key block | `-----BEGIN [A-Z ]*PRIVATE KEY-----` to the matching END line |
| Windows product key | `([A-Z0-9]{5}-){4}[A-Z0-9]{5}` |
| Cisco serial | `\b[A-Z]{3}[0-9]{4}[A-Z0-9]{4}\b` |
| LM:NT hash pair | `[0-9a-fA-F]{32}:[0-9a-fA-F]{32}` |
| IPv4 | Match all; allow RFC 1918, `127.0.0.0/8`, `169.254.0.0/16`, RFC 5737, `0.0.0.0/8`, `224.0.0.0/4`, `255.255.255.255`, and the anycast resolvers |
| IPv6 | Match all; allow `fe80::/10` and `fd00::/8` unless the interface ID embeds a physical MAC, `::1`, `::`, `ff00::/8`, `2001:db8::/32` |
| Literals | Exact strings from the literal list, matched at word boundaries and ignoring case; the author's name is kept in `LICENSE-CODE` and on a frontmatter `author` line |

The anycast resolvers are `1.1.1.1`, `1.0.0.1`, `8.8.8.8`, `8.8.4.4`,
`9.9.9.9`, and `149.112.112.112`. The GUIDs kept as captured are the nil
GUID, the Default Domain Policy
(`31B2F340-016D-11D2-945F-00C04FB984F9`), and the Default Domain
Controllers Policy (`6AC1786C-016F-11D2-945F-00C04FB984F9`).

A GUID becomes a `[TENANT-ID]` or `[SUBSCRIPTION]` token when its line
names a tenant or a subscription, and an `[OBJECT-ID]` token otherwise,
each with its suffix.

Bare 32-character hex is counted, reported as `review:bare32`, and left
in place. It matches every MD5 hash and every GUID written without
hyphens, so substituting it would bury real findings in noise. The count
is a warning, and the run still succeeds.
