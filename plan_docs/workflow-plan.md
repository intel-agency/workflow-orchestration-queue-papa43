# Workflow Execution Plan: project-setup

**Repository:** `intel-agency/workflow-orchestration-queue-papa43`
**Workflow:** `project-setup` dynamic workflow
**Plan Created:** 2026-04-27
**Status:** Approved

---

## 1. Overview

| Field | Value |
|---|---|
| **Workflow Name** | `project-setup` |
| **Dynamic Workflow File** | `ai_instruction_modules/ai-workflow-assignments/dynamic-workflows/project-setup.md` (remote canonical) |
| **Project Name** | workflow-orchestration-queue (OS-APOW) |
| **Total Main Assignments** | 6 |
| **Event Assignments** | 5 (1 pre-script + 2 post-assignment x 6 main + 1 post-script action) |

### Summary

The `project-setup` dynamic workflow initializes the `workflow-orchestration-queue` repository for active development. It creates a feature branch, configures GitHub repository settings (branch protection, labels, projects), produces a detailed application implementation plan from the seeded `plan_docs/`, scaffolds the project directory structure, generates an `AGENTS.md` for AI coding agents, documents the workflow execution, and merges the setup PR. Upon completion, the `orchestration:plan-approved` label is applied to the application plan issue, signaling readiness for epic creation.

---

## 2. Project Context Summary

### 2.1 What Is Being Built

**workflow-orchestration-queue** (also referred to as OS-APOW) is a headless agentic orchestration platform that transforms GitHub Issues into automated execution orders. It replaces the human-in-the-loop AI coding paradigm with a persistent, event-driven system where AI agents autonomously discover, claim, execute, and report on work items.

The system has four conceptual pillars:

1. **The Ear (Work Event Notifier):** FastAPI webhook receiver for secure event ingestion
2. **The State (Work Queue):** Distributed state via GitHub Issues/Labels ("Markdown as a Database")
3. **The Brain (Sentinel Orchestrator):** Persistent Python background service for polling, claiming, and dispatching
4. **The Hands (Opencode Worker):** LLM-driven agent in an isolated DevContainer

### 2.2 Technology Stack

| Layer | Technology |
|---|---|
| **Language** | Python 3.12+ |
| **Web Framework** | FastAPI + Uvicorn |
| **Data Validation** | Pydantic |
| **HTTP Client** | HTTPX (async) |
| **Package Manager** | uv (Rust-based) |
| **Containerization** | Docker, DevContainers |
| **Agent Runtime** | opencode CLI (GLM-5 / Claude) |
| **CI/CD** | GitHub Actions |
| **Shell Scripts** | PowerShell Core (pwsh) / Bash |
| **Version Control** | Git + GitHub |

### 2.3 Key Planning Documents

| Document | File | Purpose |
|---|---|---|
| Development Plan v4 | `plan_docs/OS-APOW Development Plan v4.md` | Multi-phase roadmap: Phase 0 (Seeding), Phase 1 (Sentinel MVP), Phase 2 (Webhook Ear), Phase 3 (Deep Orchestration) |
| Architecture Guide v3 | `plan_docs/OS-APOW Architecture Guide v3.md` | System-level diagrams, ADRs (Shell-Bridge, Polling-First, Provider-Agnostic), security model |
| Implementation Spec v1 | `plan_docs/OS-APOW Implementation Specification v1.md` | Detailed requirements, test cases, project structure, deliverables |
| Plan Review | `plan_docs/OS-APOW Plan Review.md` | Critical review identifying 7 strengths, 10 issues, and 9 improvement recommendations |
| Notifier Service | `plan_docs/notifier_service.py` | Reference implementation of the FastAPI webhook receiver |
| Sentinel Orchestrator | `plan_docs/orchestrator_sentinel.py` | Reference implementation of the polling/claiming/dispatch engine |
| Interactive Report | `plan_docs/interactive-report.html` | React-based presentation dashboard |

