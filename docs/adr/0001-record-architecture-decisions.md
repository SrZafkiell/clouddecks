# 0001. Record architecture decisions

- Status: Accepted
- Date: 2026-11-27
- Step: P0-09
- Decided by: the maintainer

## Context

Clouddecks is built in public by one maintainer. Its decisions are made while planning
and while building: which stack, how the tools are cut, what a deck file may contain.
Without a written record, the reason for a choice lives only in the head of the person
who made it, a later change cannot tell whether a constraint still holds, and a reader
of the code sees what was done but not why.

The plan already lists the decisions it expects, each tied to a step (the decisions
table of [PLAN.md](../PLAN.md)). What was missing is the place and the form for the
reasoning.

## Options

- No records, the plan and the pull requests only. No overhead, but the reasoning is
  scattered across pull request threads and is lost when one is hard to find.
- A wiki. Easy to edit, but it lives outside the repository, is not reviewed in pull
  requests and drifts from the code.
- Short records in the repository, one decision per file, in the format Michael Nygard
  described in 2011 (context, decision, consequences), reviewed in pull requests.

## Decision

Every decision that shapes more than one step is recorded in `docs/adr/` of this
repository, one numbered file per decision, written from [the template](template.md)
with the context, the options, the decision and the consequences, and the plan step that
asked for it. The tool repositories link here instead of keeping their own records. A
record is accepted in the pull request that adds it and is never rewritten afterwards: a
change of mind is a new record that supersedes the old one.

## Consequences

- Each decision costs a short document and a pull request, which is the point: a
  decision that cannot be written down in a page is not ready to be made.
- The decisions table of PLAN.md and the index of this folder must stay in step; until
  CI checks the index (with the gates of P0-15), the same pull request updates both by
  hand.
- A later contributor can see why the project looks the way it does, and which
  conditions would make a decision worth revisiting.
