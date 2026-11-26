# M07 - Signed deck files

Track: B. Phase: 2. Size: M. Depends on: R1. Unlocks: nothing.

## Goal

A user can tell that a deck file comes from their administrator and was not changed on
the way, without understanding cryptography.

## Scope

- The signing scheme and how keys are trusted.
- Signing from the command line, verification on import.
- What the window shows for signed, unsigned and changed files.
- Tamper tests.

## Design notes

- A changed deck file could point a tunnel at another host or ask for another profile.
  In v0.1.0 the import preview is the control; a signature makes it checkable.
- The administrator signs with a key that stays on their machine. Users trust a key once
  and see its name from then on.
- Unsigned files keep working, with a visible notice: a team should be able to start
  without a key ceremony.

## Tasks

- M07-A [security] Signed deck files -- Done when: an administrator signs a deck file with one command, a user sees who signed a file when importing it, and a changed file, a wrong key and a removed signature are each refused or flagged as designed. -- Steps: M07-01, M07-02, M07-03, M07-04

## Steps

### hopdeck

- M07-01 [adr p2 S] Deck file signatures -- Done when: accepted: the signature format, where the signature lives, how a user comes to trust a key, and what happens to unsigned files.
- M07-02 [feature p2 L] Sign and verify -- Done when: the executable has a command-line mode that creates a key and signs a deck file, and every import and every load of the decks folder verifies the signature against the trusted keys.
- M07-03 [feature p2 M] Signatures in the window -- Done when: the import preview and the tunnel detail show the signer's name, an unsigned file shows a notice, and a file that changed after it was trusted is disabled until the user decides.
- M07-04 [security p2 M] Tamper tests -- Done when: tests cover a changed file, a signature by an unknown key, a removed signature and a swapped signature, and each ends in the designed state.

## Schedule

Planned weeks. Each step starts on the Monday shown and is due that Sunday; a task runs
from its first step to its last, and the module's epic and its milestone end with the
last week. Dates change here first, and the sync script moves the board.

| Week of | Steps |
|---|---|
| 2027-04-05 | M07-01, M07-02, M07-03, M07-04 |

## Measures

- Number of tamper cases in the test list.
