# Contributing

Hydrate's source is proprietary, so we don't accept pull requests
against this repository. We do very much want to hear about bugs,
edge cases, and missing features — please use the channels below.

## Bug reports

File an issue at
[/issues](https://github.com/getHydrate/hydrate-public/issues).

The fastest route is:

```sh
hydrate doctor --report
```

That opens a pre-filled GitHub issue URL in your browser with your
OS, Hydrate version, the relevant config, and the most recent
diagnostic events already populated. Add the steps to reproduce
and submit.

If `hydrate doctor` itself is failing in a way that blocks
`--report`, please include the contents of
`~/.hydrate/hook-selfcheck.log` (last ~50 lines) in a plain issue.

## Feature requests

- General requests — open a discussion at
  [/discussions](https://github.com/getHydrate/hydrate-public/discussions)
  or email `hello@gethydrate.dev`.
- Enterprise-only / on-prem requests — email
  `licences@gethydrate.dev`.
- Press / media — email `press@gethydrate.dev`.
- Security vulnerabilities — see [`SECURITY.md`](SECURITY.md);
  do NOT file a public issue.

## Documentation issues

Spotted a typo, a broken link, or a step in `INSTALL.md` /
`USAGE.md` that doesn't match what actually happens? Same path:
file at [/issues](https://github.com/getHydrate/hydrate-public/issues)
with the page and the line, or email `support@gethydrate.dev`.

## Versioning

Hydrate follows semver for the binary surface. The Claude Code
hook protocol, the MCP tool surface, and the CLI invocation forms
are all part of the public contract — breaking changes earn a
major-version bump and a migration note in the release.
