# P0 - Foundation

Track: A. Phase: 1. Size: M. Depends on: nothing. Unlocks: everything.

## Goal

Repositories that are safe to build in public from day one: the same rules, templates,
decisions, threat model and CI security gates in both of them before the first line of
product code.

## Scope

- Repository settings, governance files and rulesets, in every repository.
- Project management: labels, milestones, one public project board across the
  repositories, and the plan-to-issues sync.
- The ADR process and the founding ADRs.
- Threat model v1 for a desktop app that handles cloud credentials.
- CI security gates as reusable workflows.
- Developer tooling that works on Windows and Linux.
- The guard that keeps real account data out of the repositories.

## Design notes

- One rule, one step: a step here covers every repository, and it is done only when
  `clouddecks` and `hopdeck` both comply. A later tool joins when its module starts.
- The CI gates are reusable workflows in this repository. Each tool calls them pinned by
  commit SHA, so a gate is fixed in one place and no repository drifts.
- Merge commits, not rebase merges: GitHub rewrites rebased commits without signature
  verification, which conflicts with required signed commits; squash merges would lose
  the per-concern commits.
- Git for Windows converts LF to CRLF on checkout by default; `.gitattributes` keeps
  every text file LF except PowerShell scripts.
- The tools are used against real cloud accounts. The repositories must never hold
  anything from one: fixtures, screenshots, issues and recordings use the documented
  example account ID and made-up instance IDs and host names, results from a lab are
  published with their identifiers replaced, and a CI guard backs the rule.
- The source is public and the binary runs on other people's computers. Nothing in the
  design may depend on the code being secret; the threat model says so explicitly.

## Tasks

- P0-A [chore] Repository rules -- Done when: every repository carries the same governance files and templates, and refuses a direct push to main, an unsigned commit and a merge without its required checks. -- Steps: P0-01, P0-02, P0-03, P0-04, P0-05, P0-06
- P0-B [chore] Project management -- Done when: one command rebuilds the labels, the milestones and the board from the plan files, a second run changes nothing, and a read-back check reports no difference between the board and the plan. -- Steps: P0-07, P0-08
- P0-C [adr] Founding decisions and threat model -- Done when: the ADR index lists the founding decisions as accepted, and the threat model built on them has an issue for every open risk. -- Steps: P0-09, P0-10, P0-11, P0-12, P0-13, P0-14
- P0-D [ci] CI security gates -- Done when: a pull request in any repository runs the shared gates from one pinned source, and a seeded violation of each gate fails the build. -- Steps: P0-15, P0-16, P0-17, P0-18, P0-19
- P0-E [chore] Developer environment and data guard -- Done when: a fresh checkout on Windows lints, tests and starts the app with one command each, and CI refuses real account data in any repository. -- Steps: P0-20, P0-21

## Steps

### clouddecks