### 2.4 Known Issues from Plan Review (Critical for Execution)

The Plan Review (`OS-APOW Plan Review.md`) identified the following issues that subsequent assignments must address:

| ID | Issue | Severity |
|---|---|---|
| I-1 | Divergent `WorkItem` models between sentinel and notifier | High |
| I-2 | Race condition in task claiming (no assign-then-verify) | High |
| I-3 | No jittered exponential backoff on poller | Medium |
| I-4 | `httpx.AsyncClient()` created per-call (no connection pooling) | Medium |
| I-5 | Hardcoded secrets in notifier scaffold | High |
| I-6 | No heartbeat implementation | High |
| I-7 | Cost guardrails story has no implementation | Medium |
| I-8 | Single-repo polling vs. spec'd cross-org discovery | Low |
| I-9 | Bare `except: pass` in `claim_task` | Medium |
| I-10 | No environment reset between tasks | Medium |

### 2.5 Project Structure (Target from Implementation Spec)

```
workflow-orchestration-queue/
├── pyproject.toml
├── uv.lock
├── src/
│   ├── notifier_service.py
│   ├── orchestrator_sentinel.py
│   ├── models/
│   │   ├── work_item.py
│   │   └── github_events.py
│   └── interfaces/
│       └── i_task_queue.py
├── scripts/
│   ├── devcontainer-opencode.sh
│   ├── gh-auth.ps1
│   └── update-remote-indices.ps1
├── local_ai_instruction_modules/
│   ├── create-app-plan.md
│   ├── perform-task.md
│   └── analyze-bug.md
└── docs/
```

### 2.6 Special Constraints

- **No `global.json`:** This is a Python ecosystem project; `.NET` configuration files are unnecessary (Implementation Spec, Language section).
- **Action SHA Pinning:** All GitHub Actions workflows created or modified during this workflow MUST pin actions to specific commit SHAs.
- **Branch naming convention:** `dynamic-workflow-project-setup`
- **Self-approval:** The setup PR is an automated setup PR; self-approval by the orchestrator is acceptable (no human stakeholder approval required for merge).

---

## 3. Assignment Execution Plan

### 3.0 Pre-Script Event: `create-workflow-plan` (THIS ASSIGNMENT)

| Field | Detail |
|---|---|
| **Assignment** | `create-workflow-plan`: Create Workflow Plan |
| **Goal** | Produce a comprehensive workflow execution plan before any other assignments begin |
| **Key Acceptance Criteria** | Dynamic workflow fully read; all assignments traced; all `plan_docs/` read; plan presented, approved, and committed |
| **Output** | This file: `plan_docs/workflow-plan.md` |
| **Branch** | Created: `dynamic-workflow-project-setup` |

---

### 3.1 `init-existing-repository`

| Field | Detail |
|---|---|
| **Assignment** | `init-existing-repository`: Initiate Existing Repository |
| **Goal** | Set up the repository's GitHub configuration, branch, labels, project, and create the setup PR |
| **Key Acceptance Criteria** | (0) Branch `dynamic-workflow-project-setup` created (must be first); (1) Branch protection ruleset imported from `.github/protected-branches_ruleset.json`; (2) GitHub Project created and linked to repo; (3) Project columns: Not Started, In Progress, In Review, Done; (4) Labels imported from `.github/.labels.json` via `scripts/import-labels.ps1`; (5) Devcontainer name renamed to `<repo-name>-devcontainer`; (6) Workspace file renamed to `<repo-name>.code-workspace`; (7) PR created from branch to `main` |
| **Project-Specific Notes** | The `.github/.labels.json` contains 15 labels (assigned, assigned:copilot, bug, documentation, duplicate, enhancement, good first issue, help wanted, invalid, question, state, state:in-progress, state:planning, type:enhancement, wontfix). The devcontainer name in `.devcontainer/devcontainer.json` must be renamed to `workflow-orchestration-queue-devcontainer`. The workspace file `ai-new-app-template.code-workspace` must be renamed to `workflow-orchestration-queue.code-workspace`. Branch protection import requires `GH_ORCHESTRATION_AGENT_TOKEN` with `administration: write` scope. |
| **Prerequisites** | GitHub authentication with scopes: `repo`, `project`, `read:project`, `read:user`, `user:email`, `administration: write` |
| **Dependencies** | None (first assignment) |
| **Risks** | (1) Branch protection import may fail if `GH_ORCHESTRATION_AGENT_TOKEN` lacks `administration: write` scope — must stop and report, not silently skip. (2) PR creation requires at least one commit ahead of `main` — ensure file renames are committed first. (3) Labels import via PowerShell requires `pwsh` available in the environment. |

