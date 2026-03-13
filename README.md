# Claude Bridge Skill

Portable agent skill for non-Claude hosts that want to call the local Claude CLI or generate Claude-ready prompt packages.

## Installing Claude Bridge

You can install this skill into Codex, Gemini CLI, Cursor, GitHub Copilot, OpenCode, and other supported hosts by using `npx`:

```bash
npx skills add https://github.com/danielraffel/claude-bridge-skill --skill claude-bridge
```

If you get the error `npx: command not found`, you need to [install Node first](https://nodejs.org/en/download).

When using `npx`, the installer will guide you through which agents to install into and whether the skill should be installed for just the current project or for all your projects.

If you prefer, you can also clone this repository and install it however you want.

## Using Claude Bridge

The skill is called `claude-bridge`.

In hosts that support explicit skill invocation, use that host's normal skill shortcut with `claude-bridge`. For example, in Codex:

```text
$claude-bridge Give me a second opinion on this refactor
$claude-bridge Analyze these test failures and return structured risks and next actions
$claude-bridge package Convert this bug report into a Claude-ready prompt package
```

You can also trigger the skill using natural language:

```text
Use the claude-bridge skill to ask Claude for a second opinion on this migration plan.
Use claude-bridge to turn this ticket into a Claude-ready prompt package.
```

## Notes

- Requires local `claude` CLI access and authentication.
- Intended for Codex, Gemini CLI, Cursor, GitHub Copilot, OpenCode, and similar hosts.
- Not intended for Claude Code bridge calls.

## License

MIT
