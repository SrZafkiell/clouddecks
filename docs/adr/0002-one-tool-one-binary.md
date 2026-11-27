# 0002. One tool, one repository, one executable

- Status: Accepted
- Date: 2026-11-26 (decided), recorded 2026-11-27
- Step: P0-10
- Decided by: the maintainer

## Context

Clouddecks started as one idea with two parts: a desktop app that opens AWS Session
Manager tunnels through a jump host, and a manager for the AWS profiles those tunnels
need. More tools may follow. The people who will use them are mostly colleagues who get
the tool from someone else, often as a single file, and most of them need only one of
the tools.

The two parts meet in one place: the profiles. The tunnel tool needs working AWS
credentials; the profile tool would create and edit them. The AWS CLI and the AWS SDKs
already agree on where profiles live (`~/.aws/config` and `~/.aws/credentials`).

## Options

- One app with tool modules (tunnels in the first release, profiles in the second). One
  download, one release pipeline, shared code without ceremony. But everyone gets every
  tool, the app grows with each module, and a release of one part ships all of them.
- Separate apps that share a Go library from the start. Each tool stands alone, but the
  shared library has to be designed before there is a second user of it.
- Separate tools under one name, each its own repository and executable, with a home
  repository for what they share (plan, decisions, risks, CI workflows), and shared code
  extracted only when a second tool needs it.

The first option was chosen on 2026-11-26 and replaced the same day by the third: a
colleague who needs only the tunnels should get only the tunnels.

## Decision

Each tool of Clouddecks is its own repository and its own executable. Tools share state
only through the standard AWS configuration files, which they read (and, for the
profile tool, write) in the formats the AWS CLI uses. This repository, `clouddecks`, is
the home: the plan, the decision records, the risk register, the board guide, the
planning scripts and the reusable CI workflows that every tool calls. Go code is shared
between tools only when a second tool needs it, by extracting it into a module of its
own.

## Consequences

- A person downloads and runs only the tool they need, and each tool is released on its
  own schedule.
- Each tool repository carries its own release pipeline; the reusable workflows in this
  repository keep the security gates in one place (P0-15).
- Until a second tool exists, some code (reading the AWS files, the logger) lives in the
  first tool only. Extracting it later is a planned cost, not a surprise.
- Two tools never talk to each other directly. If they ever need to, that is a new
  decision.
