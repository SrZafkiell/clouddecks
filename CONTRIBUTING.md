# Contributing to Clouddecks

Thank you for your interest. Clouddecks is built in public by one maintainer, and the
way the work is planned and tracked is part of the project. This page explains the rules
that every repository of Clouddecks follows.

## Where the work is

- The plan of every module is in [docs/plan](docs/plan), the overview and the decisions
  in [docs/PLAN.md](docs/PLAN.md), and the risks in [docs/RISKS.md](docs/RISKS.md).
- The work is tracked on one public board across the repositories:
  https://github.com/users/SrZafkiell/projects/3. [docs/BOARD.md](docs/BOARD.md)
  explains how to read it.
- Each piece of work is a step: an issue in the repository that does the work, closed by
  one pull request.

Before you start on something larger than a typo, open an issue or comment on the step,
so that the work fits the plan.

## Never put real account data anywhere

Issues, pull requests, commits, fixtures, test data, screenshots, recordings and logs
never contain a real AWS account ID, access key, instance ID, host name, IP address or
ARN. Use the documented examples:

| What | Example |
|---|---|
| Account ID | `123456789012` |
| Access key ID | `AKIAIOSFODNN7EXAMPLE` |
| Instance ID | `i-1234567890abcdef0` |
| Host name | `db.example.internal` |

Mask the account ID on screen before you take a screenshot. A CI check refuses a 12-digit
account ID or an access key ID that is not one of the examples.

Security problems are reported privately; see [SECURITY.md](SECURITY.md).

## Branches, commits and pull requests

- Branch names: `<type>/<issue>-<short-name>`, for example `chore/8-repository-files`.
- Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/):
  `type(scope): subject`, with a subject of at most 72 characters and a body that
  explains why. The types are `feat`, `fix`, `docs`, `chore`, `ci`, `test`, `refactor`,
  `perf`, `build` and `revert`.
- One concern per commit: a fix, a feature, a generated file and a documentation sweep
  are separate commits, even in one pull request.
- Commits are signed (see [Signing your commits](#signing-your-commits)). `main` accepts
  only signed commits through a pull request, and pull requests are merged with a merge
  commit, so the signed commits stay on `main`.
- A pull request has the type and area labels and the milestone of its step, and names
  the step in its body: `Closes #N` when it finishes the step, `Refs #N` when it does
  not.
- Everything written to the repositories is plain ASCII: code, comments, documentation,
  commit messages and pull request text. CI checks it.

## Signing your commits

`main` accepts only signed commits. Git can sign with an SSH key, and the key can be the
one you already use for GitHub or a new one used only for signing:

```
ssh-keygen -t ed25519 -C "git commit signing" -f ~/.ssh/id_ed25519_signing
git config gpg.format ssh
git config user.signingkey ~/.ssh/id_ed25519_signing.pub
git config commit.gpgsign true
git config tag.gpgsign true
```

Then add the public key on GitHub as a signing key (Settings, SSH and GPG keys, New SSH
key, key type "Signing Key"). Your commits show as Verified when the commit email is one
of the verified emails of your account; the noreply address counts.

A key with a passphrase is loaded into an SSH agent once, so Git can sign without asking.
On Windows, use the agent that comes with Windows (the OpenSSH Authentication Agent
service) and point Git at the matching tool:
`git config gpg.ssh.program C:/Windows/System32/OpenSSH/ssh-keygen.exe`.

To check signatures locally, list the keys you trust in an allowed signers file and run
`git log --show-signature`:

```
git config gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

## Definition of done

A step is done when its "Done when" holds and:

- the tests pass, including the negative tests of anything that handles a credential or
  reads a deck file (what must be refused, and what must never be written);
- the documentation is updated;
- an ADR is written if a decision was made;
- the threat model and docs/RISKS.md are updated if a trust boundary or a risk changed;
- the published results are updated if a measured number changed.

## License

By contributing, you agree that your contributions are licensed under the Apache
License, Version 2.0, the license of this repository.

Everyone taking part follows the [Code of Conduct](CODE_OF_CONDUCT.md).
