# M03 - AWS sign-in

Track: C. Phase: 1. Size: L. Depends on: M01. Unlocks: M04, M05.

## Goal

Hopdeck uses the AWS profiles a person already has, shows whether each one works right
now and for how long, and renews it from the window (an MFA code or a browser sign-in)
without writing a credential anywhere.

## Scope

- Reading the standard AWS configuration files.
- An identity check per profile, with clear states.
- A fake AWS layer so tests need no account.
- MFA prompts, browser sign-in, the expiry clock.
- A test that no secret reaches the disk.

## Design notes

- Hopdeck reads `~/.aws/config` and `~/.aws/credentials` and never changes them.
  Creating and editing profiles is the job of the profile tool, a later deck.
- The AWS SDK for Go resolves a profile the way the AWS CLI does: static keys, a role
  with `source_profile`, `credential_process`, IAM Identity Center. Two cases need work
  of our own: a role that asks for an MFA code (the SDK calls back for the code) and an
  IAM user whose policy requires MFA (Hopdeck calls GetSessionToken with the code).
- Temporary credentials live in memory only (ADR P0-12). After a restart the person
  enters a new MFA code. For IAM Identity Center, Hopdeck reads the token the AWS CLI
  cached; when it has expired it runs the browser sign-in itself and keeps the new token
  in memory.
- Every AWS call goes through small interfaces (STS, EC2, SSM) with fakes behind them,
  so the state logic is tested without an account and the lab is kept for end-to-end
  proof.

## Tasks

- M03-A [feature] Profiles and identity -- Done when: the profiles of this computer are listed without changing any file, each mapped profile shows who it is in AWS or a precise reason why it cannot say, and the tests run on fakes. -- Steps: M03-01, M03-02, M03-03
- M03-B [feature] Renewal -- Done when: a profile that needs an MFA code or a browser sign-in is renewed from the window, the time left is always visible, and a test proves no credential was written to disk. -- Steps: M03-04, M03-05, M03-06, M03-07

## Steps

### hopdeck

- M03-01 [feature p1 M] Profile discovery -- Done when: the profiles in the AWS configuration files are listed with their kind (keys, role, Identity Center, process), AWS_CONFIG_FILE and AWS_SHARED_CREDENTIALS_FILE are honored, and the files are opened read-only.
- M03-02 [feature p1 M] Identity check -- Done when: each mapped profile shows the account and the caller it resolves to, or one of the states valid, expired, needs MFA, needs sign-in, or error with the reason, and the account ID can be masked on screen.
- M03-03 [test p1 M] Fake AWS layer -- Done when: STS, EC2 and SSM are called through interfaces, fakes can script success, denial, expiry and throttling, and the identity states of M03-02 are covered by tests that use no network.
- M03-04 [feature p1 L] MFA prompt -- Done when: a role profile with `mfa_serial` and an IAM user whose policy requires MFA both ask for the code in a prompt that names the profile and the account it is for, get temporary credentials, and keep them in memory only; a wrong code gives a clear message and a new try.
- M03-05 [feature p2 L] Identity Center sign-in -- Done when: a profile with an expired Identity Center token opens the system browser for the device sign-in, the tunnel list updates when it completes, and the token is kept in memory only.
- M03-06 [feature p1 M] Expiry clock -- Done when: each profile shows the time left on its credentials, warns before they expire, and offers Renew, which runs M03-04 or M03-05 as the profile needs.
- M03-07 [security p1 S] No secret on disk -- Done when: a test runs a sign-in and a renewal on fakes and then finds no access key, secret key, session token or MFA code in the settings, the logs, the decks folder or the crash output.

## Schedule

Planned weeks. Each step starts on the Monday shown and is due that Sunday; a task runs
from its first step to its last, and the module's epic and its milestone end with the
last week. Dates change here first, and the sync script moves the board.

| Week of | Steps |
|---|---|
| 2026-11-30 | M03-01, M03-02, M03-03 |
| 2026-12-14 | M03-04, M03-06, M03-07 |
| 2026-12-21 | M03-05 |

## Measures

- Which profile kinds were proved end to end in the lab (keys with MFA, role with MFA,
  Identity Center), listed in the README.
