# R1 - Release v0.1.0

Track: A. Phase: 1. Size: S. Depends on: M05, P1. Unlocks: M06, M07.

## Goal

Hopdeck v0.1.0 for Windows is published, proved on a clean machine by following the
user guide, and the Clouddecks home page says what exists and what comes next.

## Scope

- The release checklist on a clean Windows machine.
- The tag and the release notes.
- A short recording on lab data.
- The home repository's front page.

## Design notes

- The clean machine is a Windows Sandbox or a fresh virtual machine: no Go, no Node, no
  AWS CLI, no plugin. That is the claim the release makes, so that is what is tested.
- The recording and every screenshot use the example deck file, with the account ID
  masked on screen (M03-02), made-up names for the jump host and the target, and no real
  instance ID or address in view.

## Tasks

- R1-A [chore] Release v0.1.0 -- Done when: v0.1.0 is published with its attestation, the release checklist passed on a clean Windows machine, and the home repository shows the tool, a recording and the roadmap. -- Steps: R1-01, R1-02, R1-03, R1-04

## Steps

### hopdeck

- R1-01 [test p1 M] Release checklist on a clean machine -- Done when: on a Windows machine with no developer tools the package is downloaded, verified, started and connected to the lab by following the user guide word for word, and every place where the guide was wrong is fixed.
- R1-02 [chore p1 S] Tag and release notes -- Done when: v0.1.0 is tagged from main with signed commits, the release notes list what it does, what it does not do yet and how to verify it.
- R1-03 [docs p2 M] Recording -- Done when: a recording under one minute shows import, Connect, a query through the tunnel and Disconnect on lab data, and it is in the README.

### clouddecks

- R1-04 [docs p2 S] Home page -- Done when: the README of the home repository lists the decks with their state, links the board and the plan, and shows the roadmap of the next ones.

## Schedule

Planned weeks. Each step starts on the Monday shown and is due that Sunday; a task runs
from its first step to its last, and the module's epic and its milestone end with the
last week. Dates change here first, and the sync script moves the board.

| Week of | Steps |
|---|---|
| 2026-12-28 | R1-01, R1-02, R1-03, R1-04 |

## Measures

- Minutes a first-time user needs from download to a connected tunnel, measured once
  with a person who did not build it.
