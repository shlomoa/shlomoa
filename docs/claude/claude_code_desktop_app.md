# Working with Claude in the desktop app (Cowork mode)

This file documents the platform an AI agent is running on when it operates on this
repository from the **Claude desktop app's Cowork mode** — not the standalone
`claude` CLI (Claude Code) and not the Claude Agent SDK examples this repo itself
contains. Cowork is *built on* Claude Code and the Claude Agent SDK, but it is a
distinct product with its own file-access model, tool set, and constraints. Keep
this distinction in mind: this repo teaches the Agent SDK, while this file
describes the runtime the agent uses to edit the repo.

## Two file locations, two path spaces

Every Cowork session has two roots:

- **Workspace folder** — the folder the user selected on their computer (this
  repo, `.../shlomoa/shlomoa`). Anything written here is a real, permanent file the
  user can open. Existing files can be edited freely, but once a file is written
  here it cannot be deleted or renamed by the agent without explicit user
  approval.
- **Outputs / scratchpad folder** — a temporary working directory used for
  drafts, intermediate build steps, and scripts. The user cannot see this
  folder; it is cleared between sessions. Finished deliverables get copied from
  here into the workspace folder.

File tools (`Read`, `Write`, `Edit`) address these locations with native OS
paths (e.g. `C:\Users\...\shlomoa\...`). The shell tool runs in a separate sandboxed
Linux VM where the same folders are mounted under different paths (e.g.
`/sessions/<session>/mnt/shlomoa/`). The two path spaces are **not** interchangeable
— a path that works in `Read`/`Edit` will not work in a `bash` call and vice
versa. When running repo scripts (e.g. the Python examples in `examples/claude/` or
`tests/claude/test_claude_started.py`) via the shell, use the `/sessions/.../mnt/shlomoa/...` form.

## Shell sandbox

- Isolated Ubuntu Linux VM, boots on first use (retry if it reports "still
  starting").
- Each shell call is independent: no working directory or environment carries
  over between calls, so use absolute paths and self-contained commands.
- Python, Node, and common CLI tools are preinstalled. `pip install` requires
  `--break-system-packages`.
- Has outbound network access to an allowlisted set of domains — not
  unrestricted internet access.
- This is the right place to run/test code from this repo (e.g.
  `python examples/claude/basic_call.py`, the test suite under `tests/claude/`),
  since it's a real Linux environment rather than a documentation-only sandbox.

## Tool surface relevant to this repo

- **Read / Write / Edit** — direct file operations on the workspace or
  scratchpad. `Edit` requires a prior `Read` of the same file.
- **Glob / Grep** — fast file-pattern and content search, preferred over shell
  `find`/`grep` for locating code.
- **Bash** (`mcp__workspace__bash`) — the Linux sandbox described above.
- **WebSearch / web_fetch** — for anything about current Claude Code / Agent
  SDK / Claude API behavior that might have changed since training data (see
  `AGENTS.md` → `.github/copilot-instructions.md` for the canonical source of
  repo instructions, and Anthropic's docs at docs.claude.com for platform
  behavior).
- **Skills** — packaged, reusable instructions (e.g. this repo's own
  `angular-material-component` skill under `skills/claude/angular-material-component/`) that can be invoked
  when their description matches the task. Skills that produce a specific
  output format (docx, pptx, xlsx, pdf) should only be loaded *after* research/
  content-gathering is done.
- **Agent / subagents** — for delegating open-ended, multi-step research or
  implementation work to a fresh context. Not needed for small, well-scoped
  edits like this one.
- **Task list (TaskCreate/TaskUpdate)** — used to track multi-step work and is
  shown to the user as a progress widget; not part of the repo itself.
- **Scheduled tasks / artifacts** — for recurring or persistent dashboards;
  generally not relevant to a static SDK-tutorial repo like this one.

## Practical notes for this repo

- Follow the instruction chain already defined in this repo: `CLAUDE.md` →
  `AGENTS.md` → `.github/copilot-instructions.md` (the canonical source of
  truth). This file is referenced from `CLAUDE.md` as a Claude-specific,
  platform-level note — it does not duplicate or override the instruction
  chain above it.
- Because the workspace folder is the user's real filesystem, prefer small,
  reviewable edits (`Edit`) over wholesale rewrites, and avoid deleting or
  renaming files without asking.
- Long-running or exploratory code (running examples, installing dependencies,
  executing tests) belongs in the shell sandbox, not simulated by reasoning
  about what the code would do.
