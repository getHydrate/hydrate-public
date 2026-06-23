# Hydrate slash commands (MIT)

Two Claude Code slash commands that apply Hydrate's YAGNI concision discipline to
a single session, without running an orchestration fleet. Unlike the rest of this
repository, the files in this directory are MIT-licensed (see
[`LICENSE`](LICENSE)), so you are free to use, copy, modify and redistribute them.

- **`hydrate-yagni.md`** disciplines the *build*: carry out a task under the same
  "build only what is needed" directive Hydrate injects into its develop-mode
  implementer prompts. A single concision directive cut a measured single-shot
  build by about 78% in tokens and cost at equal quality.
- **`hydrate-yagni-spec.md`** disciplines the *spec*: author a spec, plan, design
  doc or acceptance criteria under the concision rubric Hydrate's design critic
  enforces, so you write a tight spec the first time. It cuts prose but commits
  the expensive-to-change boundary decisions, which is the opposite emphasis to
  the build directive.

Both commands call `hydrate yagni-block` for the canonical directive text when the
Hydrate binary is present, and fall back to a verbatim copy of that directive when
it is not, so they work standalone.

## Install

Drop the files into your Claude Code commands directory:

```sh
mkdir -p ~/.claude/commands
curl -fsSL -o ~/.claude/commands/hydrate-yagni.md \
  https://raw.githubusercontent.com/getHydrate/hydrate-public/main/slash-commands/hydrate-yagni.md
curl -fsSL -o ~/.claude/commands/hydrate-yagni-spec.md \
  https://raw.githubusercontent.com/getHydrate/hydrate-public/main/slash-commands/hydrate-yagni-spec.md
```

Then use them in any Claude Code session:

```
/hydrate-yagni build the X endpoint
/hydrate-yagni-spec write a spec for the X endpoint
```

Called with no task, each command prints its directive and tells you to add a task
after the command.
