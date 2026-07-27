# Working with OpenAI in the ChatGPT Codex desktop app

This file documents the platform an AI agent is running on when it operates on this
repository from the **ChatGPT Codex desktop app**. It is about the agent runtime
used to edit this repository, not the OpenAI Python SDK or the OpenAI Agents SDK
examples this repository may contain.

## Instruction chain

Follow the repository instruction chain already defined here:

- `OPENAI.md` points to this file for OpenAI/Codex platform notes.
- `AGENTS.md` points to `.github/copilot-instructions.md`.
- `.github/copilot-instructions.md` is the canonical repository-specific
  instruction file.

Keep durable repository conventions in that chain. Keep OpenAI API and SDK
implementation details in [openai.md](openai.md).

## Workspace and shell model

The Codex desktop app works directly against the selected workspace folder. In
this repository, that folder is the real project directory on the user's machine:

```text
C:\Users\shlom\source\repos\shlomoa\shlomoa
```

Practical implications:

- Files written in the workspace are persistent project files.
- Prefer small, reviewable edits over wholesale rewrites.
- Do not delete, rename, or revert user changes unless the user explicitly asks.
- Use PowerShell commands from the repository root for local inspection and
  validation.
- Network access may be restricted; if a dependency install or documentation
  fetch fails because of sandboxing or connectivity, request scoped approval
  instead of assuming the result.

## OpenAI credentials

OpenAI API access for local Python examples should be configured through
environment variables, not committed files:

```powershell
$env:OPENAI_API_KEY = "sk-..."
```

The same environment variable is used by the OpenAI Python SDK and the OpenAI
Agents SDK. API credentials do not by themselves enable ChatGPT, Codex desktop
app, plugin, connector, or workspace-admin access; those are separate product
and account surfaces.

## Python environment

Install OpenAI provider dependencies from the repository root with:

```powershell
python -m pip install -r requirements/requirements_openai.txt
```

Use `requirements/requirements_openai.txt` for
OpenAI-specific runtime dependencies. Shared test dependencies remain in the
root `requirements.txt`.

## Platform boundaries

Use the Codex desktop app for repository editing, review, validation, and
interactive coordination. Use the OpenAI Python SDK or OpenAI Agents SDK inside
repo code only when implementing OpenAI-backed examples, adapters, or tests.

Provider-specific code should keep OpenAI imports, authentication, model
selection, tool wrapping, tracing, sessions, and sandbox/agent configuration
behind an OpenAI adapter boundary. Shared/domain code should stay independent of
the Codex desktop app runtime and of OpenAI-specific SDK types.
