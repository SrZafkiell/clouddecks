# Clouddecks

Small desktop tools for people who work with cloud accounts every day. Each tool is a
deck: one job, one window, one executable. Take the one you need.

> Status: planning. See [the plan](docs/PLAN.md); work is tracked in issues, milestones
> and one [project board](https://github.com/users/SrZafkiell/projects/3) across the
> repositories.

## Decks

| Deck | What it does | State |
|---|---|---|
| [Hopdeck](https://github.com/SrZafkiell/hopdeck) | Opens AWS Session Manager tunnels through a jump host to private services, such as a database, from a shared deck file | Planned: v0.1.0 for Windows, then Linux |
| Profiles deck (working name) | Creates and edits AWS profiles, and signs in, from a window | Later |

## Hopdeck in short

- An administrator writes a small deck file that lists the tunnels of a team and hands
  it out. The file holds no secret.
- A user drops the file into a folder, picks one of their AWS profiles and clicks
  Connect. One row per tunnel shows the credentials, the jump host, its agent, the
  tunnel and the local port.
- The jump host has no inbound rule, no public address and no SSH key. Access is an IAM
  identity and a policy that allows a port forward through that one host and nothing
  else.
- A session that drops comes back by itself. When the credentials expire, the window
  asks for an MFA code or opens the browser sign-in.
- The goal is that there is nothing else to install: no AWS CLI, no plugin.

## Repositories

| Repository | What it holds |
|---|---|
| clouddecks (this one) | The plan, decisions, threat model and risk register; the CI workflows shared by the decks; the plan and board scripts |
| [hopdeck](https://github.com/SrZafkiell/hopdeck) | The Hopdeck app (Go, Wails, React and TypeScript), its reference IAM policies and its guides |

A deck never needs another deck installed. Decks share state only through the standard
AWS configuration files, the same ones the AWS CLI uses.

## Engineering standards

- The plan is kept as files and synced to GitHub as epics, tasks and steps: every step
  has an owner, a priority, an estimate and a planned week on a public project board
  with a roadmap ([how to read it](docs/BOARD.md)); every change goes through a pull
  request, in every repository.
- CI security gates shared by the repositories: secret scanning, CodeQL, govulncheck,
  dependency review, OpenSSF Scorecard.
- Signed commits; releases with a checksum, a bill of materials and build provenance
  that one command verifies.
- An architecture decision record for every significant decision, a threat model and a
  [risk register](docs/RISKS.md).
- No credential is ever written to disk, and a test proves it.
- Numbers come from tests and scripts, recorded with hardware and versions.
- No real account data: fixtures, screenshots and recordings use example identifiers.

## License

Apache-2.0. See [LICENSE](LICENSE).

Clouddecks is an independent project. It is not affiliated with, or endorsed by, Amazon
Web Services.
