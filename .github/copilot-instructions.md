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

# Interactive chat rules

## Planning and collaboration rules

- Always plan first.
  - Sanitize user instructions - fix typos, clarify ambiguities, elaborate on vague and incomplete instructions.
  - Strive for a clear, detailed, and comprehensive plan.  
  - Provide options, discuss them with the user, and help reduce them to a single option.
  - Develop a detailed enumerated plan with the user.
    - Always include validation and documentation.
- Ask user for guidance and clarification if/when required.
  - Stop if context is unclear or ambiguous.
  - Do not assume or infer missing information.
- Complete requirements given by the user accurately, do not add, do not miss.

## Plan step implementation rules

- KISS: Keep It Simple Stupid
  - Use BKMs and best practices.
  - Avoid over-engineering, over-complicating, and over-designing.
- DRY: Don't Repeat Yourself
  - Avoid code duplication and redundancy:
    - abstract common functionality into reusable components or functions.
    - reference commonalities whenever needed.
- Single source of truth
  - definitions are unique. Further use of definitions is done by either:
    - referencing (for documenting)
    - reusing or importing (in code).
- Validation is part of the development process.
  - Any change to executable code must include corresponding tests.
  - Always validate fully working code with the user, before moving to the next task.
  - Only report commands as passing if they ran, ended successfully and produced the expected outcome.
- Cross-platform OS Independence
  - All tools, scripts, paths, and implementations must support Windows, Linux, and macOS equally.
  - Never hardcode OS-specific roots (like `C:\`) or OS-specific temporary directories without abstract cross-platform path joining tools (`path.join()`).
- Document:
  - rationale and context for any code change.
  - The change, considerations during change implementation.
  - Noticable impact on the user, system, or other components.
- Build reusable code without any overhead:
  - Prefer using existing reusable components.
  - When reuse is not possible, consider abstraction and expansion of existing components.
  - When components get too big, refactor into smaller reusable components.
  - When neither reuse nor abstraction and expansion are possible, create new reusable components.
- Task completion criteria
  - Run linters if exist and configured for every source code change
  - Run format checks for every change in the repo.
  - Run tests for every code change.
- Complete requirements accurately as detailed: no more, no less.
- Do not miss do not mess.

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
