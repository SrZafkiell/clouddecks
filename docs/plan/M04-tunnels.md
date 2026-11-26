# M04 - Tunnels

Track: C. Phase: 1. Size: L. Depends on: M02, M03. Unlocks: M05, R1.

## Goal

One click opens a Session Manager port forward through a jump host to a private
service, and it stays usable: the port is checked first, a dropped session comes back by
itself, an error says what to do, and nothing is left running when the app closes.

## Scope

- A spike that settles how the Session Manager session runs inside the app.
- The tunnel engine, its state machine, Connect and Disconnect.
- Reconnection and renewal.
- Proof on a lab account, with a long-running soak test.

## Design notes

- A Session Manager port forward has two halves: the StartSession API call, which the
  AWS SDK covers, and a WebSocket data channel with its own protocol, which the SDK does
  not cover. The AWS CLI hands that half to the separate `session-manager-plugin`
  program. The spike M04-01 decides between running that protocol in process (the
  plugin's own Go packages, or a third-party client) and shipping the plugin next to the
  executable. In process is the goal: the people this tool is for should not install
  anything else. The fallback keeps the release date safe.
- Two session documents are used: AWS-StartPortForwardingSessionToRemoteHost (a service
  behind the jump host, such as a database) and AWS-StartPortForwardingSession (a port
  on the jump host itself).
- The listener binds to 127.0.0.1 only, never to all interfaces, so a tunnel is not
  shared with the local network by accident.
- Session Manager ends a session after its idle timeout or its maximum duration; both
  are set by the account's administrator. "Renew" is therefore two things: the tunnel
  reconnects by itself while the credentials are valid, and the person is asked to renew
  the credentials (M03-06) when they are not.
- A tunnel is a small state machine (idle, checking, starting, connected, reconnecting,
  failed), and every state carries a reason in plain words. The window shows the state;
  it never guesses it.
- The lab code is generic Terraform. The account it runs on is not part of the project:
  nothing that identifies it (IDs, names, addresses, ARNs) enters the repository, an
  issue, a screenshot or a recording, and the real-data guard (P0-21) checks it. Every
  lab resource carries one tag, so nothing is left behind unnoticed.
- The app writes down the IDs of the sessions it opens (they are not secrets), so it can
  end them after a crash without permission to list sessions. Only one copy of the app
  runs at a time; a second copy would otherwise end the sessions of the first.
- Access errors are translated: the message names the missing permission and says what
  to ask the administrator for, because the user of this tool is usually not the person
  who wrote the policy.

## Tasks

- M04-A [spike] Session Manager inside the app -- Done when: a written result shows a port forward opened from Go on a machine without the AWS CLI and without the plugin installed, or shows why not, and the choice is an accepted ADR. -- Steps: M04-01, M04-02
- M04-B [feature] Connect and disconnect -- Done when: Connect on a tunnel checks the port, opens the forward and shows the local address to use, Disconnect ends the session in AWS, and closing the app leaves no session behind. -- Steps: M04-03, M04-04, M04-05, M04-06, M04-07
- M04-C [feature] Renew and recover -- Done when: a tunnel whose session was dropped comes back without a click while the credentials are valid, asks for a renewal when they are not, and every failure is explained in plain words. -- Steps: M04-08, M04-09, M04-10
- M04-D [test] Proof on a lab account -- Done when: a scripted run on the lab opens a tunnel with a least-privilege user, survives a forced drop and is torn down, and a soak test reports how long a tunnel stayed usable. -- Steps: M04-11, M04-12

## Steps

### hopdeck

- M04-01 [spike p1 L] Port forward from Go without the plugin installed -- Done when: time-boxed to one day, a prototype opens a remote-host port forward using the plugin's Go packages and using a third-party client, the result notes what worked, the binary size, the licenses and the maintenance state of each, and the fallback (shipping the plugin) is tried once.
- M04-02 [adr p1 S] Session Manager client -- Done when: accepted, with the options of the spike compared and the conditions under which the choice is revisited.
- M04-03 [feature p1 L] Tunnel engine -- Done when: the engine starts a session with either port-forwarding document, listens on 127.0.0.1 only, carries traffic both ways, ends the session in AWS on Disconnect, and is covered by tests on the fake layer.
- M04-04 [feature p1 M] Tunnel state machine -- Done when: the states idle, checking, starting, connected, reconnecting and failed and their transitions are one table in the code, each state carries a reason, and a table-driven test covers every transition.
- M04-05 [feature p1 M] Connect and Disconnect in the window -- Done when: each tunnel has one button that follows its state, a connected tunnel shows `127.0.0.1:port` with a copy button, and two tunnels can be connected at the same time.
- M04-06 [feature p1 S] Local port check -- Done when: a busy local port is found before the session starts, the message names the port, and the user can pick another port for this computer without editing the deck file.
- M04-07 [feature p2 M] Nothing left behind -- Done when: closing the app ends every session it started, after a crash the next start ends the sessions the app had recorded as open, and a second copy of the app shows the running one instead of starting.
- M04-08 [feature p1 M] Reconnect -- Done when: a session ended by the idle timeout, the maximum duration or a network change is reopened with backoff on the same local port while the credentials are valid, and the tunnel asks for a renewal when they are not.
- M04-09 [feature p2 S] Session history -- Done when: each tunnel shows since when it is connected, how many times it reconnected and why the last session ended.
- M04-10 [feature p2 M] Errors in plain words -- Done when: access denied (with the action), a stopped jump host, an agent that is offline, an expired token and a busy port each have their own message that says what to do or what to ask the administrator for, covered by tests.
- M04-11 [test p1 L] End-to-end proof on the lab -- Done when: a Terraform lab (a jump host with no inbound rule and no public address, a private target port, a least-privilege group) is built on an AWS account that stays outside the repository (every account-specific value comes from an untracked variables file, and the state is never committed), a script proves connect, traffic, a forced drop, reconnect and disconnect, the lab is destroyed, and the results are in the repository with every account ID, instance ID and host name replaced by the documented examples.
- M04-12 [test p2 M] Soak test -- Done when: one tunnel carries periodic traffic for eight hours on the lab, and the report counts the drops, the reconnects and the seconds the port was unusable.

## Schedule

Planned weeks. Each step starts on the Monday shown and is due that Sunday; a task runs
from its first step to its last, and the module's epic and its milestone end with the
last week. Dates change here first, and the sync script moves the board.

| Week of | Steps |
|---|---|
| 2026-11-30 | M04-01, M04-02, M04-03, M04-04, M04-05, M04-06 |
| 2026-12-14 | M04-07, M04-08, M04-09, M04-10 |
| 2026-12-21 | M04-11, M04-12 |

## Measures

- Time from Connect to a usable port.
- Drops, reconnects and unusable seconds in the eight-hour soak test.
- Size added to the executable by the session client.
