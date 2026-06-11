# Nanyi AI Coding Planner Skill

[中文](README.md)

## 1. What It Is

`nanyi-ai-coding-planner-skill` is a planning-focused Skill for AI coding work. It helps break a larger or more complex engineering goal into small, reviewable, handoff-friendly execution steps and generates only the current next prompt that can be copied into a fresh coding session.

The Skill handles planning, codebase scanning, task documentation, and next-prompt generation. It does not directly change application code.

The point is not to depend on the strongest model or a huge context window. The task state is written into documents. As long as the next tool can read those documents, you can use `next-prompt.md` to find the current prompt file, copy that prompt into a new conversation, and continue the next step.

Each time the Skill generates an execution prompt, it saves that prompt as a new file under `prompts/`. `next-prompt.md` is the current entry point. Historical prompts are kept for human traceability, but they are not part of the default execution context unless needed for diagnosis or audit.

## 2. What Problem It Solves

Complex AI coding tasks often fail because:

- One session tries to change too much at once.
- Important constraints disappear as context grows.
- Plans drift away from the real codebase.
- Follow-up sessions lack reliable handoff notes.
- Multiple tasks lack independent progress ledgers and can overwrite each other's documents.
- Different tools and models have uneven quota and availability across a team.

`nanyi-ai-coding-planner-skill` keeps those facts in independent task directories, task ledgers, and per-task handoff notes.

That makes the workflow less tied to a single AI coding tool. Use whichever tool still has quota, use `next-prompt.md` to copy the current prompt file into that tool, and continue. A smaller model can still handle a small execution step because the context and constraints are already captured in the documents.

## 3. Core Workflow

1. Read project rules such as `CLAUDE.md`, `AGENTS.md`, `README`, `Makefile`, or CI configuration.
2. Clarify the goal, non-goals, allowed edit scope, forbidden edit scope, and completion criteria.
3. Build the plan from real code scan evidence.
4. Maintain a project progress ledger and a per-task handoff document.
5. Split the work into small execution steps.
6. Generate only the current next execution prompt, not prompts for every future step.

## 4. Installation

Install with the skills command:

```bash
npx skills add https://github.com/iamnanyi/nanyi-ai-coding-planner-skill --skill nanyi-ai-coding-planner-skill
```

Manual installation:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/iamnanyi/nanyi-ai-coding-planner-skill.git
cp -R nanyi-ai-coding-planner-skill/nanyi-ai-coding-planner-skill ~/.claude/skills/
```

## 5. Quick Start

Describe the final goal. You do not need to break it into code-level steps yourself. If the requirement is long, you can first ask another AI assistant to summarize it into a clear goal statement, then use this Skill for planning.

New feature example:

```text
Use the nanyi-ai-coding-planner-skill Skill.

Goal:
Build a feature flag system for controlled rollout of new features. It should support environment-based enablement, user-group enablement, percentage rollout, and one unified read API for application code.
```

Existing feature iteration example:

```text
Use the nanyi-ai-coding-planner-skill Skill.

Goal:
Extend the existing home-page popup feature with subscription-based display rules. A popup configured as "visible only to unsubscribed users" should only be shown to unsubscribed users. Clicking the popup content should keep the existing navigation behavior. Clicking the close button should suppress the popup for 2 days. Clicking outside the popup should close only the current popup and should not trigger the 2-day suppression.
```

Refactoring example:

```text
Use the nanyi-ai-coding-planner-skill Skill.

Goal:
Refactor order status decision logic. Move status checks that are currently scattered across pages and APIs into one shared module, keep existing user behavior and API responses unchanged, and add the necessary regression validation.
```

## 6. Generated Document Structure

A typical output structure:

```text
docs/tasks/
  homepage-popup-rules/
    project-progress-ledger.md
    plan.md
    steps.md
    next-prompt.md
    prompts/
      step-01-read-and-map.md
    review-checklist.md
    handoff.md
```

Files:

- `homepage-popup-rules/` is one independent document directory for a large task. Each new large task should create its own `{task_slug}/` directory.
- `project-progress-ledger.md` lives inside the corresponding task directory and tracks that task's long-running progress, blockers, and task index.
- `plan.md` captures the goal, boundaries, risks, current code evidence, and proposed approach.
- `steps.md` contains the small-step execution plan.
- `next-prompt.md` is the current next-prompt entry point and points to the prompt that should be executed under `prompts/`.
- `prompts/` stores every generated execution prompt. Implementing agents should ignore older prompts by default to avoid extra context interference.
- `review-checklist.md` records review points for human validation.
- `handoff.md` tracks per-task execution handoff state.

## 7. Template Files

The Skill includes these templates:

```text
nanyi-ai-coding-planner-skill/templates/project-progress-ledger-template.md
nanyi-ai-coding-planner-skill/templates/task-plan-template.md
nanyi-ai-coding-planner-skill/templates/task-handoff-template.md
```

When creating task documents, the Skill should use these templates as structural references and fill them with facts from the actual project and project rules.

## 8. Where Project Commands Belong

Project-specific install, build, test, lint, type-check, and release commands should not be hard-coded into the Skill rules.

They should come from the project, such as:

- `CLAUDE.md`
- `AGENTS.md`
- `README`
- `Makefile`
- CI configuration
- Other project-level rule files

If the project does not provide validation commands, the Skill should say so instead of inventing defaults for a language or framework.

## 9. Recommended Usage

- Use this Skill before large features, refactors, bug fixes, or cross-module changes.
- Set one task collection root for the project, such as `docs/tasks` or `.ai_temp/docs/tasks`; each large task creates an independent `{task_slug}/` folder under it.
- Ask the implementer to update `handoff.md` after each execution round.
- Generate the next prompt only after the current step has been reviewed or confirmed.
- When all planned steps have been executed, ask the user to confirm the current result and state clearly that there are no further steps after confirmation; do not generate another next-step prompt.
- Keep old prompts under the task `prompts/` directory for human traceability, but do not feed them into the next execution round unless needed for diagnosis or audit.
- When a task is paused, blocked, or changed, update the `project-progress-ledger.md` inside that task directory.
- Each step should keep the code change small enough for human review. You can also give the planning docs, progress docs, and checklists under `docs/tasks` to another AI assistant for an extra review pass.

## 10. Safety Notes

- Do not write accounts, secrets, tokens, production configuration, or sensitive data into task documents.
- Do not include private paths, internal project names, or company information in public repositories.
- Clarify boundaries before touching payments, permissions, privacy, security, database schema, or public API contracts.
- Ask for confirmation before adding dependencies, deleting large amounts of code, or changing production configuration.

## 11. Good Fit

- Engineering planning before complex feature work
- Rule changes or boundary additions on top of existing features
- Impact analysis before cross-module refactors
- Splitting long AI coding tasks into multiple rounds
- Incremental development that needs human review
- Projects that need a progress ledger and task handoff records

## 12. Not a Good Fit

- Tiny edits that can be done in one or two lines
- Tasks where the user explicitly wants to skip planning and implement directly
- One-off questions that do not need code scanning or handoff docs
- Tasks that require immediate production operations, secrets, or production data access

## 13. License

MIT License. See [LICENSE](LICENSE).
