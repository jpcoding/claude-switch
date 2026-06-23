# claude-switch

Run multiple Claude Code accounts on one machine — with third-party API provider support.

A single-file Bash tool that manages isolated Claude Code profiles. Each profile
gets its own `~/.claude-<name>/` directory (separate credentials and session data),
while shared config (`skills`, `CLAUDE.md`, `plugins`, `projects`) is symlinked from
`~/.claude/`. Profiles can authenticate via Anthropic OAuth, route through your
**Google Antigravity** subscription (Claude Opus 4.6 via a local proxy), **or** point
at any third-party Anthropic-compatible API (DeepSeek, Volcano Engine / Ark, OpenRouter, GLM, …).

> Note: this is a standalone Bash implementation, unrelated to the TypeScript project on `main`.

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
claude-switch create [name]       # create a profile (Anthropic / Antigravity / third-party)
claude-switch config [name]       # (re)configure a profile's API
claude-switch proxy [name] [act]  # manage an Antigravity proxy: start | stop | status
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

### Antigravity (agy) profiles — Claude Opus 4.6 via Google Antigravity

Choose **Antigravity (agy)** during `create` to use Claude through your Google
Antigravity subscription instead of an Anthropic API key. Claude Code talks to a
**local proxy** that translates Anthropic Messages ↔ Google Cloud Code, reusing the
OAuth login from the Antigravity app / `agy` CLI.

`claude-switch` doesn't ship the proxy — it drives one. The wizard presets the model
names (`claude-opus-4-6-thinking`, `claude-sonnet-4-6`), the base URL
(`http://localhost:8080`), and a dummy token, and optionally records a command to
**auto-start** the proxy. On launch, `claude-switch`:

1. health-checks the proxy's port,
2. auto-starts it (if a start command was configured) and waits until it's reachable,
3. then launches Claude Code pointed at it.

```bash
claude-switch create agy          # pick "Antigravity (agy)", accept the defaults
claude-switch agy                 # launches; starts the proxy if it's down
claude-switch proxy agy status    # up / down
claude-switch proxy agy stop      # stop a proxy claude-switch started
```

You need a working proxy installed. Popular options (pick one, see its README):

- [`antigravity-claude-proxy`](https://github.com/badrisnarayanan/antigravity-claude-proxy)
  — `npx -y antigravity-claude-proxy@latest start` (the default start command), port `8080`.
- [`claude-code-via-antigravity`](https://github.com/SovranAMR/claude-code-via-antigravity)
  — port `51200` (set the base URL to `http://localhost:51200` during `create`).

> ⚠️ **Terms of Service / ban risk.** Google has officially prohibited reverse-proxy
> use of these models and has **banned/shadow-banned** accounts that do it. This routes
> *your own* Google Antigravity account through an unofficial proxy — use it at your own
> risk. `claude-switch` only configures and launches a proxy you choose to run; it does
> not bypass any authentication.

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
