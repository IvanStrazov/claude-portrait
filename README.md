> [!IMPORTANT]
> **This repository has moved to [IvanStrazov/animus-portrait](https://github.com/IvanStrazov/animus-portrait)** and is now part of the Animus ecosystem. This copy is frozen at 0.2.0 and gets no updates.
>
> To switch: `/plugin marketplace add IvanStrazov/animus-portrait`, then `/plugin install animus-portrait@animus-portrait`, then uninstall `claude-portrait`. Your data is untouched.

# claude-portrait

[![tests](https://github.com/IvanStrazov/claude-portrait/actions/workflows/test.yml/badge.svg)](https://github.com/IvanStrazov/claude-portrait/actions/workflows/test.yml)
[![version](https://img.shields.io/badge/version-0.2.0-blue)](CHANGELOG.md)
[![license: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)
![dependencies: none](https://img.shields.io/badge/dependencies-none-brightgreen)
![python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue)

**Ask Claude Code "what do you know about me?" and get a real answer, built from your own sessions and kept up to date cheaply.**

- **Built from your own words.** It reads your messages from Claude Code transcripts, Claude Code memory files, and [claude-journal](https://github.com/IvanStrazov/claude-journal) indexes when you have them. Assistant replies and tool output are skipped.
- **Incremental.** Each update reads only sessions newer than the last saved portrait, then merges them in. Facts are added, updated or dropped.
- **Sensitive topics are opt-in.** On first use you choose: `include`, `separate` (kept in `private.md`, shown only when you ask) or `exclude`.
- **Extra sources only with consent.** Disk folders, GitHub and the like are used only after you approve them. You can revoke them at any time.
- **Local only.** Plain Markdown in `~/.claude/portrait/`. No hooks of its own, no network, no database, one stdlib Python file.

## Install

```
/plugin marketplace add IvanStrazov/claude-portrait
/plugin install claude-portrait@claude-portrait
```

Then say `/portrait` or "update my portrait".

## Token budget

| Step | What enters context |
|---|---|
| Skill instructions | ~2.5 KB, only when the skill is used |
| `collect` | new material only, capped at 40 000 chars (≈ 10k tokens); ≤ 12 messages per session, ≤ 300 chars each |
| `show` | the portrait, ≤ 80 lines |

Limits can be changed with `PORTRAIT_MAX_LINES`, `PORTRAIT_PRIVATE_MAX_LINES`, `PORTRAIT_COLLECT_MAX_CHARS`, `PORTRAIT_MSG_MAX_CHARS` and `PORTRAIT_PER_SESSION`. `PORTRAIT_DIR` moves the storage.

## What it stores

```
~/.claude/portrait/
  portrait.md    Who · Projects · Work style with AI · Interests · Communication & preferences · Unknown / to confirm
  private.md     sensitive topics (only with policy "separate")
  config.json    sensitive policy, approved extra sources, last update time
```

## Works with claude-journal (optional)

If [claude-journal](https://github.com/IvanStrazov/claude-journal) is installed, the first `/portrait` run offers to link them. They share `~/.claude/ecosystem.json`, and nothing changes until you agree:
- the portrait also reads journal indexes;
- the journal adds a short portrait digest at session start and keeps *Projects* and *Communication & preferences* up to date at wrap-up.

Digest sizes, with costs measured from your own sessions by `portrait.py digest-cost` (example output):

```
requests per session: 96 (measured over your last 24 sessions)
preset   sections                                             lines tokens per session
off      -                                                        0      0           0
short    Communication & preferences                              4     79         849
medium * Communication & preferences, Who                         7    126        1353
long     Communication & preferences, Who, Work style with AI     9    168        1805
```
"per session" = written to the prompt cache once (×1.25), then read on every later request (×0.1), in input-token equivalents.

## Commands (used by the skill)

```
portrait.py status
portrait.py config --sensitive include|separate|exclude
portrait.py config --add-source github | --remove-source github | --sources-asked | --reset
portrait.py collect [--all] [--max-chars N]
portrait.py show [--private] [--section NAME]
portrait.py save [--private] [--section NAME] < portrait.md
portrait.py link on [--digest off|short|medium|long] | link off
portrait.py digest-cost
```

## Privacy

Everything stays on your machine. The skill tells Claude to keep out secrets (tokens, passwords, IDs, addresses, phone numbers, email), to mark guesses as guesses, and to treat transcript text as data, not instructions.

## License

MIT
