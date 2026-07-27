# OpenAI provider

This document captures the OpenAI-specific surface that should live behind a provider adapter when the repository is refactored for multiple AI engines.

Sources:

- OpenAI Cookbook: https://github.com/openai/openai-cookbook
- OpenAI Cookbook site: https://cookbook.openai.com/
- OpenAI Agents SDK documentation: https://openai.github.io/openai-agents-python/
- OpenAI Agents SDK GitHub repository: https://github.com/openai/openai-agents-python
- Quickstart: https://openai.github.io/openai-agents-python/quickstart/
- Agents: https://openai.github.io/openai-agents-python/agents/
- Models: https://openai.github.io/openai-agents-python/models/
- Tools: https://openai.github.io/openai-agents-python/tools/
- Running agents: https://openai.github.io/openai-agents-python/running_agents/
- Streaming: https://openai.github.io/openai-agents-python/streaming/
- Sessions: https://openai.github.io/openai-agents-python/sessions/
- Guardrails: https://openai.github.io/openai-agents-python/guardrails/
- MCP: https://openai.github.io/openai-agents-python/mcp/
- Sandbox agents: https://openai.github.io/openai-agents-python/sandbox_agents/

## SDK and authentication

- OpenAI Python SDK package: `openai`.
- OpenAI Agents SDK package: `openai-agents`.
- OpenAI Python SDK requirement from the GitHub README: Python 3.9 or newer.
- OpenAI Agents SDK requirement from the GitHub README: Python 3.10 or newer.
- Authentication: `OPENAI_API_KEY`.
- Latest package versions observed on PyPI: `openai==2.45.0` and `openai-agents==0.18.2`.
- Package versions last checked: 2026-07-12.

OpenAI authentication and provider selection should remain adapter/runtime configuration, not domain logic.

Install repository OpenAI dependencies with:

```bash
python -m pip install -r requirements/requirements_openai.txt
```

Use the lower-level `openai` package for direct Responses API, Chat Completions,
embeddings, files, realtime, and other OpenAI REST API workflows. Use
`openai-agents` when the workflow needs an agent loop, tool execution,
guardrails, handoffs, sessions, tracing, or sandbox agents.

This repository's OpenAI Angular Material component runner uses the lower-level
`openai` package and the Responses API, not `openai-agents`.

## Angular Material component runner

OpenAI implementation:

- Runner: `tools/openai/create_angular_material_component.py`.
- Tests: `tests/openai/test_angular_material_component_skill.py`.
- OpenAI skill source: `skills/openai/angular-material-component/`.
- Provider-neutral Angular validation fixture: `tests/fixtures/angular-material-component/`.
- Default model: `gpt-5.6`.
- SDK path: `from openai import OpenAI`, then `client.responses.create(...)`.
- Tooling path: custom Responses API function tools for `read_file`, `write_file`,
  `list_dir`, `edit_file`, `grep_search`, and `run_command`.

Authentication is environment-based:

```powershell
$env:OPENAI_API_KEY = "your_api_key"
python tools/openai/create_angular_material_component.py `
  "Create a compact status card." `
  --project-root tests/fixtures/angular-material-component
```

Do not commit API keys or `.env` files containing credentials. The OpenAI SDK
will read `OPENAI_API_KEY` by default when `OpenAI()` is constructed.

Live tests are skipped unless both the `openai` package and `OPENAI_API_KEY` are
available. They also skip when the OpenAI account cannot run the request because
of authentication, permission, rate-limit, or quota errors:

```bash
python -m pytest tests/openai/test_angular_material_component_skill.py -v
```

The runner loads the OpenAI `SKILL.md` plus component catalog, Material 3
theming, and accessibility references into the Responses API `instructions`
field. The natural-language request and optional CLI parameters become the user
input. When the model calls a function tool, the runner executes the local helper
and sends a `function_call_output` item back to the next `responses.create(...)`
call until the model returns final text.

## Test coverage and Claude parity

OpenAI test coverage mirrors the Claude test areas where this repository has an
OpenAI implementation:

- `tests/openai/test_openai_started.py` mirrors the Claude starter flow with a
  Responses API function-tool call and client-managed conversation context.
