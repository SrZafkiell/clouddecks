# 0003. Stack: Go, Wails v3, React and TypeScript

- Status: Accepted
- Date: 2026-11-26 (decided), recorded 2026-11-27
- Step: P0-11
- Decided by: the maintainer

## Context

Hopdeck, the first tool, is a desktop app for people who should not have to install
anything else: no AWS CLI, no Session Manager plugin, no runtime. It calls AWS (STS, EC2,
Systems Manager) and keeps temporary credentials in memory only. It ships for Windows
first and Linux next; macOS waits for a decision on the Apple developer program.

Facts checked on 2026-11-26: Wails v3.0.0 became the stable release on 2026-10-26 and
v3.1.0 followed on 2026-11-01; it needs Go 1.26 or newer and uses the system webview
(WebView2 on Windows, GTK 4 with WebKitGTK 6.0 on Linux). Wails v2 is still patched
(v2.17.1). Wails published several security advisories in October and November 2026,
several of them about which pages may call Go code.

## Options

- Go with Wails v2. Mature and well documented, but the older major version; new work
  goes into v3.
- Go with Wails v3. The current major version, with a single-instance lock, a system
  tray and an updater built in; young as a stable release.
- Tauri (Rust core). Small binaries and a strong security model, but the core would be
  Rust, away from the AWS SDK for Go and from the language of the rest of the toolkit.
- Electron. The most widely used desktop stack, but every app ships its own Chromium
  (well over 100 MB) and a Node runtime.
- Python with a webview or a GUI toolkit. Quick to write, but packaging a single
  executable is fragile and start-up is slow.

## Decision

The core is Go, with the AWS SDK for Go v2 in process. The window is Wails v3, pinned to
exactly one version (v3.1.0 today) in `go.mod`, with the matching Wails CLI pinned in the
tool's Taskfile. The user interface is React with TypeScript, built by Vite and embedded
in the executable. The Go core (deck files, profiles, sign-in, tunnels, status) imports
nothing from Wails, and a check enforces that (M01-07), so the framework can be upgraded
or replaced without touching the logic. The window loads local content only (M01-03).

## Consequences

- One executable of about 10 MB for the first window, with the front end embedded and
  nothing to install on Windows 11, which includes WebView2. Older Windows versions may
  need the WebView2 runtime.
- A young major version: upgrades of Wails are deliberate pull requests, its security
  advisories are watched, and the pinned version only moves after a review.
- The core is tested without a window, and the window is a thin layer over it.
- Revisit if Wails v3 stops being maintained, if a security problem in the webview
  bridge cannot be contained by loading local content only, or if macOS support needs
  something only another stack offers.
