# Open Knowledge Format (OKF) compatibility

Hydrate's durable memory is portable by design. Rather than lock your memory in
a proprietary store, Hydrate can emit every long-lived artefact it produces as
an [Open Knowledge Format](https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf)
(OKF) v0.1 bundle: a git-forkable directory of typed markdown concept documents.
Background on the format is in Google Cloud's
[introduction to OKF](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing/).

This page covers what Hydrate exports, the on-disk shape, how Hydrate-specific
metadata is carried without breaking conformance, and how conformance is
checked.

## What Hydrate exports

| Surface | Command | Concept doc types |
|---|---|---|
| Autonomous wiki | `hydrate wiki curate`, verified with `hydrate wiki verify --okf` | Reference |
| Hydration packs | `hydrate pack create --okf`, loaded with `hydrate pack import` | Fact, Canon, Session Digest |
| Handover archive | `hydrate handover export --okf` (default output `.hydrate/handover-okf/`) | Handover |

Each surface writes a self-contained bundle directory that you can commit,
diff, fork or hand to any OKF-aware tool.

## The on-disk shape

A bundle is a directory containing:

- A root `index.md` that carries the bundle's `okf_version` in its frontmatter
  (and nothing else there). It acts as the navigation entry point.
- One markdown file per concept document. Each carries a small YAML frontmatter
  with a non-empty `type` (for example `Reference`, `Fact`, `Canon`,
  `Session Digest`, `Handover`), a `title`, and, where the document owns a
  meaningful time, an RFC3339 `timestamp`. The human-readable content is the
  markdown body.

Reserved file names (`index.md` for navigation, `log.md` for history) are
identified by basename at any depth and are exempt from the concept-document
rules.

## Carrying Hydrate metadata without breaking conformance

Hydrate tracks scalar metadata that the open format does not define: confidence,
content hash, tier, git provenance, and similar. Rather than invent
non-conformant top-level keys, Hydrate keeps this in an indented frontmatter
extension (a nested mapping under a Hydrate-owned key). OKF consumers preserve
the block as opaque data and otherwise ignore it, so the bundle stays valid OKF
and a Hydrate round-trip loses nothing.

## Conformance is enforced, not asserted

Hydrate ships an in-tree linter that checks every wiki and pack bundle against
the OKF v0.1 conformance rules on each curation. Beyond the in-tree checker, the
bundles have been verified to load under the upstream OKF reference parser and
to round-trip byte-for-byte through the backend, so the claim is interop that
has been run, not interop that is hoped for.

One honest nuance: the upstream `validate()` is stricter than the published
specification. It requires `description` and `timestamp` in addition to `type`.
Hydrate emits both wherever it owns the document, so its bundles satisfy the
specification and the stricter validator alike.

## A worked example

The [`okf/`](../okf/) directory in this repository is a small, complete OKF v0.1
bundle exported in Hydrate's shape (a root `index.md` plus typed Reference
documents about Hydrate). It passes `hydrate wiki verify --okf`. Read it to see
exactly what a Hydrate OKF export looks like.

## Why this matters

Memory you cannot read, fork or move is not really yours. OKF support means your
remembered context is plain text in an open, documented format: inspectable
today, loadable by other tools, and not stranded if you ever stop using
Hydrate.