- `tests/openai/test_basic_skill_creation.py` mirrors skill creation and
  project skill discovery using `.agents/skills/<skill>/SKILL.md`, with live
  loading into `instructions`.
- `tests/openai/test_skill_creation.py` mirrors the CLI smoke test for the
  OpenAI-compatible skill scaffolder.
- `tests/openai/test_basic_hook_plugin_use.py` mirrors hook/plugin coverage
  with local function-tool hook callbacks and provider-neutral capability-bundle
  loading.
- `tests/openai/test_angular_material_component_skill.py` validates the standalone
  OpenAI Angular Material skill and live generation through the OpenAI Python SDK.

This repository now provides OpenAI-side equivalents for the three Claude test
areas that do not map directly to native OpenAI SDK options:

- project skill discovery loads `.agents/skills/*/SKILL.md` with
  `skills="all"`, `skills=[]`, or an explicit name subset;
- local function-tool hook callbacks run before and after OpenAI tool handlers;
- local capability bundles load from `.agents-plugin/plugin.json` plus bundled
  `skills/*/SKILL.md` files.

These are repository adapter behaviors, not native Claude-style OpenAI SDK
options. OpenAI runners still pass the resulting instructions and local tool
wrappers explicitly to the OpenAI SDK.

## Cookbook and examples

The OpenAI Cookbook is the primary example repository for practical OpenAI API
recipes and should be checked before adding new OpenAI examples to this
repository.

Repo-local OpenAI examples live in `examples/openai/`:

- `basic_call.py`: minimal Responses API connectivity check.
- `basic_custom_tool_use.py`: local function-tool loop with
  `function_call_output`.
- `basic_hook_use.py`: repo-level pre/post callbacks around local OpenAI tool
  handlers.
- `basic_skill_creation.py`: `.agents/skills/<name>/SKILL.md` scaffolding and
  project skill discovery.
- `basic_skill_use.py`: explicit skill loading into Responses API
  `instructions`.
- `basic_plugin_use.py`: local capability-bundle scaffolding/loading through
  `.agents-plugin/plugin.json`.

See `examples/openai/README.md` for setup and commands.

Current cookbook status observed on 2026-07-12:

- Repository: `openai/openai-cookbook`.
- Main site: https://cookbook.openai.com/
- GitHub repository: https://github.com/openai/openai-cookbook
- Latest visible `main` commit: `23fb2d3`, dated 2026-07-10.
- Latest visible commit title: `Add GPT-5.6 disclaimer to Prompt Caching 201 (#2845)`.
- The commit history showed active OpenAI/Codex, Realtime, Agents SDK, evals,
  prompt-caching, and provider-integration cookbook updates across May-July
  2026.

Use the cookbook for task-oriented recipes, migration examples, and implementation
patterns. Treat official platform and SDK documentation as the source of truth
for API contracts, authentication, model availability, and package installation.
When cookbook guidance and API documentation differ, prefer the current official
API or SDK documentation and note the cookbook version/date that was consulted.

## Core runtime APIs

Important primitives:

- `Agent`
- `Runner`
- `RunConfig`
- `ModelSettings`
- `RunResult`
- `RunResultStreaming`
- `SQLiteSession` and other session implementations

Runner entry points:

- `Runner.run(...)`: async run.
- `Runner.run_sync(...)`: synchronous wrapper.
- `Runner.run_streamed(...)`: async streaming run.

Basic agent configuration fields include:

- `name`
- `instructions`
- `prompt`
- `handoff_description`
- `handoffs`
- `model`
- `model_settings`
- `tools`
- `mcp_servers`
- `mcp_config`
- `input_guardrails`
- `output_guardrails`
- `output_type`
- `hooks`
- `tool_use_behavior`

Adapter boundary:

- Shared code should expose provider-neutral `AgentEngine` run methods and normalized results.
- The OpenAI adapter should construct `Agent`, `Runner`, `RunConfig`, tools, sessions, and guardrails.

## Agent loop and orchestration

The OpenAI Agents SDK manages an agent loop:

1. Call the current model.
2. If there is final output, return it.
3. If there is a handoff, switch active agent and continue.
4. If there are tool calls, execute tools, append results, and continue.
5. Stop when `max_turns` is exceeded, unless disabled.

