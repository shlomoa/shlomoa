# GitHub Copilot provider

This document captures the GitHub Copilot Python SDK surface that should live behind a provider adapter when the repository is refactored for multiple AI engines.

Sources:

- Copilot Python SDK documentation: https://github.com/github/copilot-sdk/tree/main/python
- Copilot Python SDK raw README: https://raw.githubusercontent.com/github/copilot-sdk/main/python/README.md
- Copilot Python SDK chat sample: https://github.com/github/copilot-sdk/blob/main/python/samples/chat.py
- Copilot Python SDK package metadata: https://github.com/github/copilot-sdk/blob/main/python/pyproject.toml
- GitHub Copilot Memory concepts: https://docs.github.com/en/copilot/concepts/agents/copilot-memory

## SDK and authentication

- Python package: `github-copilot-sdk`.
- Python requirement from the SDK documentation/package metadata: Python 3.11 or later.
- Main imports:
  - `from copilot import CopilotClient`
  - `from copilot import RuntimeConnection`
  - `from copilot import define_tool`
  - `from copilot.session import PermissionHandler`
  - `from copilot.session_events import ...`
- Authentication can use the logged-in GitHub user or an explicit `github_token` passed to `CopilotClient`.

Authentication, runtime process configuration, and token handling are provider-specific and should not leak into shared domain code.

## Runtime

The Python SDK controls the GitHub Copilot CLI runtime through JSON-RPC.

Installation:

- `pip install github-copilot-sdk`
- Optional telemetry extra: `github-copilot-sdk[telemetry]`

Runtime management:

- Published wheels include a pinned runtime version.
- `python -m copilot download-runtime` downloads and caches the runtime binary.
- If the runtime is not downloaded explicitly, the SDK attempts to download it automatically on first use.

Runtime environment variables:

- `COPILOT_CLI_PATH`: use a specific runtime binary.
- `COPILOT_CLI_EXTRACT_DIR`: override the runtime cache/extract directory.
- `COPILOT_SKIP_CLI_DOWNLOAD=1`: disable automatic runtime download.
- `COPILOT_CLI_DOWNLOAD_BASE_URL`: override the GitHub Releases download URL.

Adapter boundary:

- Shared code should request a provider-neutral engine/session.
- The Copilot adapter should own runtime discovery, download policy, `RuntimeConnection`, process environment, and `CopilotClient` lifecycle.

## Core runtime APIs

Important primitives:

- `CopilotClient`
- `RuntimeConnection`
- `CopilotClient.create_session(...)`
- `CopilotClient.resume_session(...)`
- `session.send(...)`
- `session.send_and_wait(...)`
- `session.on(...)`
- `session.get_events()`
- `session.disconnect()`

Runtime connection variants:

- `RuntimeConnection.for_stdio(...)`: spawn a local runtime over stdio.
- `RuntimeConnection.for_tcp(...)`: spawn a local runtime in TCP mode.
- `RuntimeConnection.for_uri(...)`: connect to an existing CLI server.

Adapter boundary:

- Shared code should expose provider-neutral one-shot, streaming, and session APIs.
- The Copilot adapter should map those APIs to `CopilotClient`, session creation/resumption, event subscriptions, and JSON-RPC transport choices.

## Session configuration

Important `create_session(...)` options include:

- `model`
- `reasoning_effort`
- `session_id`
- `tools`
- `system_message`
- `streaming`
- `provider`
- `infinite_sessions`
- `memory`
- `on_permission_request`
- `on_user_input_request`
- `on_elicitation_request`
- `hooks`
- `commands`

Adapter boundary:

- Shared run options should capture model intent, system prompt behavior, tools, safety policy, streaming preference, and persistence policy.
- The Copilot adapter should convert those options to Copilot session kwargs and validate combinations such as custom provider requirements.

## Models and custom providers

The SDK supports Copilot-managed models and custom OpenAI-compatible providers.

Custom provider configuration supports:

- `type`: `openai`, `azure`, or `anthropic`.
- `base_url`.
- `api_key`.
- `bearer_token`.
- `wire_api`: `completions` or `responses`.
- `azure.api_version`.

Important behavior:

- `model` is required when using a custom provider.
- Azure OpenAI endpoints must use `type: "azure"`.
- For Azure, `base_url` should be the host, not an `/openai/v1` path.

Adapter boundary:

- Shared model selection should describe requested capabilities.
- The Copilot adapter should choose Copilot-managed model routing or custom provider routing and validate provider-specific requirements.

## Events and messages

The SDK is event-driven.

Common event data classes include:

- `AssistantMessageData`
- `AssistantMessageDeltaData`
- `AssistantReasoningData`
- `AssistantReasoningDeltaData`
- `SessionIdleData`
- `ToolExecutionStartData`

Streaming behavior:

- `streaming=True` emits text delta events such as `assistant.message_delta`.
- Final `assistant.message` and `assistant.reasoning` events are emitted regardless of streaming mode.
- `SessionIdleData` indicates that the session has finished processing the current turn.

Adapter boundary:

