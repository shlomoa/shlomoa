# Claude provider

This document captures the Claude-specific surface that should live behind a provider adapter when the repository is refactored for multiple AI engines.

Sources:

- Claude Agent SDK overview: https://code.claude.com/docs/en/agent-sdk/overview
- Custom tools: https://code.claude.com/docs/en/agent-sdk/custom-tools
- Hooks: https://code.claude.com/docs/en/agent-sdk/hooks
- Plugins: https://code.claude.com/docs/en/agent-sdk/plugins
- Skills: https://code.claude.com/docs/en/agent-sdk/skills

## SDK and authentication

- Python package: `claude-agent-sdk`.
- Python requirement from the official overview: Python 3.10 or later.
- Primary authentication: `ANTHROPIC_API_KEY`.
- Alternate provider modes are selected by Claude-specific environment variables, for example:
  - `CLAUDE_CODE_USE_BEDROCK=1`
  - `CLAUDE_CODE_USE_ANTHROPIC_AWS=1`
  - `CLAUDE_CODE_USE_VERTEX=1`
  - `CLAUDE_CODE_USE_FOUNDRY=1`

These authentication and provider-routing details are adapter-specific and should not leak into shared domain code.

## Core runtime APIs

The current repository uses the Claude Agent SDK directly in examples and the starter-agent test.

Important Python primitives:

- `query(...)`: async generator for one-shot runs.
- `ClaudeSDKClient`: client/session style API.
- `ClaudeAgentOptions`: provider-specific run configuration.
- Message/result classes commonly used in this repo:
  - `SystemMessage`
  - `AssistantMessage`
  - `ResultMessage`
  - `TextBlock`
  - `ToolUseBlock`

Claude-specific options currently visible in the repo include:

- `model`
- `cwd`
- `allowed_tools`
- `tools`
- `disallowed_tools`
- `permission_mode`
- `setting_sources`
- `skills`
- `plugins`
- `hooks`
- `mcp_servers`
- `resume`
- `max_turns`

## Built-in tools and permissions

The Claude Agent SDK exposes Claude Code-style built-in tools such as:

- `Read`
- `Write`
- `Edit`
- `Bash`
- `Monitor`
- `Glob`
- `Grep`
- `WebSearch`
- `WebFetch`
- `AskUserQuestion`
- `Agent`
- `Skill`

Tool access has two layers:

- **Availability**: which tools appear in Claude's context, controlled by options such as `tools` and bare `disallowed_tools` entries.
- **Permission**: whether a tool call is pre-approved, blocked, or routed through permission handling, controlled by `allowed_tools`, scoped `disallowed_tools`, hooks, and permission settings.

Claude MCP tool names are fully qualified as:

- `mcp__<server_name>__<tool_name>`

The shared model should use provider-neutral tool identifiers. The Claude adapter should translate those identifiers to Claude's fully qualified MCP names and permission options.

## Custom tools

Claude custom tools are implemented as in-process MCP servers.

Provider-specific primitives:

- `@tool(...)`
- `create_sdk_mcp_server(...)`
- `ToolAnnotations`

A Claude tool definition includes:

- Name.
- Description.
- Input schema.
- Async handler.
- Optional annotations such as `readOnlyHint`, `destructiveHint`, `idempotentHint`, and `openWorldHint`.

Tool handlers return MCP-style result dictionaries with `content`, optional `is_error`, and content blocks such as `text`, `image`, and `resource`.

Adapter boundary:

- Shared code should define provider-neutral tools with name, description, schema, side-effect metadata, and handler.
- The Claude adapter should wrap shared tools with `@tool(...)`, build an in-process MCP server, and expose `mcp__...` names through `ClaudeAgentOptions`.

## Hooks and policy enforcement

Claude hooks are a major provider-specific extension point.

Python-supported hook events include:

- `PreToolUse`
- `PostToolUse`
- `PostToolUseFailure`
- `UserPromptSubmit`
- `Stop`
- `SubagentStart`
- `SubagentStop`
- `PreCompact`
- `PermissionRequest`
- `Notification`

Provider-specific primitives and payloads:

- `HookMatcher`
- `options.hooks`
- Hook callback signature: `(input_data, tool_use_id, context)`.
- Matchers filter by event-specific fields, commonly tool name for tool hooks.
- Hook output fields:
  - `systemMessage`
  - `continue_`
  - `async_`
  - `asyncTimeout`
  - `hookSpecificOutput`
  - `permissionDecision`
  - `permissionDecisionReason`
  - `updatedInput`
  - `additionalContext`
  - `updatedToolOutput`

