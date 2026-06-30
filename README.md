# claude-switch

Run multiple Claude Code accounts on one machine — with third-party API provider support.

A single-file Bash tool that manages isolated Claude Code profiles. Each profile
gets its own `~/.claude-<name>/` directory (separate credentials and session data),
while shared config (`skills`, `CLAUDE.md`, `plugins`, `projects`) is symlinked from
`~/.claude/`. Profiles can authenticate via Anthropic OAuth **or** point at any
third-party Anthropic-compatible API (DeepSeek, Volcano Engine / Ark, OpenRouter,
GLM, …).

**Pure bash, zero external runtimes** — no `gum`, no `python3`, no `jq`. Just
`claude`, `bash`, and standard coreutils.

## Requirements

- [`claude`](https://docs.anthropic.com/en/docs/claude-code) (Claude Code CLI) in `PATH`
- `bash` 4+ (for `read -r`, regex, and parameter substitution)
- Standard coreutils (`sed`, `grep`, `printf`) — present on every Linux/macOS

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
- `create` adds a `claude-<name>` alias to `~/.bashrc` that simply runs
  `claude-switch launch <name>` — so **no secrets or provider env are written
  into your shell rc**.

### Third-party API profiles

Choosing **Third-party API** during `create` (or running `config`) stores a
`.provider.json` in the profile directory and wires up the relevant environment
variables when launching:

`ANTHROPIC_BASE_URL`, `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_MODEL`,
`ANTHROPIC_DEFAULT_OPUS_MODEL`, `ANTHROPIC_DEFAULT_SONNET_MODEL`,
`ANTHROPIC_DEFAULT_HAIKU_MODEL`, `CLAUDE_CODE_SUBAGENT_MODEL`.

You can set one model for everything, or per-tier models (Haiku/Sonnet/Opus +
subagent) for task routing. No OAuth login is needed — it works immediately.

### Where the API key is stored (v2.4+)

The key is **never** written in cleartext into `.provider.json` or your shell rc.
During `create`/`config` you pick a secret backend; the key is handed to it, and
`.provider.json` keeps only a **retrieval command** (`auth_token_cmd`) that prints
the key back at launch time — the same pattern as git's `credential.helper`.

Supported backends (auto-detected):

| Backend | Stored with | `auth_token_cmd` |
| --- | --- | --- |
| [`pass`](https://www.passwordstore.org/) | `pass insert claude-switch/<name>` | `pass show claude-switch/<name>` |
| `secret-tool` (libsecret / GNOME Keyring) | `secret-tool store …` | `secret-tool lookup service claude-switch account <name>` |
| `keychain` (macOS) | `security add-generic-password …` | `security find-generic-password … -w` |
| `plaintext` (fallback) | `~/.claude-<name>/.token` (chmod 600) | `cat ~/.claude-<name>/.token` |

Because `auth_token_cmd` is just a shell command, you can hand-edit
`.provider.json` to use **any** source — e.g. `op read op://vault/item/key`
(1Password CLI), `vault kv get -field=key secret/claude`, or
`printf %s "$MY_ENV_VAR"`.

> Profiles created before v2.4 used an inline `"auth_token"` field. Those still
> work (read as a legacy fallback); run `claude-switch config <name>` to migrate
> one to a secret backend.

> ⚠️ `auth_token_cmd` is executed via `eval` at launch. `.provider.json` lives in
> your home dir and is written `chmod 600`; treat it like any file that can run
> code as you (same trust model as a shell rc or git hook).
