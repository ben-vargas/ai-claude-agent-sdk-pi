# claude-agent-sdk-pi

This extension registers a custom provider that routes LLM calls through the **Claude Agent SDK** while **pi executes tools** and renders tool results in the TUI.

## Highlights

- Claude Agent SDK is used as the LLM backend (Claude Code auth or API key).
- Tool execution is **blocked in Claude Code**; pi executes tools natively.
- Built-in tool calls are mapped to Claude Code tool names.
- Custom tools are exposed to Claude Code via in-process MCP.
- Skills can be appended to Claude Code’s default system prompt (optional).

## Setup

1) Install the extension globally:

```
pi install git:github.com/prateekmedia/claude-agent-sdk-pi
```

Installed path (for git installs, pi places it under):
```
~/.pi/agent/git/github.com/prateekmedia/claude-agent-sdk-pi/
```

2) **Authenticate** (choose one):

- **Claude Code login** (Pro/Max):
  ```bash
  npx @anthropic-ai/claude-code
  ```
  Ensure no API key env vars are set.

- **API key** (API plan):
  ```bash
  export ANTHROPIC_API_KEY=sk-ant-...
  ```

3) Reload pi:

```
/reload
```

## Provider ID

`claude-agent-sdk`

Use `/model` to select:
- `claude-agent-sdk/claude-opus-4-5`
- `claude-agent-sdk/claude-haiku-4-5`

## Tool Behavior

- Claude Code **proposes** tool calls.
- pi **executes** them.
- Tool execution in Claude Code is **denied**.

Built-in tool mapping (Claude Code → pi):

- Read → read
- Write → write
- Edit → edit
- Bash → bash
- Grep → grep
- Glob → find

Claude Code only sees the tools that are active in pi.

### Custom tools

Any extra tools registered in pi are exposed to Claude Code via an in-process MCP server:

- MCP server name: `custom-tools`
- Claude Code tool name format: `mcp__custom-tools__<toolName>`
- Example: `mcp__custom-tools__subagent`

The provider automatically maps these back to the pi tool name (e.g. `subagent`).

## System Prompt Append (enabled by default)

By default, the provider **appends extra instructions** (AGENTS.md + skills block when available) using Claude Code’s preset prompt. This is **enabled by default**; set `appendSystemPrompt: false` to disable.

```
systemPrompt: { type: "preset", preset: "claude_code", append: "..." }
```

### Disable system prompt append

We suggest you to use ~/.claude dir for skills and your agent context like CLAUD.md, because that's what agent-sdk supports by default. Although for parity with other providers it appends skills + agents to your context.

To disable appending entirely, set in **global** or **project** settings:

**Global**: `~/.pi/agent/settings.json`
**Project**: `./.pi/settings.json`

```json
{
  "claudeAgentSdkProvider": {
    "appendSystemPrompt": false
  }
}
```

### AGENTS.md resolution

Default resolution order:
1) `AGENTS.md` walking up from the current working directory
2) `~/.pi/agent/AGENTS.md`

## Skills Location Aliases

To avoid leaking internal paths in the skills block, skill locations are rewritten before being sent to Claude Code:

- `~/.pi/agent/skills/...` → `~/.claude/skills/...`
- `.pi/skills/...` → `.claude/skills/...`

When Claude Code calls `Read/Edit/Write/Grep/Glob` using those aliases, the provider maps them back to the real paths.