---

### 3.2 `create-app-plan`

| Field | Detail |
|---|---|
| **Assignment** | `create-app-plan`: Create Application Plan |
| **Goal** | Create a comprehensive application implementation plan based on `plan_docs/` and document it as a GitHub Issue |
| **Key Acceptance Criteria** | (1) Application template analyzed; (2) Project structure documented; (3) Plan uses Appendix A template; (4) Detailed phase breakdown; (5) All components and dependencies planned; (6) Tech stack followed; (7) Mandatory requirements addressed (testing, docs, containerization); (8) Acceptance criteria addressed; (9) Risks and mitigations identified; (10) Code quality standards; (11) Plan ready for implementation; (12) Plan documented as GitHub Issue; (13) Milestones created; (14) Issue added to GitHub Project; (15) Issue assigned to milestone; (16) Labels applied |
| **Project-Specific Notes** | The `plan_docs/` directory contains 7 files (3 markdown docs, 2 Python reference implementations, 1 HTML dashboard, 1 this plan). The application template is not named `ai-new-app-template.md` but rather the Implementation Specification (`OS-APOW Implementation Specification v1.md`) serves as the primary spec. The plan should reflect the 4-phase roadmap (Seeding, Sentinel MVP, Webhook Ear, Deep Orchestration) from the Development Plan. Tech-stack and architecture documents should be created as `plan_docs/tech-stack.md` and `plan_docs/architecture.md`. The Plan Review's 10 issues and 9 recommendations must be incorporated into the plan phases. The Implementation Spec specifies no `global.json` (Python ecosystem). |
| **Prerequisites** | Repository initialized (assignment 3.1 complete); `plan_docs/` available |
| **Dependencies** | GitHub Project from 3.1 (for linking the issue); labels from 3.1 (for labeling the issue) |
| **Risks** | (1) The `plan_docs/` doesn't contain a standard `ai-new-app-template.md` — the agent must treat the Implementation Specification as the primary spec. (2) The plan must reconcile the reference implementations with the spec, noting which code issues to fix. (3) This is PLANNING ONLY — no code implementation. |
| **Events** | `pre-assignment-begin`: `gather-context`; `on-assignment-failure`: `recover-from-error`; `post-assignment-complete`: `report-progress` |

---

### 3.3 `create-project-structure`