- Shared code should normalize Copilot events into provider-neutral events such as text delta, message completed, reasoning delta, tool started, tool completed, idle, usage, and error.
- The Copilot adapter should map `session_events` payload classes into that shared event model.

## Tools

The SDK supports custom tools and built-in tool overrides.

High-level tool API:

- `@define_tool(...)`
- Pydantic parameter models for automatic JSON schema generation.
- Declaration-only tools with generated Pydantic parameters.

Low-level tool API:

- `copilot.tools.Tool`
- `ToolInvocation`
- `ToolResult`
- Manual JSON schema definitions.

Tool options include:

- `overrides_built_in_tool=True`: intentionally replace a built-in tool such as `edit_file` or `read_file`.
- `skip_permission=True`: allow a specific tool to run without permission prompts.
- `defer="auto" | "never"`: control lazy loading/tool search behavior.
- `handler=None`: expose a declaration-only tool and resolve requests through pending external tool RPCs.

Adapter boundary:

- Shared code should define provider-neutral tool names, descriptions, schemas, handlers, side-effect metadata, and loading preferences.
- The Copilot adapter should choose between `define_tool`, low-level `Tool`, built-in override, permission skipping, and declaration-only behavior.

## Permissions and safety policy

Copilot permissions are handled through `on_permission_request`.

Built-in helper:

- `PermissionHandler.approve_all`.

Custom permission handlers can inspect `PermissionRequest` variants such as shell requests and return decisions including:

- `PermissionDecisionApproveOnce()`
- `PermissionDecisionReject(feedback="...")`
- `PermissionDecisionUserNotAvailable()`
- `PermissionNoResult()`
- Longer-lived approval variants such as approve-for-session/location/permanent.

Permission requests can also be emitted as events and resolved manually when no handler is provided.

Adapter boundary:

- Shared policy should describe allow, deny, ask, unavailable, and longer-lived approval intent.
- The Copilot adapter should map shared policy decisions to Copilot permission decision classes and permission events.

## Session hooks

Copilot sessions support lifecycle hooks through the `hooks` session option.

Available hooks include:

- `on_pre_tool_use`: inspect or modify tool calls before execution; can allow, deny, ask, change args, or add context.
- `on_post_tool_use`: process successful tool results.
- `on_post_tool_use_failure`: observe failed tool executions and add retry/context guidance.
- `on_user_prompt_submitted`: inspect or modify user prompts.
- `on_session_start`: run logic when a session starts or resumes.
- `on_session_end`: cleanup/logging.
- `on_error_occurred`: handle errors with retry, skip, or abort behavior.

Adapter boundary:

- Shared hook/policy logic should be expressed without Copilot payload shapes.
- The Copilot adapter should translate shared hook results to Copilot hook return fields such as permission decisions, modified arguments/prompts, additional context, and error handling.

## User input, commands, and UI elicitation

User input support:

- `on_user_input_request` handles agent questions through the `ask_user` tool.
- Requests can include a question, choices, and freeform settings.

Commands:

- `CommandDefinition` registers slash commands for the Copilot CLI TUI.
- Command handlers receive a `CommandContext` with session ID, full command text, command name, and raw args.
- Commands can also be supplied when resuming a session.

UI elicitation:

- `session.ui.confirm(...)`
- `session.ui.select(...)`
- `session.ui.input(...)`
- `session.ui.elicitation(...)`
- `on_elicitation_request` handles server or MCP elicitation requests.

Adapter boundary:

- Shared user-interaction requests should use provider-neutral ask/confirm/select/input schemas.
- The Copilot adapter should map those schemas to `on_user_input_request`, `session.ui`, command handlers, and elicitation request handlers.

## Image and multimodal input

The SDK supports image attachments through `session.send(..., attachments=[...])`.

Attachment forms:

- File attachment: `{"type": "file", "path": "..."}`.
- Blob attachment: `{"type": "blob", "data": "...", "mimeType": "image/png"}`.

The runtime can also use its image/view capabilities against files in the workspace.

Adapter boundary:

- Shared input should model attachments independently of provider payloads.
- The Copilot adapter should convert provider-neutral file/blob attachments to Copilot attachment dictionaries.

## Infinite sessions, state, and memory

Infinite sessions are enabled by default and manage context-window pressure with background compaction and persisted workspace state.

Session state details:

- Session workspace paths use Copilot-managed storage such as `~/.copilot/session-state/{session_id}/`.
- `infinite_sessions` can tune background compaction and buffer-exhaustion thresholds or disable the feature.
- Sessions emit compaction events when background compaction starts/completes.

Memory:

- Sessions can opt into persistent memory with `memory={"enabled": True}`.
- Copilot Memory stores repository-level facts and user-level preferences.
- Repository-level facts are scoped to the same repository and validated against current code before use.
- User-level preferences are scoped to the initiating user.
- Stored facts/preferences that go unused are automatically deleted after 28 days, with the timer reset when validated and used.

Adapter boundary:

- Shared code should expose provider-neutral session persistence, compaction, and memory preferences.
- The Copilot adapter should own Copilot workspace paths, infinite session config, `memory` settings, and Copilot Memory behavior.

## System message customization

