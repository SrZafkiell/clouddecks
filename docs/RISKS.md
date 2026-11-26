# Risks and constraints

Known constraints of a desktop tool that handles cloud credentials and is handed to
people who are not developers, and how the plan handles each one. The list is reviewed at
the end of every milestone; the step IDs point to the plan files in [plan/](plan/).

## Credentials

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| The app holds temporary AWS credentials while it runs | Another program running as the same user could read them from memory | Never on disk; temporary credentials with the lifetime the account sets; a reference policy that limits what a stolen session can do. Accepted: a desktop app cannot defend a user account that is already compromised | P0-12, M03-07, M05-06 |
| A secret reaches a log, the settings or a bug report | Credentials end up in files with weaker protection | A logger that redacts, with a test per kind of secret; a test that searches every file the app writes; the diagnostics copy is built from the redacted log | M01-05, M03-07 |
| The app asks for an MFA code | People learn to type codes into any window that asks | The prompt names the profile and the account it is for; an Identity Center sign-in happens in the system browser, never in the app's webview | M03-04, M03-05 |
| The app changes the AWS configuration files | The AWS CLI and other tools break, or behave differently | Hopdeck opens them read-only; editing profiles is a later deck with its own threat model | M03-01, P0-10 |
| Long-lived access keys in the credentials file | The tool makes a weak practice more convenient | Supported because they exist; the guides say which kinds of sign-in to prefer and why | P1-08, P1-09 |

## Deck files

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| A deck file is written by someone else | A hostile or mistaken file points a tunnel at another host, or tries to exhaust the parser | A strict schema, unknown keys refused, limits on size and nesting, no YAML aliases, a fuzz test; a preview in plain words before the file is added | M02-02, M02-03, M02-05 |
| A deck file is changed on its way (chat, email, a shared folder) | The user connects to a host the administrator never listed | In v0.1.0 the preview; in Phase 2 signatures, and a notice for a file that is unsigned or changed | M02-05, M07-02, M07-03 |
| Someone puts a secret in a deck file | Credentials travel by chat and email | A field that looks like a credential is refused with a message; the authoring guide says what never goes in | M02-03, M02-07 |
| A profile name means something else on another computer | A tunnel opens with the wrong account | The name in the deck file is only a suggestion; the mapping is local, and the row shows the account the profile resolves to | M02-06, M03-02 |

## Tunnels and the session client

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| The AWS SDK does not cover the Session Manager data channel | Running it in process means relying on the plugin's own Go packages or on a third-party client, which can break or stop being maintained | A time-boxed spike that compares both, with their licenses and maintenance state; an ADR with the conditions to revisit; the client behind an interface; the fallback of shipping the plugin | M04-01, M04-02, M04-03 |
| A local listening port can be used by any program on the computer | Another local process uses an open tunnel | Loopback only. Accepted for local processes, as with any port forward; Disconnect and closing the app end the session | M04-03, M04-07 |
| Sessions are left open after a crash | An access path stays open until the account's timeout | The app records the sessions it opens and ends them at the next start; only one copy of the app runs; the account's idle timeout is the backstop | M04-07 |
| Session Manager ends sessions (idle timeout, maximum duration) | A database client loses its connection in the middle of work | Reconnect on the same port with backoff, a visible history of drops, an eight-hour soak test with the numbers published | M04-08, M04-09, M04-12 |
| AWS error messages are written for administrators | A user cannot tell what to ask for | Each known failure has a message in plain words that names the missing permission | M04-10 |
| A jump host is shared | One person's Stop would cut everyone else's tunnels | Start only, behind an optional policy statement limited by a tag; no Stop in the app; stopping an idle host belongs to the AWS side | M05-04, M05-05, M05-06 |
| Describe calls cannot be limited to one instance in IAM | Either users see more of the account than they need, or the tool looks broken | Status is optional: "not visible with your permissions" is a normal state, and the policy explains what the optional statement reveals | M05-02, M05-06 |

