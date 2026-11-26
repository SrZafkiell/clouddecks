# M02 - Deck files

Track: B. Phase: 1. Size: M. Depends on: M01. Unlocks: M04, M05.

## Goal

An administrator writes one small file that describes the tunnels of a team, hands it
out, and a user drops it into a folder or imports it. The file never holds a secret, and
a broken or hostile file cannot hurt the person who opens it.

## Scope

- The deck file format and its published schema.
- A strict parser with tests against hostile input.
- The decks folder, the import dialog and drag and drop.
- The mapping from the profile a deck file suggests to a profile on this computer.
- An example file and an authoring guide.

## Design notes

- A deck file is YAML with a `version` field. It describes tunnels: a name, a region,
  the jump host (by instance ID or by tags), the remote host and port, the local port,
  and a suggested profile name. Nothing else.
- Targets by tag survive the replacement of the instance, so an administrator does not
  have to hand out a new file after a rebuild.
- A deck file comes from someone else, so it is input to distrust: unknown keys are
  refused, the size and the nesting are limited, YAML aliases are off, anything that
  looks like a credential is refused with a clear message, and the listener can only be
  on the loopback address and on a port above 1023.
- The profile mapping is local. Two people with different profile names use the same
  deck file.
- Before a file is added, the import dialog shows in plain words what it will do: which
  account profile it uses and where each tunnel ends.
- Signed deck files are M07 (Phase 2). Until then the import dialog is the control.

## Tasks

- M02-A [feature] Deck file format -- Done when: the format has an accepted ADR and a published schema, the parser accepts the example file, and every hostile case in the test list is refused with a message that names the line. -- Steps: M02-01, M02-02, M02-03
- M02-B [feature] Adding a deck file -- Done when: a file dropped into the decks folder or imported from the window shows its tunnels after a preview, a broken file shows its error without stopping the others, and each tunnel is mapped to a local profile. -- Steps: M02-04, M02-05, M02-06, M02-07

## Steps

### hopdeck

- M02-01 [adr p1 S] Deck file format -- Done when: accepted: versioned YAML, no secrets, targets by instance ID or by tags, loopback listeners only, and how a later version of the format is read.
- M02-02 [feature p1 M] Schema and strict parser -- Done when: a JSON Schema of the format is published in the repository, the parser refuses unknown keys, wrong types and files over the size limit, and every error names the file, the line and the field.
- M02-03 [security p1 M] Hostile deck file tests -- Done when: tests cover aliases and deep nesting, oversized files, fields that look like access keys or passwords, a listener that is not loopback, a local port under 1024 and duplicate tunnel names, and a fuzz test of the parser runs in CI.
- M02-04 [feature p1 M] Decks folder -- Done when: files in the decks folder are loaded at start and when they change, each tunnel appears in the list, and a broken file is shown with its error while the other files keep working.
- M02-05 [feature p2 M] Import with preview -- Done when: a file chosen in the dialog or dropped on the window is shown as a plain-language preview (profile, region, jump host, remote host and port, local port) and is copied into the decks folder only after the user accepts.
- M02-06 [feature p1 S] Profile mapping -- Done when: each deck file is mapped to a local AWS profile chosen from a list, the mapping is stored in the settings and never in the deck file, and a tunnel without a mapping says so instead of failing.
- M02-07 [docs p2 S] Example deck file and authoring guide -- Done when: the repository has an example file on made-up data that the tests load, and a page that tells an administrator how to write one and what never goes in it.

## Schedule

Planned weeks. Each step starts on the Monday shown and is due that Sunday; a task runs
from its first step to its last, and the module's epic and its milestone end with the
last week. Dates change here first, and the sync script moves the board.

| Week of | Steps |
|---|---|
| 2026-11-30 | M02-01, M02-02, M02-04, M02-06 |
| 2026-12-14 | M02-03, M02-05, M02-07 |

## Measures

- Number of hostile cases in the test list, and fuzzing time per CI run.
