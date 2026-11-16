# tools

The standards refer to each tool by its role. This page names each tool
and explains how to install and run it. `redact-util` and the pre-push
hook live in this directory, and the other two are standard commands:

| Tool | File or command | Implements |
|---|---|---|
| Redaction utility | `redact-util` | [`REDACTION.md`, Redaction utility](../REDACTION.md#redaction-utility) |
| Pre-push hook | `hooks/pre-push` | [`REDACTION.md`, Push gate](../REDACTION.md#push-gate) |
| Placeholder check | Two `grep` commands | [`conventions.md`, Placeholders](../conventions.md#placeholders) |
| Session logger | `script`, from util-linux | [`writeup-standard.md`, Capture protocol](../writeup-standard.md#capture-protocol) |

## Installation

`redact-util` needs Python 3 and nothing else. On the workstation that
holds the raw logs, create the capture directory and the three setup
files once:

```
$ mkdir -p ~/captures
$ chmod 700 ~/captures
$ head -c 32 /dev/urandom | base64 > ~/.redact-salt
$ touch ~/.redact-allow ~/.redact-literals
$ chmod 600 ~/.redact-salt ~/.redact-allow ~/.redact-literals
```

Then install the hook from the repository root:

```
$ git config core.hooksPath tools/hooks
```

The hook setting is stored per clone, so repeat this command in every
new clone.

Back up the salt and both lists, and keep the backups out of any synced
review location. Losing the salt changes every token suffix from that
point on.

Every other machine that pushes to this repository also needs Python 3,
both lists, and the hook. It does not need the salt, because check mode
does not use it. Restore the allow list and the literal list on that
machine from their backup, then run these commands from the clone's
root:

```
$ chmod 600 ~/.redact-allow ~/.redact-literals
$ git config core.hooksPath tools/hooks
```

## Configuration

`redact-util` reads three setup files:

| Path | Holds |
|---|---|
| `~/.redact-salt` | The secret salt for token suffixes |
| `~/.redact-allow` | Values `redact-util` keeps, one per line |
| `~/.redact-literals` | Values `redact-util` always substitutes, as one `TYPE value` pair per line |

The environment variables `REDACT_SALT`, `REDACT_ALLOW`, and
`REDACT_LITERALS` override the three paths. Raw logs and draft entries
live in `~/captures/`.

### Literal list

Each line names a type, then a value that runs to the end of the line
and may contain spaces. Lines starting with `#` are comments, and
matching ignores case.

| Type | Substitute |
|---|---|
| `USER` | `[USER]` |
| `HOSTNAME` | `[HOSTNAME]` |
| `SSID` | `[SSID]` |
| `BSSID` | `[BSSID]` |
| `DOMAIN` | `example.com` |
| `AUTHOR` | `[NAME]`, apart from `LICENSE-CODE` and an entry's frontmatter `author` line |
| `EMAIL` | `[EMAIL]` |
| `NAME` | `[NAME]` |
| `OTHER` | `[REDACTED]` |

For example:

```
USER jdoe
HOSTNAME workstation1
SSID Home Network 5G
DOMAIN mydomain.net
AUTHOR Jane Doe
AUTHOR Doe
EMAIL jane.doe@example.com
NAME John Roe
OTHER 203.0.113.5
OTHER 203-0-113-5
OTHER 192.168.1.23
```

List your legal name under `AUTHOR`, once in full and once for each part
that could appear alone, such as a surname in a file path. List every
email address you use under `EMAIL`.

List the host-plane values from REDACTION.md under `OTHER`, so each one
becomes `[REDACTED]` instead of a masked MAC or a documentation address:

- each host's physical MAC addresses, in both colon and hyphen form;
- each host's serial numbers and hardware UUID;
- the home WAN address and any VPS address, in both dotted and dashed
  form;
- every home-network address a host command might print, such as the
  router, the host's own address, and the broadcast address.

Update these lines whenever a value changes. Other public addresses need
no line, because the IPv4 and IPv6 rules substitute them.

### Allow list

Each line holds a value that `redact-util` keeps exactly as captured.
Add a false positive to clear it, and add each generic KMS client key an
entry shows.

## Usage

### Recording a session

Record each lab session with `script`, which writes everything the
terminal shows to the raw log. Use one raw log per entry, named after
its lab ID:

```
$ script -a -f -q ~/captures/LAB-0002.log
```

`-a` appends, so a session restarted after an interruption keeps its
earlier output. `-f` writes each update to disk at once, so the log
survives a crashed terminal. Type `exit` to stop recording. Do not add
`-I` or `-B`: both also log keystrokes, including passwords typed at a
prompt that does not echo them.

### Redacting a capture

After each lab, redact the raw log and the draft entry, each into a new
file:

```
$ tools/redact-util ~/captures/LAB-0002.log ~/captures/LAB-0002-session.log
$ tools/redact-util ~/captures/LAB-0002.md ~/captures/LAB-0002-entry.md
```

The summary on standard error counts each substitution by rule. A hit on
an `AUTHOR` value counts as `author`, and a hit on any other literal
counts as `literal`. Read the summary when setting `redacted`.

When the summary reports a Class A rule, run check mode on the raw log
to find the line, then rotate the secret:

```
$ tools/redact-util --check ~/captures/LAB-0002.log
```

To check files without changing them:

```
$ tools/redact-util --check file...
```

Check mode also reports how many 32-character hex strings each file
contains, as `review:bare32`. It names each image it skips, and refuses
any other binary file. Both modes read UTF-16 input as text when the
file starts with a byte-order mark.

| Exit status | Meaning |
|---|---|
| 0 | Done, or check mode found nothing |
| 1 | Check mode found a value to substitute |
| 2 | A setup file is missing or malformed, the arguments are wrong, or the input is binary and, in check mode, not an image |
| 3 | Done, and a Class A value was substituted; rotate the secret |

### Pre-push hook

The hook runs check mode over every file version in the commits being
pushed, including versions a later commit replaced and the changes a
merge commit makes. Any finding blocks the push, and so does any binary
file that is not an image. The hook does not check commit messages, and
`git push --no-verify` skips it; see [`REDACTION.md`, Push
gate](../REDACTION.md#push-gate).

### Placeholder check

Run both commands against an entry or a layer README. Silence from both
is a pass.

The first command finds bracketed placeholders and unassigned IDs
anywhere in the file:

```
$ grep -nE '\[PENDING\]|\[PASTE\]|LAB-XXXX' file
```

The second command finds angle-bracket slots outside command and output
position:

```
$ grep -nE '<[^ >!][^>]*>' file | grep -vE '^[0-9]+:( {8}| {9}| {14,})[^ ]'
```

The second `grep` drops lines that start with eight or nine spaces, or
with fourteen or more, which is where commands and output sit under a
section or a subsection. Output nested deeper inside a section-level
block is still checked, so read any hit there by eye.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `~/.redact-salt is missing`, or the same for `~/.redact-allow` or `~/.redact-literals` | Installation was skipped on this machine | Run the installation commands, or restore the files from their backup |
| `bad line in ~/.redact-literals` | A line has no type, an unknown type, or no value | Rewrite it as `TYPE value`, with a type from the literal list table |
| `is binary; redact-util reads text only` | The file has NUL bytes and is not an image. A UTF-16 transcript without a byte-order mark looks the same | Convert a transcript with `iconv -f UTF-16LE -t UTF-8 raw.log > raw-utf8.log`. Keep any other binary file out of the repository |
| `skipped image; read it by eye` | The push includes an image, which check mode cannot read | Review the image as REDACTION.md, Screenshots and binary files requires |
| `review:bare32` | The file contains a 32-character hex string, such as an MD5 hash or a bare NT hash | Read each one. If one is a credential hash, rotate the credential and remove the hash before pushing |
| `pre-push: finding in <path> at <hash>` | A commit being pushed contains a value `redact-util` would substitute, or a binary file that is not an image | Fix that commit before pushing: `git commit --amend` for the latest commit, or `git rebase -i` for an older one |
| Every push is blocked on a new machine | The allow list or the literal list is missing, and `redact-util` names it before the hook reports a finding | Restore both lists from their backup |
| `<file>:<line>: author` | Your name appears outside `LICENSE-CODE` and a frontmatter `author` line | Remove it from that line. If the line is meant as the `author` field, check that it sits inside the frontmatter |
| A safe value was substituted | A false positive | Add the exact value to `~/.redact-allow` |
| A private value survived | No rule covers it | Add it to `~/.redact-literals` and redact again, or edit it by hand and set `redacted: true` |
| The hook never runs | `core.hooksPath` is unset in this clone, or the hook lost its execute bit | Run `git config core.hooksPath tools/hooks` and `chmod +x tools/hooks/pre-push` |
| Token suffixes changed between runs | The salt changed or was lost | Restore the salt from its backup |
