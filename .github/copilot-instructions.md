# General Instructions

Quick-start operating manual applicable everywhere in the repository and  workspace.

- Two main sections in this document representing two modes of work (MOW) with any AI agent:
  - Interactive chat.
  - Automatic (Agentic) mode of operation.

## General definitions

Approved plan:
- A plan that has been reviewed and accepted by the relevant stakeholders.

Context:
- Current state of data, code, memory, information gathered so far of the current task or decision.

Context is lost when:
- The agent cannot reliably connect the current task to:
  - The approved plan
  - Relevant code
  - Prior steps or decisions

Ambiguity is any situation where:
- Multiple valid interpretations exist
- A choice between alternatives is required
- The impact of a decision cannot be determined with high confidence

Ambiguity is classified as:
- Low: affects only formatting or non-functional aspects
- Medium: affects implementation details but not external behavior
- High: affects behavior, data, APIs, or system state

Low ambiguity:
- May be resolved locally without user input
- Must not affect behavior, data, APIs, or system state
- Must not expand scope beyond the current step

Cost is considered high when:
- Searching beyond the immediate codebase is required.
- Reverse-engineering behavior across multiple components is required.
- Assumptions that will generate a large change.
- Trial-and-error or speculative implementation is required.
- Gathering information is difficult or a large task.

Sufficient information exists when following holds:
- Requirements logic is explicitly stated and requires no interpretation
- Action to be taken is clearly defined
- The exact target (code, component, data, or artifact) is identified
- The scope of the change is clearly bounded
- The expected outcome is explicitly defined
- The task can be executed without introducing assumptions or making decisions.
- The expected outcome can be verified.

Context is unclear when:
- The agent lacks sufficient information to execute safely
AND
- Resolving the missing information is high cost.

---

## General expectations

The agent should:
- never deviate from the approved plan and context.
- provide responses with the sources and reasoning it is based on.
- ask user for guidance and clarification if/when required.
- stop if context is unclear or ambiguous.
- avoid making assumptions or inferring missing information, and if it does it must explicitly state the assumptions and reasoning behind them.
- accurately complete requirements given by the user, without dropping, adding or changing without explicit approval.

## Key principles

The agent should follow these principles:
- KISS: Keep It Simple Stupid
  - Use BKMs and best practices.
  - Avoid over-engineering, over-complicating, and over-designing.
- DRY: Don't Repeat
  - Avoid code duplication and redundancy:
    - abstract common functionality into reusable components or functions.
    - reference commonalities whenever needed.
- Single source of truth
  - definitions are unique. Further use of definitions is done by either:
    - referencing (for documenting)
    - reusing or importing (in code).

## Tool use

The agent must:
- Check what is available before stating that something cannot be done or seen:
  - search the available tools with more than one wording;
  - check the installed command-line tools (for example `gh`) when no tool offers the action;
  - read the actual state (a release, a tag, an issue) instead of asking the user for it.
- Use the dedicated GitHub tools first, and the `gh` CLI only for actions they lack
  (for example deleting a comment).
  - Run a `gh` write action only when the user asked for it, on an item the agent created or the
    user named, after reading the target to confirm it.
  - Report that the CLI was used and why.
- Read another repository, including a private one, through the repositories attached to the session:
  - use `gh api repos/<owner>/<repo>/contents/<path>?ref=<ref>` for an attached repository, and
    `git clone`/`fetch` for a public one that is not attached;
  - if access is refused, attach the repository (for example with `add_repo`) instead of copying
    its files into the working repository;
  - take a contract, schema or specification from its owning repository at a stated ref, never
    from a copy.

# Interactive chat rules

## Planning and collaboration rules

- Plan first - the agent should:
  - Sanitize user instructions:
    - fix typos
    - clarify ambiguities vague or incomplete instructions.
    - Strive for clear, detailed and comprehensive plan.
  - Provide options, discuss them with the user, and help reduce them to a single option.
  - Develop a detailed enumerated plan with the user.
    - Always include validation and documentation.

## Step / Task implementation rules

The agent must always:
- Use best known practices and patterns.
- Validate steps as part of the development process:
  - Validate all code changes.
  - code is always alive - continuously validated, user must confirm before moving to the next task.
  - report commands successful completion if and only if: they ran, ended successfully and produced the expected outcome.
- Maintain cross-platform OS Independence:
  - tools, scripts, paths, and implementations must support at least Windows and Linux, macOS is optional.
  - never hardcode OS-specific paths (like `C:\`) or OS-specific temporary directories without abstract cross-platform path joining tools (`path.join()`).
- Document:
  - any change: rationale and considerations.
    - update references across the codebase and if required in GitHub issues.
      - Record status, evidence and links in the body of the tracking issue, and edit the body in
        place. Do not add comments to a tracking issue unless the user asks for one.
      - Tick only the checklist items the evidence supports; leave the rest to the owner.
  - noticeable impact on the user, system, or other components.
- Build reusable code with minimal overhead:
  - prefer existing known reusable components.
  - consider abstraction and expansion of existing components when reuse is impossible.
  - When neither reuse nor abstraction and expansion are possible, create new reusable components.
  - refactor code when necessary to maintain coding standards.
- Task completion criteria
  - Run linters if exist and configured for every source code change
  - Run format checks for every change in the repo.
  - Run tests for every code change.

## Rules for executing the plan.

- When context is lost, or unclear, or medium to high ambiguity is encountered
  - → STOP
    - Provide a clear explanation of the missing context and its impact.
      - Wait for user instructions.
- Low ambiguity → proceed to step completion.
  - Document the ambiguity and its resolution.

- Research before generating solution code to achieve clear understanding of the latest and greatest from the web:
  - best practices, patterns, and approaches for the specific problem at hand.
  - Latest package versions, features and capabilities.
- Avoid trial-and-error and speculative implementation.
- Do not write outside the repo.

## Specific rules for interactive chat mode
- Execute in lock-step mode with the user.
  - Before executing any step:
    - The agent must confirm that sufficient information exists
    - If not → treat as unclear and STOP
- Start step execution only after user approval.
- Allow user to modify the proposed plan at any time.
- Avoid trial-and-error and speculative implementation.
- Stop after 3 failed attempts and ask for user guidance.

## Success criteria for any task

- Format checks
  - Run the check if setup and configured for any repo change.
    - Fix violations using pre-defined command and report.
- Lint checks
  - Run linting if setup and configured for any change in source or data used in execution
    - Fix linting issues and report.
- Run validation
  - Run all validation tasks setup and configured for any change in source or data used in execution.
    - Fix all issues and report.

---

# Automatic (Agentic) mode of operation rules

## Planning and collaboration rules - interactive chat rules apply.

## Plan step implementation rules - interactive chat rules apply.

## Rules for executing the plan - interactive chat rules apply.

## Success criteria for any task - interactive chat rules apply.
- Low ambiguity → proceed
  - Document the ambiguity and its resolution.

---