| Field | Detail |
|---|---|
| **Assignment** | `create-project-structure`: Create Project Structure |
| **Goal** | Create the actual project directory structure, scaffolding, and infrastructure configuration |
| **Key Acceptance Criteria** | (1) Solution/project structure created per tech stack; (2) All project files and directories established; (3) Initial configuration files created; (4) Basic CI/CD pipeline structure; (5) Documentation structure created; (6) Development environment validated; (7) Initial commit with scaffolding; (8) Stakeholder approval; (9) Repository summary created; (10) All GitHub Actions pinned to commit SHAs |
| **Project-Specific Notes** | Tech stack is Python 3.12+ with `pyproject.toml` and `uv`. Target structure from Implementation Spec: `src/` (main code), `src/models/` (Pydantic schemas), `src/interfaces/` (abstract base classes), `scripts/` (shell bridge), `local_ai_instruction_modules/` (markdown logic), `docs/` (documentation). Must create Dockerfile, docker-compose.yml, and healthcheck using Python stdlib (not curl). The existing scripts directory already has PowerShell scripts — extend, don't replace. Must create `.ai-repository-summary.md` at repo root. CI/CD workflows must use SHA-pinned actions. |
| **Prerequisites** | Application plan exists (from 3.2); repository initialized (from 3.1) |
| **Dependencies** | Application plan issue from 3.2 (for structure guidance); branch from 3.1 |
| **Risks** | (1) The project already has existing scripts and config files — must preserve them while adding new structure. (2) The reference implementations in `plan_docs/` are scaffolds with known issues (I-1 through I-10) — the project structure must accommodate fixing these during implementation. (3) Docker healthcheck must not use curl (base image may not have it). |

---

### 3.4 `create-agents-md-file`

| Field | Detail |
|---|---|
| **Assignment** | `create-agents-md-file`: Create AGENTS.md File |
| **Goal** | Create an `AGENTS.md` file at the repository root providing AI coding agents with the context they need |
| **Key Acceptance Criteria** | (1) `AGENTS.md` exists at repo root; (2) Project overview with purpose and tech stack; (3) Setup/build/test commands verified; (4) Code style and conventions; (5) Project structure section; (6) Testing instructions; (7) PR/commit guidelines; (8) Standard Markdown; (9) Commands validated; (10) Committed and pushed; (11) Stakeholder approval |
| **Project-Specific Notes** | Must reflect Python 3.12+ / FastAPI / Pydantic / uv / Docker stack. Build commands: `uv sync`, `uv run pytest`, `uv run ruff check`. The existing `AGENTS.md` at repo root already has extensive template-agent documentation — this assignment should UPDATE it with project-specific content (not replace the template instructions wholesale, but add project-specific sections for the OS-APOW application). Key architecture notes: 4-pillar system (Ear/State/Brain/Hands), Shell-Bridge pattern, Polling-First resiliency, Provider-Agnostic interfaces. |
| **Prerequisites** | Repository initialized (3.1); application plan exists (3.2); project structure created (3.3) |
| **Dependencies** | Project structure from 3.3 (commands to validate); application plan from 3.2 (tech stack details) |
| **Risks** | (1) The existing `AGENTS.md` already has substantial template-agent content — must be careful to preserve what's needed while adding project-specific sections. (2) Build/test commands may not be fully functional yet since this is scaffolding — the agent may need to document expected commands rather than verified ones. |

---

### 3.5 `debrief-and-document`

| Field | Detail |
|---|---|
| **Assignment** | `debrief-and-document`: Debrief and Document Learnings |
| **Goal** | Capture key learnings, insights, deviations, and improvement areas from the entire workflow execution |
| **Key Acceptance Criteria** | (1) Detailed report with all 12 required sections; (2) Report in `.md` format; (3) All sections complete; (4) All deviations documented; (5) Stakeholder approval; (6) Committed and pushed; (7) Execution trace saved as `debrief-and-document/trace.md` |
| **Project-Specific Notes** | The debrief must document: which plan review issues (I-1 through I-10) were addressed in the scaffolding vs. deferred to later phases; any deviations from the standard project-setup workflow (e.g., the lack of a standard `ai-new-app-template.md` file); and the Plan Adjustment Mandate findings that should influence subsequent phases. The `continuous-improvement` assignment is triggered from this debrief. |
| **Prerequisites** | All prior assignments (3.1-3.4) completed |
| **Dependencies** | Outputs and artifacts from all prior assignments |
| **Risks** | Minimal risk. The main concern is ensuring accurate trace documentation of all assignment actions. |

---

### 3.6 `pr-approval-and-merge`