Multi-agent patterns include:

- Manager/orchestrator agents using agents as tools.
- Handoffs where a specialist agent takes over the conversation.
- Experimental hosted multi-agent behavior on supported OpenAI Responses models.

Adapter boundary:

- Shared orchestration should model capabilities without assuming OpenAI-specific handoffs or `Agent.as_tool(...)`.
- The OpenAI adapter can use handoffs or agents-as-tools when a workflow needs provider-native multi-agent support.

## Models and providers

The Agents SDK supports OpenAI models through:

- `OpenAIResponsesModel` for the Responses API path.
- `OpenAIChatCompletionsModel` for the Chat Completions API path.

It also includes provider integration points for non-OpenAI models:

- `set_default_openai_client(...)` for OpenAI-compatible endpoints.
- `ModelProvider` at run scope.
- `Agent.model` for per-agent model selection.
- `MultiProvider` for prefix-based routing.
- Optional beta adapters such as Any-LLM and LiteLLM.

OpenAI Responses-specific features include:

- Hosted tools.
- Tool search.
- OpenAI server-managed conversation state.
- Responses websocket transport.
- Some advanced `ModelSettings` fields.

Adapter boundary:

- Shared model selection should describe intent and required capabilities.
- The OpenAI adapter should validate whether the selected OpenAI model path supports the requested features.

## Tools

The OpenAI Agents SDK supports several tool categories.

Hosted OpenAI tools:

- `WebSearchTool`
- `FileSearchTool`
- `CodeInterpreterTool`
- `HostedMCPTool`
- `ImageGenerationTool`
- `ToolSearchTool`

Local/runtime execution tools:

- `ComputerTool`
- `ShellTool`
- `LocalShellTool` for legacy local shell integration
- `ApplyPatchTool`

Function tools:

- `@function_tool`
- `FunctionTool`
- automatic function signature and docstring schema extraction
- Pydantic/TypedDict/dataclass schema support
- optional timeouts
- custom error formatting

Agent tools:

- `Agent.as_tool(...)`
- structured input for tool-agents
- approval gates
- custom output extraction
- nested streaming
- conditional tool enabling

Experimental workspace delegation:

- `codex_tool(...)` for bounded workspace-scoped Codex tasks.

Hosted shell can also run in OpenAI-hosted containers and can mount skill references or inline skill bundles.

Adapter boundary:

- Shared code should define provider-neutral tool contracts.
- The OpenAI adapter should choose the best tool representation: `function_tool`, `FunctionTool`, hosted tool, MCP server, shell, sandbox capability, or agent-as-tool.
- Tool metadata such as read-only/destructive/idempotent/open-world should be shared, while OpenAI-specific execution options remain adapter-owned.

## Guardrails, hooks, and approvals

OpenAI has several policy and lifecycle mechanisms.

Agent-level guardrails:

- Input guardrails run on the first agent in the chain.
- Output guardrails run on the final output.
- Guardrails return `GuardrailFunctionOutput` and can trip exceptions.

Tool guardrails:

- Input tool guardrails run before function tool execution.
- Output tool guardrails run after function tool execution.
- They can allow, replace output, reject, or trip exceptions.

Lifecycle hooks:

- `RunHooks` observe an entire `Runner.run(...)` invocation.
- `AgentHooks` attach to a specific `Agent`.
- Hook timing includes agent start/end, LLM start/end, tool start/end, and handoffs.

Approvals and human-in-the-loop:

- Tool approval can pause a run.
- Interrupted runs can be converted to `RunState`, approved/rejected, and resumed.

Adapter boundary:

- Shared policy should describe checks and desired outcomes.
- The OpenAI adapter should map policy to guardrails, lifecycle hooks, tool approval, or wrapper functions depending on the tool type.

## Streaming and events

OpenAI streaming uses `Runner.run_streamed(...)` and `result.stream_events()`.

Stream event groups include:

- `raw_response_event`: raw OpenAI Responses API events such as text deltas.
- `run_item_stream_event`: semantic item events.
- `agent_updated_stream_event`: active agent changes.

Run item event names include:

