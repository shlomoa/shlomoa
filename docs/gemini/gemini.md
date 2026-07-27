# Gemini provider

This document captures the Gemini-specific surface that should live behind a provider adapter when the repository is refactored for multiple AI engines.

Source:

- Gemini Python SDK documentation: https://googleapis.github.io/python-genai/

## SDK and authentication

- Python package: `google-genai`.
- Main imports:
  - `from google import genai`
  - `from google.genai import types`
- Gemini Developer API client:
  - `genai.Client(api_key="...")`
- Vertex AI client:
  - `genai.Client(vertexai=True, project="...", location="...")`

Environment variables:

- Gemini Developer API:
  - `GEMINI_API_KEY`
  - `GOOGLE_API_KEY`
- Vertex AI:
  - `GOOGLE_GENAI_USE_VERTEXAI=true`
  - `GOOGLE_CLOUD_PROJECT`
  - `GOOGLE_CLOUD_LOCATION`

These settings are provider-specific and should be owned by a Gemini adapter or runtime configuration layer, not domain code.

### Obtaining a Gemini API Key (and Project Requirements)

To obtain a Gemini Developer API key, you must use **Google AI Studio** (https://aistudio.google.com/app/apikey). 

If the system prompts you to select a Google Cloud Project and your desired project is "not listed and not an option" (or if you don't have one):
1. In the "Create API key" dialog, look for the option to **"Create API key in new project"**. Selecting this will automatically provision a free-tier Google Cloud project for you in the background and generate your key.
2. Alternatively, if you need to use a specific existing Google Cloud project but it isn't showing up, ensure that you have the correct permissions (Owner/Editor) in the [Google Cloud Console](https://console.cloud.google.com/), and that the **Generative Language API** is enabled for that project.

## Core generation APIs

Gemini's Python SDK exposes model calls through `client.models`.

Important APIs:

- `client.models.generate_content(...)`
- `client.models.generate_content_stream(...)`
- `client.aio.models.generate_content(...)`
- `client.aio.models.generate_content_stream(...)`

Chat APIs:

- `client.chats.create(...)`
- `chat.send_message(...)`
- `chat.send_message_stream(...)`

Common response fields:

- `response.text`
- `response.parts`
- `response.function_calls`
- `response.usage_metadata`
- `response.parsed`

Adapter boundary:

- Shared code should depend on a provider-neutral `generate`, `stream`, and optional `chat/session` contract.
- The Gemini adapter should translate that contract into `models.generate_content`, `generate_content_stream`, or `chats` calls.

## Model configuration

Generation settings are passed through `types.GenerateContentConfig`.

Important provider-specific fields include:

- `system_instruction`
- `max_output_tokens`
- `temperature`
- `top_p`
- `top_k`
- `safety_settings`
- `tools`
- `tool_config`
- `response_schema`
- `response_json_schema`
- `response_mime_type`
- `thinking_config`

Adapter boundary:

- Shared code should define generic run options such as system instructions, temperature, token limits, safety policy, structured-output schema, and tool definitions.
- The Gemini adapter should convert those options into `GenerateContentConfig`.

## Inputs and multimodal content

Gemini supports text and multimodal content.

Relevant SDK primitives include:

- `types.Part.from_text(...)`
- `types.Part.from_uri(...)`
- `types.Part.from_bytes(...)`

Gemini also supports file, image, video, embedding, cache, tuning, and batch APIs. Those are provider features and should only enter the shared layer when a domain explicitly needs them.

Adapter boundary:

- Shared prompts should be plain provider-neutral text or structured content objects.
- The Gemini adapter should convert structured content into Gemini `Content` / `Part` values.

## Tools and function calling

Gemini supports function calling and tool use, but it does not provide the same Claude Code-style filesystem agent runtime out of the box.

Tool options from the SDK documentation include:

- Passing Python functions directly in `tools=[...]`.
- Manually declaring functions with `types.FunctionDeclaration`.
- Grouping declarations through `types.Tool(function_declarations=[...])`.
- Controlling tool behavior through `types.FunctionCallingConfig`.
- Disabling automatic function calling with `types.AutomaticFunctionCallingConfig(disable=True)`.
- Reading requested function calls from `response.function_calls`.
- Experimental MCP support by passing an MCP session as a tool.

Adapter boundary:

- Shared code should define tools by name, description, schema, side-effect metadata, and handler.
- The Gemini adapter should decide whether to expose those tools as direct Python callables, `FunctionDeclaration` objects, or MCP-backed tools.
- Shared orchestration should not assume Claude-style built-in tools such as `Read`, `Edit`, or `Bash`; Gemini requires explicit application-provided tools for those capabilities.

## Streaming and async behavior

Gemini supports both synchronous and asynchronous streaming.

Provider-specific APIs:

- `client.models.generate_content_stream(...)`
- `client.aio.models.generate_content_stream(...)`
- `chat.send_message_stream(...)`

Adapter boundary:

- Shared code should normalize streaming into provider-neutral events, for example text deltas, completed messages, tool-call requests, tool results, and usage metadata.
- The Gemini adapter should translate Gemini stream chunks and function-call parts into those events.

## Safety and policy

Gemini exposes model safety controls through `safety_settings` and tool behavior through tool/function-calling config.

There is no direct equivalent to Claude Agent SDK hooks such as `PreToolUse` or OpenAI Agents SDK tool guardrails in the same runtime shape.

Adapter boundary:

- Shared safety policy should be expressed independently of provider payloads.
- The Gemini adapter can map model-level policy to `safety_settings`.
- Application-side tool policies should wrap shared tool handlers before Gemini receives or executes tool calls.

## Skills, plugins, and capability bundles

Gemini does not have a direct equivalent to Claude `.claude/skills` or `.claude-plugin` filesystem packages in the fetched Python SDK documentation.

Adapter boundary:

- Shared domain capability bundles should remain provider-neutral.
- For Gemini, those bundles can be adapted as:
  - system instructions,
  - prompt context,
  - retrieved reference files,
  - explicit function tools,
  - MCP tools when the experimental MCP path is suitable.

## Sessions and state

Gemini chat sessions are represented by `client.chats.create(...)` and subsequent `chat.send_message(...)` calls.

Adapter boundary:

- Shared code should own a provider-neutral conversation abstraction.
- The Gemini adapter should choose between Gemini chat sessions and application-managed history depending on the workflow.

## Error handling

The fetched SDK documentation references `google.genai.errors.APIError` for API errors.

Adapter boundary:

- The Gemini adapter should catch provider-specific exceptions and normalize them into shared error categories such as authentication, quota/rate limit, invalid request, safety block, transport failure, and tool failure.

## Shared vs Gemini-specific split

Keep shared:

- Domain prompt builders and input parameter models.
- Provider-neutral tool definitions.
- Structured-output schemas.
- Capability bundle metadata.
- Application-side tool policy and validation.
- Normalized messages, stream events, usage, and errors.

Keep Gemini-specific:

- `google-genai` imports.
- `genai.Client` construction.
- Gemini Developer API vs Vertex AI configuration.
- `types.GenerateContentConfig` conversion.
- Gemini `Content` / `Part` construction.
- Gemini function declaration and function-calling config.
- Gemini streaming chunk handling.
- `GEMINI_API_KEY`, `GOOGLE_API_KEY`, and Vertex AI environment variables.

## Gemini-Specific Skills

- **angular-material-component**: A standalone Gemini skill at `skills/gemini/angular-material-component` that fully leverages Gemini's autonomous execution model within the Antigravity IDE. Tests are modernized with `ComponentHarness`, and the skill is explicitly registered via `.agents/skills.json` instead of relying on a packaged `.skill` zip archive.