- P0-01 [chore p2 S] Repository files: LICENSE, NOTICE, SECURITY.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, CODEOWNERS -- Done when: every repository has them under Apache-2.0, and each SECURITY.md points to GitHub private vulnerability reporting.
- P0-02 [chore p2 S] .gitattributes and .gitignore -- Done when: in every repository LF is enforced for every text file except PowerShell scripts, binaries are marked, keys, .env files, AWS config files and build output are ignored, and a fresh checkout on Windows with autocrlf=true keeps LF.
- P0-03 [security p1 S] Repository security settings -- Done when: in every repository private vulnerability reporting, secret scanning with push protection and Dependabot alerts are on; the default Actions token is read-only; and workflows from fork pull requests need approval.
- P0-04 [chore p1 S] Commit signing -- Done when: an SSH signing key is registered on GitHub, git signs every commit, and commits show as Verified.
- P0-05 [security p1 M] Rulesets for main and release tags -- Done when: in every repository main requires pull requests, required status checks and signed commits, allows merge commits only, and blocks force pushes and deletion; only the owner can create or delete v* tags; enabled after P0-04.
- P0-06 [chore p3 S] Issue and pull request templates -- Done when: every repository has feature, bug, spike and ADR issue templates, plus a pull request template with the definition-of-done checklist.
- P0-07 [chore p2 S] Labels, milestones and public project board -- Done when: the type, area, priority and phase labels and the milestones with their due dates exist in every repository that has steps for them, and one public project lists every step of every repository under its task and the epic of its module, with an assignee, a priority, a phase, a track, an estimate and start and target dates, and with the saved views of docs/BOARD.md.
- P0-08 [chore p2 M] Plan scripts and read-back check -- Done when: the sync, board and schedule scripts read docs/plan/*.md and file each step in the repository named by its heading, a second run changes nothing, and a read-back script compares every card on the board with the plan and ends with zero problems.
- P0-09 [adr p2 S] Record architecture decisions -- Done when: a template in `docs/adr/` and an index that CI keeps current, and the tool repositories link to it instead of keeping their own records.
- P0-10 [adr p2 S] One tool, one binary -- Done when: accepted: each tool is its own repository and its own executable, tools share state only through the standard AWS configuration files, and shared Go code is extracted into a module when a second tool needs it.
- P0-11 [adr p2 S] Stack: Go, Wails v3, React and TypeScript -- Done when: accepted, with the alternatives considered (Wails v2, Tauri, Electron, Python), the reasons (one binary, the AWS SDK in process, no runtime to install), and what a young major version asks for: an exact pinned version and a Go core that imports nothing from the framework.
- P0-12 [adr p1 S] Credentials stay in memory -- Done when: accepted: a tool reads the standard AWS files, never writes a credential or a token to disk, a log or a deck file, and what that costs (a new MFA code after a restart).
- P0-13 [adr p1 S] Distribution without a paid certificate -- Done when: accepted, with the options compared (unsigned with checksums and build attestations, a free open-source signing program, a paid certificate), what the user sees on Windows in each case, and the choice for v0.1.0.
- P0-14 [security p1 L] Threat model v1 -- Done when: `docs/threat-model/` has the system context, trust boundaries, data flows and a STRIDE table for the credentials in memory, deck files written by someone else, the local listening port, the webview and the release channel, linked to RISKS.md, with an issue for every open risk.
- P0-15 [ci p1 L] CI skeleton -- Done when: reusable workflows here run actionlint, zizmor, gitleaks, an ASCII guard, a Conventional Commits check and dependency review; every repository calls them on every pull request, pinned by commit SHA; every action is pinned by commit SHA; workflow permissions default to read-only; no workflow uses pull_request_target, and untrusted input never reaches a run step directly.
- P0-16 [ci p2 M] Code scanning -- Done when: CodeQL for Go and TypeScript, govulncheck and gosec run on pull requests and weekly in `hopdeck`, enabled when its first code lands.
- P0-17 [ci p2 S] Dependabot -- Done when: in every repository action SHAs, Go modules and npm dependencies get grouped weekly pull requests.
- P0-18 [ci p3 S] OpenSSF Scorecard -- Done when: a weekly run in every repository publishes results to code scanning and a README badge, and the README notes the checks a new single-maintainer repository cannot pass yet.
- P0-19 [ci p2 M] License check and notices -- Done when: Go and npm dependency licenses are checked against an allow list in CI, and the third-party notices that ship next to the binary are generated from the same data.
- P0-21 [security p1 S] Real-data guard -- Done when: CONTRIBUTING states that issues, fixtures, screenshots and logs never contain a real account ID, instance ID, host name or ARN, and a shared CI check fails in every repository on a 12-digit account ID or an access key ID outside the documented examples.

### hopdeck

- P0-20 [chore p1 M] Developer tooling -- Done when: the Go, Node and Wails versions are pinned, a Taskfile has lint, test, dev and build tasks, pre-commit hooks run formatting and gitleaks, and a fresh checkout on Windows opens the app window with `task dev`.

## Schedule

Planned weeks. Each step starts on the Monday shown and is due that Sunday; a task runs
from its first step to its last, and the module's epic and its milestone end with the
last week. Dates change here first, and the sync script moves the board.

| Week of | Steps |
|---|---|
| 2026-11-30 | P0-01, P0-02, P0-03, P0-04, P0-20 |
| 2026-12-07 | P0-05, P0-06, P0-07, P0-08, P0-09, P0-10, P0-11, P0-12, P0-13, P0-14, P0-15, P0-16, P0-17, P0-18, P0-19, P0-21 |

## Measures

- OpenSSF Scorecard score of each repository at the end of P0 (baseline, with the known
  caps noted).