| Field | Detail |
|---|---|
| **Assignment** | `pr-approval-and-merge`: PR Approval and Merge |
| **Goal** | Complete the full PR approval and merge process for the setup PR |
| **Key Acceptance Criteria** | (1) All CI status checks pass; (2) CI remediation loop executed (up to 3 attempts); (3) Code review delegated (NOT self-review); (4) Review comments resolved; (5) Approval obtained; (6) Merge performed; (7) Source branch deleted; (8) Related issues updated |
| **Project-Specific Notes** | **Special Handling:** This is an automated setup PR. Self-approval by the orchestrator is acceptable — no human stakeholder approval is required. The `$pr_num` input comes from the PR created in assignment 3.1. The CI remediation loop (Phase 0.5) MUST still be executed: if CI checks fail, attempt up to 3 fix cycles before escalating. On successful merge, delete the `dynamic-workflow-project-setup` branch. |
| **Prerequisites** | All assignments (3.1-3.5) completed; PR number available |
| **Dependencies** | PR number from 3.1; all commits from 3.1-3.5 pushed to the branch |
| **Risks** | (1) CI may fail on the new Python project structure — the remediation loop must handle missing dependencies, lint errors, and test failures. (2) Branch protection rules (imported in 3.1) may require specific checks to pass — these must be satisfied. (3) If CI cannot be fixed in 3 attempts, escalation is required. |

---

### 3.7 Post-Script Event: Apply `orchestration:plan-approved` Label

| Field | Detail |
|---|---|
| **Action** | Apply `orchestration:plan-approved` label to the application plan issue |
| **Goal** | Signal that the plan is ready for epic creation (triggers the next phase of the orchestration pipeline) |
| **Dependencies** | Application plan issue from 3.2; all assignments complete |

---

### 3.8 Cross-Cutting Event Assignments

These assignments run at defined points during workflow execution:

| Event | Assignment | When |
|---|---|---|
| `post-assignment-complete` | `validate-assignment-completion` | After each main assignment (3.1-3.6) |
| `post-assignment-complete` | `report-progress` | After each main assignment (3.1-3.6) |

---

## 4. Sequencing Diagram

```
START
  │
  ▼
[pre-script-begin] ── create-workflow-plan ──► plan_docs/workflow-plan.md committed
  │
  ▼
[3.1] init-existing-repository
  │   ├─ Create branch: dynamic-workflow-project-setup
  │   ├─ Import branch protection ruleset
  │   ├─ Create GitHub Project + columns
  │   ├─ Import labels from .github/.labels.json
  │   ├─ Rename devcontainer + workspace files
  │   └─ Create PR (→ outputs $pr_num)
  │
  ▼
  ── validate-assignment-completion ──► ── report-progress ──►
  │
  ▼
[3.2] create-app-plan
  │   ├─ Analyze plan_docs/ (7 files)
  │   ├─ Create tech-stack.md + architecture.md
  │   ├─ Create GitHub Issue with implementation plan
  │   ├─ Create milestones (Phase 0-3)
  │   └─ Link issue to Project + assign milestone + labels
  │
  ▼
  ── validate-assignment-completion ──► ── report-progress ──►
  │
  ▼
[3.3] create-project-structure
  │   ├─ Create pyproject.toml, src/, models/, interfaces/
  │   ├─ Create Dockerfile, docker-compose.yml
  │   ├─ Create CI/CD workflows (SHA-pinned actions)
  │   ├─ Create docs/ structure
  │   ├─ Create .ai-repository-summary.md
  │   └─ Initial commit with scaffolding
  │
  ▼
  ── validate-assignment-completion ──► ── report-progress ──►
  │
  ▼
[3.4] create-agents-md-file
  │   ├─ Gather context from existing docs
  │   ├─ Validate build/test commands
  │   ├─ Draft AGENTS.md (project-specific)
  │   └─ Cross-reference with existing documentation
  │
  ▼
  ── validate-assignment-completion ──► ── report-progress ──►
  │
  ▼
[3.5] debrief-and-document
  │   ├─ Create 12-section debrief report
  │   ├─ Document all deviations
  │   ├─ Save execution trace
  │   └─ Commit and push
  │
  ▼
  ── validate-assignment-completion ──► ── report-progress ──►
  │
  ▼
[3.6] pr-approval-and-merge
  │   ├─ CI verification + remediation loop (≤3 attempts)
  │   ├─ Code review delegation
  │   ├─ Review comment resolution
  │   ├─ Self-approve (automated setup PR)
  │   ├─ Merge PR
  │   └─ Delete setup branch
  │
  ▼
  ── validate-assignment-completion ──► ── report-progress ──►
  │
  ▼
[post-script-complete] ── Apply orchestration:plan-approved label to plan issue
  │
  ▼
DONE
```

