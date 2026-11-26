# P1 - Release pipeline

Track: A. Phase: 1. Size: M. Depends on: P0, M01. Unlocks: R1.

## Goal

A tag produces a Windows package that anyone can check before running it: built on a
hosted runner from pinned inputs, with its bill of materials, its provenance, and guides
for the person who uses it and for the person who hands it out.

## Scope

- The Windows build and package.
- Bill of materials, build provenance, the vulnerability gate.
- The outcome of the signing decision.
- User guide, admin guide, README.

## Design notes

- The executable is given to people who are not developers, so "how do I know this file
  is the real one" needs an answer they can follow: a checksum in the release notes and
  one command that verifies the build provenance.
- An executable without a paid certificate makes Windows show a SmartScreen warning on
  first run. ADR P0-13 decides what v0.1.0 does about it; this module carries it out and
  the user guide shows the screen.
- The release job is the only one with write permissions, runs only on a protected tag,
  and builds nothing from a fork.
- The window needs the WebView2 runtime, which Windows 11 ships. The package handles a
  machine that lacks it: a bundled bootstrapper, or a message that says what to install.
- Linux packages are M06 (Phase 2). macOS waits for a decision on the Apple developer
  program.

## Tasks

- P1-A [ci] Windows build -- Done when: a protected tag produces a versioned Windows package on a hosted runner from pinned inputs, and two builds of the same tag are compared and the differences explained. -- Steps: P1-01, P1-02, P1-03
- P1-B [security] Provenance -- Done when: every release carries a bill of materials and a build attestation that one documented command verifies, a known high vulnerability blocks the release, and the signing decision is carried out. -- Steps: P1-04, P1-05, P1-06, P1-07
- P1-C [docs] Guides -- Done when: a person who has never used AWS tooling can install, add a deck file and connect by following the user guide, and an administrator can prepare a jump host, a policy and a deck file by following the admin guide. -- Steps: P1-08, P1-09, P1-10

## Steps

### hopdeck

- P1-01 [ci p1 M] Windows build in CI -- Done when: a v* tag builds the executable on a GitHub-hosted Windows runner with pinned Go, Node and Wails versions, a machine without the WebView2 runtime is handled as the design notes say, and the job has no secret it does not need.
- P1-02 [ci p2 S] Version stamping -- Done when: the version and the commit are in the binary, in the file properties on Windows and on the About screen, and they come from the tag.
- P1-03 [ci p2 M] Portable package -- Done when: the release has a zip with the executable, LICENSE, the third-party notices and a checksum file, and a second build of the same tag is compared with the first and the differences are written down.
- P1-04 [security p1 M] Bill of materials -- Done when: a CycloneDX document covering the Go modules and the npm packages is attached to every release.
- P1-05 [security p1 M] Build provenance -- Done when: every release artifact has a GitHub artifact attestation, the user guide gives the one command that verifies it, and CI runs that command against the artifact before the release is published.
- P1-06 [security p2 M] Signing outcome -- Done when: what ADR P0-13 chose is in place: either the executable is signed and Windows shows the publisher, or the release notes and the user guide show the SmartScreen screen and the verification to do first.
- P1-07 [security p2 M] Vulnerability gate -- Done when: govulncheck and an npm audit run in the release job and a known high vulnerability in shipped code stops the release, with a documented way to record an accepted exception.
- P1-08 [docs p1 M] User guide -- Done when: one page covers download and verify, first start, adding a deck file, choosing a profile, Connect, Renew and the common errors, with screenshots taken on the example deck file only.
- P1-09 [docs p2 M] Admin guide -- Done when: one page covers the jump host (no inbound rule, no public address, no key pair), the reference policy, the sign-in kinds to prefer (IAM Identity Center or a role with MFA rather than long-lived keys), writing a deck file, handing it out, and stopping an idle host on the AWS side.
- P1-10 [docs p2 S] README -- Done when: it says in one screen what Hopdeck is and is not, how it differs from an SSH tunnel manager (no open port, no SSH key, IAM identity), and links the guides, the threat model and the verification command.

## Schedule

Planned weeks. Each step starts on the Monday shown and is due that Sunday; a task runs
from its first step to its last, and the module's epic and its milestone end with the
last week. Dates change here first, and the sync script moves the board.

| Week of | Steps |
|---|---|
| 2026-12-21 | P1-01, P1-02 |
| 2026-12-28 | P1-03, P1-04, P1-05, P1-06, P1-07, P1-08, P1-09, P1-10 |

## Measures

- Minutes from tag to published release.
- Number of components in the bill of materials.