Copilot sessions accept a `system_message` configuration.

Modes include:

- `append`: append custom content.
- `customize`: override selected prompt sections while preserving the rest.
- `replace`: replace the full system prompt, including SDK guardrails and security restrictions.

`customize` can target sections such as:

- `identity`
- `tone`
- `tool_efficiency`
- `environment_context`
- `code_change_rules`
- `guidelines`
- `safety`
- `tool_instructions`
- `custom_instructions`
- `last_instructions`

Section actions include `replace`, `remove`, `append`, `prepend`, and transform callbacks.

Adapter boundary:

- Shared code should model system instructions as append/customize/replace intent.
- The Copilot adapter should decide whether to use safe append/customize behavior or allow full replacement only for trusted workflows.

## Telemetry

The SDK supports OpenTelemetry through `CopilotClient(telemetry={...})`.

Telemetry options include:

- `otlp_endpoint`
- `otlp_protocol`
- `file_path`
- `exporter_type`
- `source_name`
- `capture_content`

Trace context is propagated between the SDK and CLI on session creation, resume, send, and tool-handler calls.

Adapter boundary:

- Shared observability should describe tracing/export intent.
- The Copilot adapter should map that intent to Copilot telemetry configuration and avoid capturing content unless explicitly requested.

## Current repo fit

GitHub Copilot SDK support is now implemented in this repository at the example/test/tool-runner
level. The current implementation intentionally keeps shared domain assets provider-neutral and
adapts them at the provider edge:

- [`examples/copilot/basic_call.py`](../../examples/copilot/basic_call.py) creates a minimal session and sends one prompt.
- [`examples/copilot/basic_custom_tool_use.py`](../../examples/copilot/basic_custom_tool_use.py) registers a custom calculator tool through the low-level `Tool` API.
- [`examples/copilot/basic_hook_use.py`](../../examples/copilot/basic_hook_use.py) demonstrates `on_pre_tool_use` and `on_post_tool_use` hooks.
- [`examples/copilot/basic_skill_creation.py`](../../examples/copilot/basic_skill_creation.py) mirrors Claude skill creation with host-managed `.copilot/capabilities/` files loaded through `system_message` append mode.
- [`examples/copilot/basic_skill_use.py`](../../examples/copilot/basic_skill_use.py) loads those local capability bundles into a session.
- [`examples/copilot/basic_plugin_use.py`](../../examples/copilot/basic_plugin_use.py) documents and demonstrates the plugin-equivalent path: host-managed command bundles plus SDK `CommandDefinition` registration.
- [`tools/copilot/create_angular_material_component.py`](../../tools/copilot/create_angular_material_component.py) loads the Copilot-owned Angular Material skill from [`skills/copilot/angular-material-component/`](../../skills/copilot/angular-material-component/) and exposes local read/write/list/edit/search/command tools to Copilot.
- [`tests/fixtures/angular-material-component/`](../../tests/fixtures/angular-material-component/) is the reusable Angular validation fixture used by Copilot live-generation tests; it is not runtime skill content and is not packaged into `.agents/skills/`.
- [`tests/copilot/`](../../tests/copilot/) contains offline unit coverage plus opt-in live coverage gated by `COPILOT_LIVE_TESTS=1`.

The Angular Material skill content is intentionally Copilot-owned. It may duplicate provider-domain
guidance from other implementations where necessary; that duplication is the accepted tradeoff for
keeping Copilot behavior correct and self-sufficient.

Implementation choices:

- Keep Angular Material prompt and parameter logic in shared/domain code.
- Own `CopilotClient`, sessions, model selection, and event/session behavior in Copilot-specific modules.
- Translate shared tools into low-level `Tool` values for the Angular generator.
- Translate shared safety policy into session hooks and permission handlers.
- Translate shared capability bundles into system-message customization, commands, tools, memory, or future provider-specific packaging.

Live validation requires `github-copilot-sdk`, a downloaded/configured runtime, and GitHub/Copilot
authentication. Unit tests avoid live sessions by default; set `COPILOT_LIVE_TESTS=1` to opt in.

## Shared vs Copilot-specific split

Keep shared:

- Domain prompt builders and input parameter models.
- Provider-neutral tool definitions, side-effect metadata, and loading preferences.
- Provider-neutral session/run options.
- Safety, permission, and approval policy intent.
- Normalized stream events, tool events, usage, state, and errors.
- User interaction schemas for questions, choices, confirmations, and forms.
- Attachment models for files and blobs.

Keep Copilot-specific:

- `github-copilot-sdk` imports.
- `CopilotClient` and `RuntimeConnection` lifecycle.
- Runtime binary download/cache configuration.
- GitHub authentication and `github_token` handling.
- Copilot session creation/resumption options.
- Copilot `session_events` classes and event names.
- `define_tool`, low-level `Tool`, and built-in tool override behavior.
- Permission decision classes and pending permission RPC behavior.
- Session hook names and payload/return shapes.
- Command definitions, UI elicitation, and user-input request handlers.
- Infinite sessions, Copilot workspace paths, and Copilot Memory settings.
- `COPILOT_*` environment variables and telemetry config.