# 0004. Deck file format

- Status: Accepted
- Date: 2026-11-27
- Step: M02-01 (Hopdeck)
- Decided by: the maintainer

## Context

An administrator describes the tunnels of a team in one small file and hands it out;
each person drops it into Hopdeck's decks folder. The file is written by someone else,
so Hopdeck treats it as input to distrust. It must never hold a secret, because it is
passed around by email and chat. Each person has their own AWS profiles, with their own
names, so the file can only suggest one. A jump host may be rebuilt, which changes its
instance ID.

A tunnel is a Session Manager port forward. With a remote host, it uses the document
AWS-StartPortForwardingSessionToRemoteHost (a service behind the jump host, such as a
database); without one, AWS-StartPortForwardingSession (a port on the jump host itself).

## Options

- YAML. Readable, allows comments, familiar to people who write AWS templates. Its
  general features are a risk in a file from someone else: anchors and aliases (the
  "billion laughs" expansion), merge keys, several documents in one file, and values
  whose type depends on how they are written.
- JSON. Strict and simple to parse, but no comments, and administrators would write
  braces and quotes by hand.
- TOML. Strict and allows comments, but a list of tunnels with nested tables reads
  worse than YAML and is less familiar to the people who will write these files.

## Decision

A deck file is YAML, restricted as follows.

The file:

- is named `<something>.hopdeck.yaml`; other files in the decks folder are ignored;
- is UTF-8, at most 64 KiB, and holds exactly one YAML document;
- uses no anchors, aliases, merge keys or custom tags, and no nesting deeper than the
  format itself needs.

The fields, and nothing else:

```yaml
version: 1                 # required; the format version
name: Team databases       # required; 1 to 64 characters
profile: team-dev          # optional; a suggestion, mapped to a local profile
region: us-east-1          # optional; the default for every tunnel
tunnels:                   # required; 1 to 100 tunnels
  - name: orders-db        # required; unique in the file
    region: eu-west-1      # optional; required when the file sets no region
    jump_host:             # required; exactly one of:
      instance_id: i-1234567890abcdef0  #   an instance ID, or
      # tags: {Role: jump-host}         #   1 to 10 tags that find the instance
    remote:
      host: orders.db.example.internal  # optional; without it, a port on the jump host
      port: 3306                        # required; 1 to 65535
    local_port: 13306      # required; 1024 to 65535, unique in the file
```

- A tunnel name is 1 to 64 letters, digits, dots, underscores or hyphens. A region looks
  like `us-east-1`. An instance ID is `i-` followed by 8 or 17 hexadecimal digits. A tag
  key is 1 to 128 characters and a tag value at most 256. A remote host is a DNS name or
  an IPv4 address.
- There is no field for the listening address. Every tunnel listens on 127.0.0.1, so a
  deck file cannot share a tunnel with the local network.
- The suggested profile is only a suggestion. Each person maps the deck file to one of
  their own profiles, and that mapping is stored in Hopdeck's settings, never in the
  deck file (M02-06).
- Hopdeck refuses the whole file, with a message that names the file, the line and the
  field, when it finds an unknown key, a missing required field, a wrong type, a value
  out of range, a duplicate key, tunnel name or local port, a YAML feature listed above,
  or a value that looks like a credential: an AWS access key ID, or a string shaped like
  a secret access key.
- `version` is required. A file with a version newer than this Hopdeck knows is refused
  with a message that it needs a newer Hopdeck. A later version of the format comes with
  a reader for the earlier ones, so a file that works keeps working; a change that would
  give an old file a different meaning is not allowed in the same version.
- A JSON Schema of the format is published in the Hopdeck repository for editors and for
  administrators who validate their files; Hopdeck's own parser is the authority.

## Consequences

- Administrators get a short, commented file that reads like documentation of the
  tunnels. Targets by tags survive the rebuild of a jump host.
- The parser cannot simply decode into a struct: it reads the YAML node tree first, so
  it can refuse aliases and report the line of every error. Hopdeck uses
  `go.yaml.in/yaml/v3`, the YAML organization's maintained line of the archived
  `gopkg.in/yaml.v3`, pinned to one version.
- Strictness means a typo is caught when the file is loaded, not when a tunnel fails.
  It also means a file written for a newer format is refused rather than half-read.
- The limits (64 KiB, 100 tunnels) are far above any real team's needs and keep a
  hostile file from costing memory or time. The hostile cases get their own tests and a
  fuzz test (M02-03).
- Signatures are not part of version 1. Until signed deck files arrive (M07), the import
  preview is the control: it shows in plain words what a file will do before it is
  added (M02-05).
