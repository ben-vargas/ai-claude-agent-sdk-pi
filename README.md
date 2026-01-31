# claude-agent-sdk-pi

This extension registers a custom provider that routes LLM calls through the **Claude Agent SDK** while **pi executes tools** and renders tool results in the TUI.

## Highlights

- Claude Agent SDK is used as the LLM backend (Claude Code auth or API key).
- Tool execution is **blocked in Claude Code**; pi executes tools natively.
- Built-in tool calls are mapped to Claude Code tool names.
- Custom tools are exposed to Claude Code via in-process MCP.
- Skills can be appended to Claude Code’s default system prompt (optional).

## Setup

1) Install the extension globally (already done):

```
~/.pi/agent/extensions/claude-agent-sdk-provider/
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

## Skills Append (optional)

By default, the provider **does not replace** Claude Code’s system prompt. It **appends only the skills block** (if any) using Claude Code’s preset prompt:

```
systemPrompt: { type: "preset", preset: "claude_code", append: "...skills block..." }
```

### Disable skills append

To disable skills appending entirely, set in **global** or **project** settings:

**Global**: `~/.pi/agent/settings.json`
**Project**: `./.pi/settings.json`

```json
{
  "claudeAgentSdkProvider": {
    "disableSkillsAppend": true
  }
}
```

## Skills Location Aliases

To avoid leaking internal paths in the skills block, skill locations are rewritten before being sent to Claude Code:

- `~/.pi/agent/skills/...` → `~/.claude/skills/...`
- `.pi/skills/...` → `.claude/skills/...`

When Claude Code calls `Read/Edit/Write/Grep/Glob` using those aliases, the provider maps them back to the real paths.

## Notes

- If Claude Code still executes a tool, ensure you’re using the `claude-agent-sdk` provider and reload after changes.
- Skills must have valid YAML frontmatter in `SKILL.md` (quoted `description` if it contains `:`).

---

If you want additional settings (e.g., include project context or custom prompt sections), let me know.
