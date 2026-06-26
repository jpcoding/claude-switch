# claude-switch

Run multiple Claude Code accounts on one machine — with third-party API provider support.

A single-file Bash tool that manages isolated Claude Code profiles. Each profile
gets its own `~/.claude-<name>/` directory (separate credentials and session data),
while shared config (`skills`, `CLAUDE.md`, `plugins`, `projects`) is symlinked from
`~/.claude/`. Profiles can authenticate via Anthropic OAuth **or** point at any
third-party Anthropic-compatible API (DeepSeek, Volcano Engine / Ark, OpenRouter,
GLM, …).

## Requirements

- [`claude`](https://docs.anthropic.com/en/docs/claude-code) (Claude Code CLI) in `PATH`
- [`gum`](https://github.com/charmbracelet/gum) — for interactive menus (`brew install gum`)
- `python3` — for reading/writing provider config
- `bash`

## Install

```bash
install -m 755 claude-switch ~/.local/bin/claude-switch
# ensure ~/.local/bin is on your PATH
```

## Usage

```bash
claude-switch                     # interactive menu
claude-switch <profile-name>      # quick-launch a profile
claude-switch create [name]       # create a profile (Anthropic OAuth / third-party)
claude-switch config [name]       # (re)configure a profile's API
claude-switch list                # list profiles and auth status
claude-switch delete [name]       # delete a profile
claude-switch help                # show help
claude-switch version             # show version
```

### Examples

```bash
claude-switch create deepseek     # create + configure a DeepSeek API profile
claude-switch create hs           # create + configure a Volcano Engine (Ark) profile
claude-switch deepseek            # launch the deepseek profile
claude-switch config deepseek     # change URL / key / models later
claude-switch list                # show all profiles
```

## How it works

- Each profile lives in `~/.claude-<name>/` and is launched with
  `CLAUDE_CONFIG_DIR` pointing at it, so credentials and history stay isolated.
- Shared items (`skills`, `CLAUDE.md`, `plugins`, `projects`) are symlinked from
  `~/.claude/` so you maintain them once.
- `create` adds a `claude-<name>` alias to `~/.bashrc` for convenience.

### Third-party API profiles

Choosing **Third-party API** during `create` (or running `config`) stores a
`.provider.json` in the profile directory and wires up the relevant environment
variables when launching:

`ANTHROPIC_BASE_URL`, `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_MODEL`,
`ANTHROPIC_DEFAULT_OPUS_MODEL`, `ANTHROPIC_DEFAULT_SONNET_MODEL`,
`ANTHROPIC_DEFAULT_HAIKU_MODEL`, `CLAUDE_CODE_SUBAGENT_MODEL`.

You can set one model for everything, or per-tier models (Haiku/Sonnet/Opus +
subagent) for task routing. No OAuth login is needed — it works immediately.

> API keys are stored locally in each profile's `~/.claude-<name>/.provider.json`.
> They are **never** committed to this repository.
