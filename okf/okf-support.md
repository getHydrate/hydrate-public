---
type: Reference
title: Open Knowledge Format support
description: Hydrate exports its durable memory as Open Knowledge Format v0.1 bundles, with conformance enforced by an in-tree linter and verified against the upstream reference parser.
timestamp: 2026-06-21T00:00:00Z
hydrate:
  tier: canon
  confidence: 1.0
---

# Open Knowledge Format support

Every long-lived artefact Hydrate produces can be emitted as an Open Knowledge
Format (OKF) v0.1 bundle: a git-forkable directory of typed markdown concept
documents. The autonomous wiki exports as Reference documents (`hydrate wiki
curate`, verified with `hydrate wiki verify --okf`), hydration packs export
Fact, Canon and Session Digest documents (`hydrate pack create --okf`), and the
handover archive exports Handover documents (`hydrate handover export --okf`).

Each concept document carries a small YAML frontmatter (`type`, `title`,
`description`, and an RFC3339 `timestamp`), with the human-readable content in
the body. Hydrate-specific scalar metadata sits in an indented frontmatter
extension that OKF consumers preserve as opaque data and otherwise ignore, so a
bundle stays valid OKF while losing nothing on a round-trip. This bundle is an
example of that output. The format is described at
https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf.
