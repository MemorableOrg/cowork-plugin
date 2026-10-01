# Memorable for Claude Cowork

A Claude Code plugin that gives Cowork procedural memory: it records how a task was actually carried out and replays that procedure the next time a similar one arrives.

Cowork runs Claude Code underneath (`~/Library/Application Support/Claude/claude-code/`), keeps a per-space config dir, and loads skills through the same plugin format Claude Code uses. So one plugin carries all three surfaces at once.

```
.claude-plugin/plugin.json       the plugin
.claude-plugin/marketplace.json  so this directory can be added as a marketplace
hooks/hooks.json                 UserPromptSubmit: recall on every prompt, capture on the boundary
.mcp.json                        the read-only memorable MCP server
bin/memorable                    resolves the CLI without depending on PATH
skills/memorable/SKILL.md        when to recall, what the block means, consent
```

## Install

The CLI has to exist on the machine first. No npm needed:

```sh
curl -fsSL https://memorable.sh/install.sh | sh
memorable login     # one browser approval, links this machine
memorable enable    # explicit consent; nothing is stored before this
```

Then add this directory as a plugin marketplace and install from it:

```
/plugin marketplace add MemorableOrg/cowork-plugin
/plugin install memorable@memorable
```

## Context sources

[Use Screenpipe context with Memorable](docs/screenpipe.md) to combine recorded
desktop context with recalled procedures in a local Claude Code session.

## Why `bin/memorable` exists

Cowork is a GUI app, so its process PATH is the launchd default: no `/opt/homebrew/bin`, no nvm shim, no `~/.local/bin`. A hook written with the bare name `memorable` would resolve to nothing and fail silently on every turn. The wrapper tries `$MEMORABLE_BIN`, then `~/.memorable/bin/memorable`, then `~/.local/bin/memorable`, then PATH, then `npx`, and exits 0 with no output if none of them resolve. A missing CLI never breaks somebody's turn.

## What each surface does

**Hook, `UserPromptSubmit`.** Every prompt gets a recall check. A match is returned as `additionalContext`, wrapped in a note saying it is reference data and not instructions. The same event is the segmentation boundary, so the previous piece of work is closed and recorded at that moment; capture needs no second hook. Consent gates it: unset or denied means the hook does nothing.

**MCP server, read-only.** `memorable_recall`, `memorable_show`, `memorable_list`, `memorable_status`, `memorable_explain_recall`. Lets the model pull a procedure mid-task rather than only receiving one at the prompt. No tool in it writes.

**Skill.** Tells the model when a task is worth a recall, how to treat the block it gets back, how to record explicitly, and what to say when consent is off.

## Verified

Run from this directory with `CLAUDE_PLUGIN_ROOT=$PWD`:

- `./bin/memorable --version` resolves the CLI.
- The hook returns valid `hookSpecificOutput` JSON on a matching prompt, exits 0 on a non-matching one, and exits 0 on malformed stdin.
- The MCP server completes an `initialize` handshake at protocol `2025-06-18` and lists its five read-only tools.

## Uninstall

`/plugin uninstall memorable`, then `rm -rf ~/.memorable` to remove the stored procedures.

## License

This plugin is MIT (see `LICENSE`). It is a wrapper: the `memorable` CLI it
calls is a separate package under its own license, and installing this plugin
does not grant any rights to it.