---

## 5. Open Questions

### 5.1 Missing Standard Template File

The `plan_docs/` directory does not contain a standard `ai-new-app-template.md` or `new app spec.md` file. The Implementation Specification (`OS-APOW Implementation Specification v1.md`) serves as the primary application specification. **Question:** Should the `create-app-plan` agent treat the Implementation Specification as the canonical app template, or should a formal `ai-new-app-template.md` be synthesized from the existing plan documents before proceeding?

### 5.2 Branch Protection Token Scope

The branch protection ruleset import (assignment 3.1, step 2) requires `GH_ORCHESTRATION_AGENT_TOKEN` with `administration: write` scope. If this token is not available or lacks the required scope, the import will fail. **Question:** Is `GH_ORCHESTRATION_AGENT_TOKEN` configured as a repository secret with the appropriate scope?

### 5.3 Reference Implementation Handling

The `plan_docs/` directory contains two reference Python implementations (`notifier_service.py` and `orchestrator_sentinel.py`) that are intended as scaffolds with known issues (documented in the Plan Review). **Question:** Should the `create-project-structure` assignment copy these scaffolds into `src/` as starting points (to be fixed in later phases), or should it create clean stubs based on the Implementation Spec's target structure?

### 5.4 Existing Repository Files

The repository already contains substantial infrastructure from the template (scripts/, .github/workflows/, .devcontainer/, .opencode/, local_ai_instruction_modules/, etc.). **Question:** Should the `create-project-structure` assignment create a parallel Python project structure alongside the existing template infrastructure, or should it integrate the Python application into the existing structure?

### 5.5 AGENTS.md Scope

The repository root already has an `AGENTS.md` file with extensive template-agent instructions (AGENTS.md in the repository root). **Question:** Should the `create-agents-md-file` assignment update the existing `AGENTS.md` in-place by adding project-specific sections, or should it create a new file that replaces the template content?

---

## Appendix: Assignment Resolution Trace

| Assignment | Resolution Source | File/URL |
|---|---|---|
| `project-setup` (dynamic workflow) | Remote canonical | `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-workflow-assignments/dynamic-workflows/project-setup.md` |
| `create-workflow-plan` | Remote canonical | `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-workflow-assignments/create-workflow-plan.md` |
| `init-existing-repository` | Remote canonical | `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-workflow-assignments/init-existing-repository.md` |
| `create-app-plan` | Remote canonical | `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-workflow-assignments/create-app-plan.md` |
| `create-project-structure` | Remote canonical | `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-workflow-assignments/create-project-structure.md` |
| `create-agents-md-file` | Remote canonical | `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-workflow-assignments/create-agents-md-file.md` |
| `debrief-and-document` | Remote canonical | `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-workflow-assignments/debrief-and-document.md` |
| `pr-approval-and-merge` | Remote canonical | `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-workflow-assignments/pr-approval-and-merge.md` |
| `validate-assignment-completion` | Remote canonical | `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-workflow-assignments/validate-assignment-completion.md` |
| `report-progress` | Remote canonical | `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-workflow-assignments/report-progress.md` |
