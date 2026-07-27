# Working with Gemini in the Antigravity IDE desktop app

This file documents the platform an AI agent is running on when it operates on this repository from the **Antigravity IDE desktop app** (using Gemini models). It is about the agent runtime used to edit this repository, not the Gemini Python SDK or Gemini-specific adapter examples this repository may contain.

## Instruction chain

Follow the repository instruction chain already defined here:

- `GEMINI.md` points to this file for Gemini/Antigravity platform notes.
- `AGENTS.md` points to `.github/copilot-instructions.md`.
- `.github/copilot-instructions.md` is the canonical repository-specific instruction file.

Keep durable repository conventions in that chain. Keep Gemini API and SDK implementation details in [gemini.md](gemini.md).

## Workspace and shell model

The Antigravity IDE desktop app works directly against the selected workspace folder. In this repository, that folder is the real project directory on the user's machine:

```text
c:\Users\shlom\source\repos\shlomoa\shlomoa
```

Practical implications:

- Files written in the workspace are permanent project files.
- Prefer small, reviewable edits over wholesale rewrites.
- Do not delete, rename, or revert user changes unless the user explicitly asks.
- Use PowerShell commands from the repository root for local inspection and validation.
- Since the workspace is the user's real filesystem, ensure write tools only target files inside the workspace root.

## Gemini credentials

Gemini API access for local Python examples should be configured through environment variables, not committed files:

```powershell
$env:GEMINI_API_KEY = "your-api-key-here"
```

For Vertex AI configurations:

```powershell
$env:GOOGLE_GENAI_USE_VERTEXAI = "true"
$env:GOOGLE_CLOUD_PROJECT = "your-project-id"
$env:GOOGLE_CLOUD_LOCATION = "your-region-e.g.-us-central1"
```

## Python environment

Install Gemini provider dependencies from the repository root with:

```powershell
python -m pip install -r requirements/requirements_gemini.txt
```

Use `requirements/requirements_gemini.txt` for Gemini-specific runtime dependencies. Shared test dependencies remain in the root `requirements.txt`.

## Platform boundaries and customizations

Use the Antigravity IDE desktop app for repository editing, review, validation, and interactive coordination. Use the Gemini Python SDK inside repo code only when implementing Gemini-backed examples, adapters, or tests.

Provider-specific code should keep Gemini imports (`google-genai`), authentication, model selection, tool wrapping, and session configuration behind a Gemini adapter boundary.

Customizations (Rules and Skills) for Antigravity IDE:
- Reusable rules can be appended to the `AGENTS.md` file in the workspace customizations root `.agents`.
- Custom skills can be placed under the `.agents/skills/<skill_name>/` directory, containing a `SKILL.md` instruction file.
