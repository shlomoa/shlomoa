# Agent default intructions file search path

## Claude

Quick correction first: I'm running as **Cowork** (built on the Claude Agent SDK/Claude Code engine), not the Claude Code CLI itself — but the underlying instruction mechanism is the same.

**File**: `CLAUDE.md`

**Default paths, by scope** (broadest → most specific; all get concatenated into context, later ones override in practice):

| Scope | Path | Purpose |
|---|---|---|
| Managed/org policy | Win: `C:\Program Files\ClaudeCode\CLAUDE.md` | IT-deployed, applies to all users |
| User (you, all projects) | `~/.claude/CLAUDE.md` (Win: `%USERPROFILE%\.claude\CLAUDE.md`) | Your personal preferences |
| Project (team-shared) | `./CLAUDE.md` or `./.claude/CLAUDE.md` at the project root | Project conventions, checked into source control |
| Local (you, this project only) | `./CLAUDE.local.md` | Your sandbox settings, gitignored |

Resolution: it walks up the directory tree from the working directory, loading every `CLAUDE.md`/`CLAUDE.local.md` it finds along the way (root-most first, closest-to-cwd last), plus nested `CLAUDE.md` files in subdirectories load lazily when Claude reads files there. `/init` generates a starter one from the codebase.

Separately, there's **auto memory** — notes Claude writes itself (not user-authored) — stored at `~/.claude/projects/<project>/memory/MEMORY.md` plus topic files, distinct from CLAUDE.md.

In this Cowork session specifically, the equivalent role is played by the workspace's `.claude/` folder (skills live in `.claude/skills/`) rather than an arbitrary project CLAUDE.md, since no folder is currently mounted from your computer.

Sources:
- [How Claude remembers your project](https://code.claude.com/docs/en/memory.md)
- [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory)

---

## Gemini

When working on a project in the Antigravity IDE, I automatically discover and read instructions (rules and behavioral constraints) from specific Customization Roots.

Here are the exact files and default paths I read for instructions:

### 1. Project-Scoped Instructions (Workspace Level)

- Default Path: .agents/AGENTS.md (relative to the root of the current workspace).

### 2. Global Instructions (Cross-Project Level)

- Default Path: `~/.gemini/config/AGENTS.md` (Win: `%USERPROFILE%\.gemini\config\AGENTS.md`)
- Note: Any instructions placed here will apply universally across all workspaces and projects you open in the IDE.

### 3. Model-Specific Instructions (Root Level)

- Path: GEMINI.md (in the root of the workspace).

---

## Copilot

### Short answer

I am GitHub Copilot. For a project/workspace, the default repo instruction file I read from is:

`<workspace-root>/.github/copilot-instructions.md`

For your current repo, that is:

copilot-instructions.md

### Other supported instruction locations

Copilot/VS Code can also use these customization files:

| Purpose | Default/project path |
|---|---|
| Project-wide agent instructions | copilot-instructions.md |
| Alternative/open-standard agent instructions | `AGENTS.md` at repo root or subfolders |
| File/glob-specific instructions | `.github/instructions/*.instructions.md` |
| Reusable prompts | `.github/prompts/*.prompt.md` |
| Custom agents | `.github/agents/*.agent.md` |
| Skills | `.github/skills/<skill-name>/SKILL.md` |

User-level customizations can live under your VS Code user prompts folder:

Win: `%USERPROFILE%\AppData\Roaming\Code - Insiders\User\prompts`

---

## OpenAI

Codex Desktop reads durable workspace instructions from files named `AGENTS.md`.

For your Windows setup:

- Global instructions for every project:
  `%USERPROFILE%\.codex\AGENTS.md`
  
  This is `%CODEX_HOME%\AGENTS.md`; `CODEX_HOME` defaults to `%USERPROFILE%\.codex`.

- Project instructions:
  `%USERPROFILE%\source\repos\shlomoa\ai\AGENTS.md`
  
  In general: `<Git repository root>\AGENTS.md`.

Lookup order:

1. Global: `~/.codex/AGENTS.override.md`, otherwise `~/.codex/AGENTS.md`
2. Repository root: `AGENTS.override.md`, otherwise `AGENTS.md`
3. Each directory from the repository root down to the current working directory
4. More deeply nested instructions take precedence over broader instructions

For example, while working in `ai\services\api`, Codex can combine:

```text
%USERPROFILE%\.codex\AGENTS.md
%USERPROFILE%\source\repos\shlomoa\ai\AGENTS.md
%USERPROFILE%\source\repos\shlomoa\ai\services\AGENTS.md
%USERPROFILE%\source\repos\shlomoa\ai\services\api\AGENTS.md
```

The model name—such as GPT-5.6 Sol—does not change this instruction-file convention; discovery is handled by Codex. Empty files are ignored, and instructions are normally loaded when the task/session starts. [Official AGENTS.md documentation](https://developers.openai.com/codex/guides/agents-md).

---
