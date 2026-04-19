# OpenCode Multi-Agent Workflow

A practical multi-agent workflow for OpenCode that runs locally and coordinates a small AI dev team through shared context, scoped permissions, and specialized roles.

This setup includes:
- A **planner** that asks clarifying questions first
- A **debater** that checks whether there is a better plan
- An **implementor** that makes the code changes
- A **reviewer** that checks correctness and maintainability
- A **tester** that runs focused tests
- A **linter** that runs a project lint/check script
- A **commit-message** agent that prints a final commit message

The workflow uses:
- `AGENTS.md` for shared rules across the project
- `WORKFLOW_STATE.md` for persistent handoffs between agents
- Per-agent permissions for safer edits and command execution

OpenCode supports project-specific agents in `.opencode/agents/`, lets you configure primary and subagent roles, and loads `AGENTS.md` from the project root as shared instructions. [page:1][page:2]

---

## Why this exists

A single coding agent can do many things, but multi-step work often becomes messy when planning, implementation, review, testing, and documentation are mixed together.

This repo separates those concerns into focused agents with narrow responsibilities and a shared handoff file. The result is a workflow that is easier to inspect, easier to reuse, and easier to control.

---

## Workflow

The high-level flow looks like this:

```text
User
  ↓
Planner
  ↓
Debater
  ↓
Implementor
  ↓
Reviewer
  ↓
Tester
  ↓
Linter
  ↓
Commit Message
```

### Step-by-step

1. **Planner**
   - Reads the request
   - Asks clarifying questions first
   - Confirms scope and acceptance criteria
   - Writes the implementation plan to `WORKFLOW_STATE.md`

2. **Debater**
   - Reviews the planner’s proposal
   - Decides whether a meaningfully better plan exists
   - Writes feedback into `WORKFLOW_STATE.md`

3. **Implementor**
   - Applies the approved plan
   - Makes the smallest useful code changes
   - Records changed files and implementation notes

4. **Reviewer**
   - Reviews the implementation against the plan and acceptance criteria
   - Flags risks, side effects, or incomplete work

5. **Tester**
   - Runs the smallest relevant test set
   - Records commands and results

6. **Linter**
   - Runs the project’s lint/check script
   - Records pass/fail and findings

7. **Commit-message**
   - Reads the final diff and workflow state
   - Prints a conventional commit message

---

## Repository structure

```text
.
├─ AGENTS.md
├─ WORKFLOW_STATE.md
└─ .opencode/
   └─ agents/
      ├─ planner.md
      ├─ debater.md
      ├─ implementor.md
      ├─ reviewer.md
      ├─ tester.md
      ├─ linter.md
      └─ commit-message.md
```

### Files

- `AGENTS.md`
  - Shared rules for all agents in this project
  - Loaded into OpenCode’s context for project-specific behavior [page:2]

- `WORKFLOW_STATE.md`
  - Shared state and handoff record between agents
  - Keeps planning, decisions, status, and outputs in one place

- `.opencode/agents/*.md`
  - Project-specific agent definitions
  - OpenCode uses the markdown filename as the agent name [page:1]

---

## Core design ideas

### 1. Shared rules

`AGENTS.md` acts like the project constitution.

It defines workflow-wide behavior, such as:
- all agents must read `WORKFLOW_STATE.md`
- all agents must update only the sections relevant to their role
- agents should prefer Context7 when docs lookup is useful
- the workflow order and handoff expectations

OpenCode explicitly supports a root `AGENTS.md` file for project-specific instructions. [page:2]

### 2. Shared state

`WORKFLOW_STATE.md` acts like the workflow notebook.

Instead of relying only on chat history, agents use a persistent file to coordinate:
- clarified scope
- open questions
- constraints
- acceptance criteria
- plan
- debate notes
- implementation notes
- review findings
- test and lint results
- commit message draft

This makes the workflow more transparent and easier to debug.

### 3. Scoped permissions

Each agent gets only the permissions it needs.

For example:
- planner, debater, reviewer, tester, linter, and commit-message may only edit `WORKFLOW_STATE.md`
- implementor can edit code
- tester and linter can run only approved commands
- planner can invoke only the intended downstream agents

OpenCode supports per-agent permissions for `edit`, `bash`, `webfetch`, and task invocation. [page:1]

---

## Example permissions

For an agent that should only update `WORKFLOW_STATE.md`:

```yaml
permission:
  edit:
    "*": deny
    "WORKFLOW_STATE.md": allow
```

OpenCode supports granular path-based permissions, and the last matching rule takes precedence. [page:1]

For a linter that should run only a Python script:

```yaml
permission:
  edit:
    "*": deny
    "WORKFLOW_STATE.md": allow
  bash:
    "*": deny
    "python scripts/lint.py*": allow
    "python3 scripts/lint.py*": allow
```

OpenCode also supports command-pattern rules for bash permissions. [page:1]

---

## Models and temperature

Each agent can use its own model and temperature.

Typical strategy:
- low temperature for planner, reviewer, tester, and linter
- slightly higher temperature for debater
- stronger model for planner and implementor
- faster or cheaper model for narrow support roles

OpenCode supports per-agent `model`, `temperature`, `max_steps`, and `mode` settings. [page:1]

---

## MCP support

This workflow can be extended with MCP tools such as:
- **Context7** for up-to-date documentation lookup

If configured in OpenCode, these tools can be encouraged through `AGENTS.md` and agent prompts so the planner, implementor, and reviewer use them when helpful. OpenCode exposes MCP servers as tools once they are configured and enabled. [page:1]

---

## Getting started

### 1. Copy the files into your repo

Place:
- `AGENTS.md` in the project root
- `WORKFLOW_STATE.md` in the project root
- all agent markdown files inside `.opencode/agents/`

### 2. Adjust commands

Update:
- test commands in `tester.md`
- lint/check commands in `linter.md`
- model IDs in each agent file
- any MCP-related instructions if you use Context7

### 3. Start OpenCode in the project

Run OpenCode from the root of your repository so project-level rules and agents are picked up correctly.

### 4. Start with the planner

Example:

```bash
opencode run --agent planner "Add a settings page where users can update their profile and notification preferences"
```

The planner should:
- ask clarifying questions first
- write the clarified scope to `WORKFLOW_STATE.md`
- generate a plan
- hand off to the next agent

---

## Recommended usage

This setup works best when:
- the task is non-trivial
- you want a visible plan and review trail
- you care about safe permissions
- you want reusable workflow behavior across repos

It is especially useful for:
- feature work
- refactors
- bug fixes with multiple steps
- code review and validation flows
- local-first AI development setups

---

## Customization ideas

You can extend this repo with additional agents such as:
- `security-reviewer`
- `docs-writer`
- `release-notes`
- `migration-planner`
- `performance-reviewer`

OpenCode lets you create additional agents and restrict which subagents can be invoked with task permissions. [page:1]

---

## Notes

- `AGENTS.md` is for shared instructions, not shared memory
- `WORKFLOW_STATE.md` is the canonical handoff file
- the agent markdown filename becomes the agent name
- the planner is intentionally clarification-first
- the debater is intentionally narrow: it reviews the plan and checks whether a better one exists

---

## Credits

Built for OpenCode’s project-level agent system and rule files.

Relevant docs:
- OpenCode Agents: https://opencode.ai/docs/agents/
- OpenCode Rules: https://opencode.ai/docs/rules/

---

## License

Add your preferred license here.