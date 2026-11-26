# M01 - App shell

Track: B. Phase: 1. Size: M. Depends on: P0. Unlocks: M02, M03, P1.

## Goal

A Hopdeck window that starts from one executable, shows where tunnels will live, keeps
its settings and logs in the right place, and has a test harness, before any AWS code.

## Scope

- The Wails project with a React and TypeScript front end.
- The layout: tunnel list, detail pane, status bar, empty state.
- A locked-down webview.
- Application folders, settings and logs.
- Go and front-end test setup.

## Design notes

- Wails v3, pinned to an exact version. It became stable in October 2026, so the Go core
  (deck files, profiles, sign-in, tunnels) imports nothing from the framework and is
  tested without a window. Only the thin window layer knows Wails, and a check keeps it
  that way (M01-07).
- Wails v3 has a system tray, a single-instance lock and an updater. v0.1.0 uses the
  lock (M04-07). The tray and the updater are later modules, so closing the window ends
  the tunnels.
- The webview shows local files only. No remote content, no navigation away, a strict
  content security policy, developer tools off in release builds, and the Go methods the
  front end may call are an explicit list.
- Logs are for the person at the keyboard and for a bug report. A redaction layer sits
  in the logger itself, so a later log line cannot leak a token by accident.
- Settings carry a schema version from the first release, so a later version can migrate
  them.

## Tasks

- M01-A [feature] Running window -- Done when: one executable opens a window with the tunnel list, the detail pane and the status bar, shows an empty state that says how to add a deck file, and loads nothing from the network. -- Steps: M01-01, M01-02, M01-03
- M01-B [feature] Settings and logs -- Done when: settings and logs live in the per-user application folder, survive a restart, and a diagnostics copy contains no secret. -- Steps: M01-04, M01-05, M01-06
- M01-C [test] Test harness -- Done when: Go and front-end tests run with one command and in CI, with the race detector on and coverage reported. -- Steps: M01-07, M01-08

## Steps

### hopdeck

- M01-01 [feature p1 M] Wails project with React and TypeScript -- Done when: `task dev` opens the window with live reload and `task build` produces a single executable that starts on a machine without Go or Node.
- M01-02 [feature p1 M] Layout and empty state -- Done when: the window has the tunnel list, the detail pane and the status bar, works with the keyboard alone, and with no deck file it explains in two sentences how to add one.
- M01-03 [security p1 S] Webview lockdown -- Done when: the content security policy allows local assets only, navigation and new windows are refused, developer tools are off in release builds, and the Go methods exposed to the front end are an explicit list covered by a test.
- M01-04 [feature p2 S] Application folders and settings -- Done when: the settings file and the decks folder live under the per-user application folder on Windows and Linux, the settings file has a schema version, and a damaged file is set aside with a message instead of a crash.
- M01-05 [feature p2 M] Logs with redaction -- Done when: a rotating local log exists, the logger removes access keys, secret keys, session tokens and MFA codes before writing, a test proves each case, and "Copy diagnostics" puts the version, the system and the last log lines on the clipboard.
- M01-06 [feature p3 S] About screen -- Done when: it shows the version, the commit and the third-party notices.
- M01-07 [test p1 M] Go test setup -- Done when: `task test` runs the Go tests with the race detector, CI runs the same on Windows and Linux, coverage is reported on the pull request, and a check fails when a package of the core imports the window framework.
- M01-08 [test p2 M] Front-end test setup -- Done when: component tests run against mocked Go bindings with one command and in CI, and the layout of M01-02 has a test for the empty state and for a list of tunnels.

## Schedule

Planned weeks. Each step starts on the Monday shown and is due that Sunday; a task runs
from its first step to its last, and the module's epic and its milestone end with the
last week. Dates change here first, and the sync script moves the board.

| Week of | Steps |
|---|---|
| 2026-11-30 | M01-01, M01-02 |
| 2026-12-14 | M01-03, M01-04, M01-05, M01-06, M01-07, M01-08 |

## Measures

- Size of the Windows executable and cold start time to a usable window.