## The window and the framework

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| The window is a browser engine inside the app | Script injection in the page could call the Go core | Local content only, a content security policy, navigation refused, an explicit list of exposed methods; text from a deck file is shown as text | M01-03, M02-05 |
| The framework has had advisories about which page may call Go code | A flaw there turns any injected content into code execution | No remote content is ever loaded; the framework version is pinned to a patched release; advisories arrive through Dependabot alerts | M01-03, P0-03, P0-17, P0-20 |
| Wails v3 became stable in October 2026 | A young major version can still change or break | An exact pinned version; a core that imports nothing from the framework, so a fallback to v2 would only touch the window layer; a check that keeps it that way | P0-11, M01-07 |
| Linux distributions ship different versions of GTK and WebKitGTK | The window does not start on some systems | A spike before the pipeline is written; a list of the distributions that were proved | M06-01, M06-03 |

## Distribution

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| The executable has no paid code-signing certificate | Windows shows a SmartScreen warning; people learn to click through warnings, and a fake build would look the same | A decision record that compares the options; a checksum and a build attestation, with one command to verify; the guide shows the screen and the check to do first | P0-13, P1-05, P1-06, P1-08 |
| The executable is passed from hand to hand | Old or altered builds circulate | The version and the commit are in the file and on the About screen; the guide says to download from the release page and verify | P1-02, P1-05, P1-08 |
| A dependency with a known vulnerability ships in a release | People run vulnerable code next to their cloud credentials | Dependency review on pull requests, weekly scans, a vulnerability gate in the release job, a bill of materials per release | P0-15, P0-16, P1-04, P1-07 |
| An action or a build tool is replaced upstream | A poisoned release | Actions pinned by commit SHA, pinned tool versions, hosted runners, a release job that runs only on a protected tag, build provenance | P0-05, P0-15, P1-01, P1-05 |
| No update channel in v0.1.0 | People stay on a version with a known flaw | The version is visible in the window and releases are announced on the release page; in-app updates are a later module | P1-02, Later modules |

## Repositories and delivery

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| Single maintainer | Reviews are self-reviews | Required checks, signed commits and scanners; the cap is noted in the Scorecard results | P0-05, P0-18 |
| The first code lands before every CI gate exists | A week of commits that not every scanner has seen | The basic rules come before the first code (license, security settings, secret scanning with push protection, signed commits); the other gates follow in the second week and then run on everything already merged | P0-01, P0-03, P0-04, P0-15, P0-16 |
| Rules, scanners and settings exist once per repository | One repository drifts from the standard | Reusable workflows called by every repository, one step per rule that covers them all, a Scorecard run in each | P0-03, P0-05, P0-15, P0-18 |
| A reusable workflow is trusted by every repository | A bad change to it weakens every build | Callers pin it by commit SHA; a change to it goes through a pull request with the same gates | P0-15 |
| Rebase merges create commits without signature verification | Required signed commits and per-concern commits conflict | Merge commits: branch commits are signed by the author, merge commits by GitHub | P0-05 |
| Public repository: untrusted pull requests | Secrets or runners exposed | GitHub-hosted runners only; no `pull_request_target`; actionlint and zizmor; write permissions only in the release job, on a protected tag | P0-15, P1-01 |
| Git for Windows converts LF to CRLF on checkout by default | Scripts and generated files differ between machines | `.gitattributes` forces LF (CRLF only for PowerShell scripts) | P0-02 |
| Issues are spread across repositories | Work is lost between them | The plan files in the home repository are the single source; the sync files each issue in its repository, one board lists them all, and a read-back check compares the board with the plan | P0-07, P0-08 |
| The hours per step are estimates, not measurements | Dates slip | The sizes are corrected after the first week and the rest of the plan is re-dated by script; a late card gets a new date, not an early close | P0-08 |

## The lab and real data

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| The lab runs on a real AWS account | An account ID, an instance ID, a host name or an address reaches the repository, an issue or an image, for good | Every account-specific value comes from an untracked file and the Terraform state is never committed; results are published with example identifiers; a CI guard on account IDs and access key IDs; secret scanning with push protection | P0-03, P0-21, M04-11 |
| Lab resources are left running | Cost, and an access path nobody watches | The end-to-end step is done only when the lab is destroyed; every lab resource carries one tag | M04-11 |
| Screenshots and recordings are taken on a working setup | A real name or identifier in a published image | The example deck file, the account ID masked on screen, made-up names, no real instance ID or address in view | M03-02, P1-08, R1-03 |