Adapter boundary:

- Shared policy should be expressed as provider-neutral checks such as `allow`, `deny`, `ask`, `defer`, `rewrite input`, and `append context`.
- The Claude adapter should map those policy decisions into Claude hook output shapes.
- Existing logic in `examples/claude/basic_hook_use.py` is a good shared-policy candidate; its Claude return format is adapter code.

## Skills

Claude Agent Skills are filesystem artifacts, not programmatic registrations.

Provider-specific conventions:

- Project skills live under `.claude/skills/<skill-name>/SKILL.md`.
- User skills live under `~/.claude/skills/<skill-name>/SKILL.md`.
- Skills are discovered through `setting_sources` / `settingSources`.
- `skills="all"`, a list of skill names, or `[]` controls which discovered skills are available.
- SDK tool access still comes from run options such as `allowed_tools`; `allowed-tools` frontmatter is not honored by the SDK path.

Adapter boundary:

- Shared domain packages should own reusable skill instructions, references, examples, and metadata.
- The Claude adapter should render that package as `.claude/skills/.../SKILL.md` and configure `setting_sources` / `skills`.
- The Angular Material component generator in `skills/claude/angular-material-component/` contains domain knowledge that the Claude adapter repackages for `.claude/skills/...` discovery.

## Plugins

Claude plugins package multiple Claude Code extensions together.

Provider-specific conventions:

- Plugins are loaded from local filesystem paths via `plugins=[{"type": "local", "path": "..."}]`.
- A plugin root can contain:
  - `.claude-plugin/plugin.json`
  - `skills/`
  - `agents/`
  - `hooks/`
  - `commands/` for legacy commands
  - `.mcp.json`
- Plugin skills are namespaced as `<plugin-name>:<skill-name>` and can be invoked as `/plugin-name:skill-name`.

Adapter boundary:

- Shared code should model a capability bundle independently of Claude plugin layout.
- The Claude adapter can compile that bundle into the plugin directory structure when Claude plugin behavior is desired.

## Sessions and state

Claude sessions can be resumed by capturing a session ID from the initialization system message and passing it back via `ClaudeAgentOptions(resume=...)`.

Provider-specific state includes:

- Claude session IDs.
- Claude message classes.
- Claude Code filesystem settings and session storage behavior.

Adapter boundary:

- Shared code should expose provider-neutral conversation or run IDs.
- The Claude adapter should translate those IDs to Claude `resume` behavior and normalize returned events/messages.

## Current repo coupling

Claude-specific coupling currently appears in provider-scoped docs, examples, tools, and tests:

- `README.md`: multi-provider top-level positioning that includes Claude status, documentation links, tools, and tests.
- `tutorial_claude.md`: Claude provider tutorial with SDK setup, core APIs, tools, hooks, skills, and component runner examples.
- `tests/claude/test_claude_started.py`: direct `claude_agent_sdk` client/tool usage (starter-agent demo + tests).
- `examples/claude/basic_call.py`: direct `query(...)` usage.
- `examples/claude/basic_custom_tool_use.py`: in-process MCP tools.
- `examples/claude/basic_hook_use.py`: Claude hook payloads and return shapes.
- `examples/claude/basic_plugin_use.py`: `.claude-plugin` and plugin skill layout.
- `examples/claude/basic_skill_use.py`: Claude skill discovery.
- `examples/claude/basic_skill_creation.py`: `.claude/skills` scaffolding.
- `skills/claude/angular-material-component/`: Angular Material skill source used by the Claude runner.
- `tools/claude/create_angular_material_component.py`: installs the shared Angular Material skill into `.claude/skills/...` and invokes Claude.
- Tests that skip integration unless `claude_agent_sdk` and `ANTHROPIC_API_KEY` exist.

## Shared vs Claude-specific split

Keep shared:

- Domain prompts and parameter models.
- Tool names, descriptions, schemas, handlers, and side-effect metadata.
- Capability bundle metadata.
- Safety policy intent.
- Message/event normalization contracts.
- Test cases for domain prompt generation and tool behavior.

Keep Claude-specific:

- SDK imports and message classes.
- `ClaudeAgentOptions` construction.
- `query(...)` and `ClaudeSDKClient` invocation.
- `.claude/skills` filesystem layout.
- `.claude-plugin` plugin layout.
- Claude hook event names and output schema.
- Claude built-in tool names and permission settings.
- `mcp__<server>__<tool>` name mapping.
- `ANTHROPIC_API_KEY` and Claude provider-routing environment variables.