- `message_output_created`
- `handoff_requested`
- `handoff_occured`
- `tool_called`
- `tool_search_called`
- `tool_search_output_created`
- `tool_output`
- `reasoning_item_created`
- `mcp_approval_requested`
- `mcp_approval_response`
- `mcp_list_tools`

Adapter boundary:

- Shared code should normalize all providers to common stream events such as text delta, message completed, tool requested, tool completed, approval requested, agent changed, usage, and error.
- The OpenAI adapter should translate OpenAI stream events into that shared event set.

## Sessions and state

OpenAI offers multiple state strategies.

Client-managed options:

- `result.to_input_list()` for manual history.
- `session=...` for SDK-managed history.

Built-in session implementations include:

- `SQLiteSession`
- `AsyncSQLiteSession`
- `RedisSession`
- `SQLAlchemySession`
- `MongoDBSession`
- `DaprSession`
- `OpenAIConversationsSession`
- `OpenAIResponsesCompactionSession`
- `AdvancedSQLiteSession`
- `EncryptedSession`

OpenAI server-managed options:

- `conversation_id`
- `previous_response_id`

The SDK notes that sessions cannot be combined with `conversation_id`, `previous_response_id`, or `auto_previous_response_id` in the same run.

Adapter boundary:

- Shared code should define a conversation/session abstraction and persistence policy.
- The OpenAI adapter should choose one OpenAI state mechanism per run and prevent invalid combinations.

## MCP

The OpenAI Agents SDK supports Model Context Protocol integrations through several paths.

Hosted MCP:

- `HostedMCPTool` lets OpenAI Responses models call publicly reachable MCP servers on the model's behalf.

Local/managed MCP servers:

- `MCPServerStreamableHttp`
- `MCPServerSse`
- `MCPServerStdio`
- `MCPServerManager`

Common MCP features:

- Tool filtering.
- Prompt retrieval.
- Tool list caching.
- Approval policies.
- Per-call metadata through `tool_meta_resolver`.
- Tracing.

Adapter boundary:

- Shared code should model MCP as one possible tool transport.
- The OpenAI adapter should decide hosted vs local transport, approval policy, caching, and tool naming.

## Sandbox agents and skills

Sandbox agents are beta in the OpenAI Agents SDK.

Important primitives:

- `SandboxAgent`
- `Manifest`
- `SandboxRunConfig`
- `GitRepo`
- `LocalDir`
- `UnixLocalSandboxClient`
- Docker-backed sandbox clients via the `openai-agents[docker]` extra
- Sandbox capabilities such as filesystem, shell, memory, compaction, and skills

The tools guide also describes hosted container shell environments that can use skill references or inline skill bundles.

Adapter boundary:

- Shared capability bundles should not assume Claude `.claude/skills` layout.
- The OpenAI adapter can map shared capability bundles to sandbox skills, shell skill bundles, instructions, or function/MCP tools depending on the target runtime.

## Current repo fit

OpenAI is partially implemented in this repository through the Angular Material
component runner. To add more OpenAI-backed flows cleanly:

- Keep Angular Material component generation prompts in shared/domain code.
- Use the lower-level Responses API for direct model/tool loops, or implement an
  OpenAI provider adapter around `Agent` and `Runner` when the workflow needs
  Agents SDK orchestration.
- Translate shared tools into `@function_tool` or `FunctionTool`.
- Translate shared safety policy into guardrails, lifecycle hooks, approval settings, or tool wrappers.
- Translate shared capability packages into instructions, tools, sandbox capabilities, or shell skills.

## Shared vs OpenAI-specific split

Keep shared:

- Domain prompt builders and parameter models.
- Tool schemas, handlers, and side-effect metadata.
- Capability bundle metadata.
- Provider-neutral session/run options.
- Safety and approval policy intent.
- Normalized result, event, usage, and error models.

Keep OpenAI-specific:

- `openai-agents` imports.
- `Agent`, `Runner`, `RunConfig`, and `ModelSettings` construction.
- OpenAI Responses vs Chat Completions model selection.
- Hosted tool selection.
- `function_tool` and `FunctionTool` wrapping.
- Guardrail and lifecycle hook implementation.
- OpenAI session implementation choices.
- MCP transport configuration.
- Sandbox agent manifests and capabilities.
- `OPENAI_API_KEY` and OpenAI-specific tracing/provider environment.
