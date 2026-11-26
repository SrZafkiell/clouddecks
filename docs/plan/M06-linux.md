# M06 - Linux

Track: A. Phase: 2. Size: S. Depends on: R1. Unlocks: nothing.

## Goal

The same Hopdeck release also ships for Linux, built and attested by the same pipeline
and proved on a clean machine.

## Scope

- A spike on the webview dependency across distributions.
- The Linux build and package in CI.
- The release checklist on Linux.

## Design notes

- On Linux, Wails v3 uses GTK4 and WebKitGTK 6.0 by default, which older distributions
  do not ship. The spike finds which distributions can run it and which package format
  to ship before the pipeline is written.
- macOS is not in this module. It needs a paid Apple developer membership to pass
  Gatekeeper without a warning; that is a decision of its own.

## Tasks

- M06-A [ci] Linux package -- Done when: a tag also produces an attested Linux package, and the release checklist passes on a clean machine of each supported distribution. -- Steps: M06-01, M06-02, M06-03

## Steps

### hopdeck

- M06-01 [spike p2 M] Hopdeck on Linux -- Done when: the app is run on the current Ubuntu LTS and on Fedora, the WebKitGTK versions and the application folders are checked, and the result names the package format to ship.
- M06-02 [ci p2 M] Linux build in CI -- Done when: a v* tag also builds the Linux package on a hosted runner, with the same bill of materials, attestation and vulnerability gate as the Windows one.
- M06-03 [test p2 M] Release checklist on Linux -- Done when: the checklist of R1-01 passes on a clean machine of each supported distribution, and the user guide has its Linux section.

## Schedule

Planned weeks. Each step starts on the Monday shown and is due that Sunday; a task runs
from its first step to its last, and the module's epic and its milestone end with the
last week. Dates change here first, and the sync script moves the board.

| Week of | Steps |
|---|---|
| 2027-04-05 | M06-01, M06-02, M06-03 |

## Measures

- Distributions and versions proved, listed in the README.
