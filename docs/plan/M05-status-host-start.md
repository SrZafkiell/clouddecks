# M05 - Status and host start

Track: C. Phase: 1. Size: M. Depends on: M04. Unlocks: R1.

## Goal

One row per tunnel tells the whole story (credentials, jump host, agent, tunnel, port),
a stopped jump host can be started from the window when the policy allows it, and an
administrator gets a reference policy that grants exactly this and nothing more.

## Scope

- Jump host and agent status, and what to show without permission to see them.
- The status row.
- Starting a stopped jump host.
- The decision not to offer Stop.
- Reference IAM policies with simulator tests.

## Design notes

- Status needs ec2:DescribeInstances and ssm:DescribeInstanceInformation. A
  least-privilege user may have neither, and Connect must still work. Missing permission
  is shown as "not visible with your permissions", never as an error.
- Describe actions cannot be limited to one instance in IAM, so the reference policy
  offers them as an optional statement and says what they reveal.
- Start is an optional statement too: ec2:StartInstances limited by a tag on the jump
  host. The deck file says whether to offer the button.
- No Stop button. A jump host is shared, and one person's Stop would cut everyone
  else's tunnels. Stopping an idle host belongs to the AWS side, and the admin guide
  shows one way to do it.
- Colors never carry a state alone; every state also has a word.

## Tasks

- M05-A [feature] Status in one place -- Done when: each tunnel row shows credentials, jump host, agent, tunnel and local port in words and colors, refreshed on its own, and a user without describe permissions sees "not visible" and can still connect. -- Steps: M05-01, M05-02, M05-03
- M05-B [feature] Start a stopped jump host -- Done when: a stopped jump host is started from the window when the deck file and the policy allow it, the tunnel connects once the agent is online, and the decision not to offer Stop is an accepted ADR. -- Steps: M05-04, M05-05
- M05-C [docs] Reference IAM policies -- Done when: the repository has a user policy that allows a port forward through one jump host and nothing else, with optional status and start statements, and a simulator run on the lab shows every allowed and every denied case as expected. -- Steps: M05-06, M05-07

## Steps

### hopdeck

- M05-01 [feature p1 M] Jump host and agent status -- Done when: the instance state and the agent's ping status are read per tunnel, targets by tag are resolved to one instance (none or several is a clear error), and the refresh backs off when AWS throttles.
- M05-02 [feature p1 S] Status without permissions -- Done when: a denied describe call shows "not visible with your permissions" for that item, is not retried on every refresh, and Connect stays available.
- M05-03 [feature p2 M] Status row -- Done when: each row shows the five items with a word and a color, the detail pane explains each state, and the component tests cover every combination that the state machine can produce.
- M05-04 [feature p2 M] Start the jump host -- Done when: a stopped host shows Start when the deck file allows it, the row follows pending, running and agent online, the tunnel then connects, and a denied start says which permission to ask for.
- M05-05 [adr p2 S] No Stop in the app -- Done when: accepted, with the reason (a shared host) and the alternative on the AWS side.
- M05-06 [docs p1 M] Reference user policy -- Done when: `docs/iam/` has the policy as JSON with a line-by-line explanation: StartSession on one instance and the two documents, terminate and resume on the user's own sessions only, and the optional status and start statements.
- M05-07 [test p1 M] Policy simulator tests -- Done when: a script runs the IAM policy simulator on the lab against the reference policy and prints a table of allowed and denied cases (a shell session, another instance, another user's session, another document among the denied ones), and the table is in the repository with the account's identifiers replaced by the documented examples.

## Schedule

Planned weeks. Each step starts on the Monday shown and is due that Sunday; a task runs
from its first step to its last, and the module's epic and its milestone end with the
last week. Dates change here first, and the sync script moves the board.

| Week of | Steps |
|---|---|
| 2026-12-21 | M05-01, M05-02, M05-03, M05-04, M05-05, M05-06, M05-07 |

## Measures

- Number of allowed and denied cases proved in the policy simulator.
