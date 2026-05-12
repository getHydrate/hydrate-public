# Security policy

## Reporting a vulnerability

Email **security@gethydrate.dev** with the details. We will respond within
two working days (Europe/London business hours).

For urgent issues — actively exploited vulnerabilities, credential leaks,
supply-chain compromise — mark the email subject `URGENT` and we will
prioritise within the same working day.

Please do not file public GitHub issues for security reports. We will
coordinate a fix and a coordinated disclosure timeline with you directly.

### What to include

- A description of the vulnerability and the affected component
  (`hydrate-server`, `claude-context`, `claude-capture`, `hydrate-mcp`,
  the Homebrew formula, the Windows MSI, or `gethydrate.dev`).
- Steps to reproduce.
- Impact assessment (data exposed, privilege gained, etc.).
- Your preferred credit line for the eventual disclosure (or "anonymous").

We do not currently run a paid bug-bounty programme. We do credit
researchers in release notes for valid reports.

## Official sites

The only canonical Hydrate website is **`https://gethydrate.dev`**. The
following alternative domains, where they exist, redirect (HTTP 301) to
`gethydrate.dev`:

- `gethydrate.com`
- `gethydrate.io`
- `hydrate.app`
- `hydrate.ai`
- Common typo guards: `gehydrate.dev`, `hyrdate.dev`, `hydate.dev`,
  `hyrate.dev`

`hydrate.dev` is **not** owned by Sedasoft Ltd and is **not** an official
Hydrate site. Anything served from that hostname is not from us. (Verified
2026-05-07.)

If you encounter a site claiming to be Hydrate that is not on the list
above, please report it to **security@gethydrate.dev** so we can coordinate
takedown.

## Verifying installers

The official install path is:

```
curl -fsSL gethydrate.dev/install | sh
```

Hosted on the canonical domain, served over HTTPS with a Let's Encrypt
certificate. The Homebrew tap is `seamuswaldron/hydrate` (GitHub:
`SeamusWaldron/homebrew-hydrate`). Release artefacts are signed; the public
key is published in the GitHub release notes.

If a third party offers a Hydrate binary that does not originate from one
of the channels above, treat it as untrusted.

## Supported versions

We support the latest minor release on `main` and the previous minor
release for security patches. Older versions are end-of-life — please
upgrade before reporting issues against them.

## Out of scope

- Vulnerabilities in upstream dependencies (file with the upstream
  project; we will track the fix here).
- Issues that require an attacker to already have local code execution on
  the user's machine.
- Theoretical attacks without a demonstrated impact path.
