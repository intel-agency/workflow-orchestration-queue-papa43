# Workflow Execution Plan: `project-setup`

**Repository:** `intel-agency/workflow-orchestration-queue-papa43`  
**Branch:** `dynamic-workflow-project-setup`  
**Plan Created:** 2026-04-20  
**Dynamic Workflow:** `project-setup`  

---

## 1. Overview

This document is the execution plan for the **`project-setup`** dynamic workflow, which orchestrates the initial setup of the **workflow-orchestration-queue** project. The system being built is a headless agentic orchestration platform that transforms GitHub Issues into autonomous Execution Orders fulfilled by specialized AI agents running in isolated DevContainers.

The workflow consists of **6 sequential assignments**, executed in order, with pre/post event hooks for validation, progress reporting, and final label application. This plan is produced as the `pre-script-begin` event (`create-workflow-plan`) before any assignments execute.

### Workflow Summary

| Property | Value |
|---|---|
| Workflow Name | `project-setup` |
| Project | workflow-orchestration-queue (OS-APOW) |
| Total Assignments | 6 |
| Pre-Script Events | 1 (`create-workflow-plan`) |
| Post-Assignment Events | 2 (`validate-assignment-completion`, `report-progress`) — run after each assignment |
| Post-Script Event | 1 (`orchestration:plan-approved` label application) |
| Working Branch | `dynamic-workflow-project-setup` |
| Target Merge Base | `main` |

---

## 2. Project Context Summary

### 2.1 Project Description

**workflow-orchestration-queue** is a headless agentic orchestration platform that shifts AI from a passive co-pilot role to an autonomous background production service. The system uses a 4-pillar architecture:

1. **The Ear (Notifier):** FastAPI webhook receiver for secure event ingestion with HMAC validation
2. **The State (Work Queue):** GitHub Issues as a database ("Markdown as a Database") with label-based state machine
3. **The Brain (Sentinel):** Async Python background service for polling, claiming, and dispatching tasks
4. **The Hands (Worker):** Isolated DevContainer running opencode-server for code execution

### 2.2 Technology Stack

- **Primary Language:** Python 3.12+
- **Web Framework:** FastAPI + Uvicorn
- **Data Validation:** Pydantic
- **HTTP Client:** HTTPX (async)
- **Package Management:** uv (Rust-based)
- **Containerization:** Docker / DevContainers
- **Shell Scripts:** PowerShell Core (pwsh) + Bash
- **AI Runtime:** opencode-server CLI with LLM (GLM-5 / Claude 3.5 Sonnet)
- **State Management:** GitHub Issues, Labels, Milestones, Projects
- **CI/CD:** GitHub Actions

### 2.3 Key Architecture Decisions

| ADR | Decision | Rationale |
|-----|----------|-----------|
| ADR 07 | Shell-Bridge Execution via `devcontainer-opencode.sh` | Reuses existing shell infrastructure; ensures environment parity between AI and human developers |
| ADR 08 | Polling-First Resiliency Model | Webhooks are fire-and-forget; polling ensures self-healing on restart via GitHub label reconciliation |
| ADR 09 | Provider-Agnostic Interface (ITaskQueue) | Strategy pattern enables swapping GitHub for Linear/Notion/SQL queues without rewriting orchestrator logic |

### 2.4 Phased Development Roadmap

- **Phase 0 (Seeding):** Manual clone of template, seed plan docs, initial orchestration — *this is what `project-setup` achieves*
- **Phase 1 (Sentinel MVP):** Persistent polling engine, shell-bridge dispatcher, automated status feedback
- **Phase 2 (The Ear):** FastAPI webhook receiver, intelligent template triaging, dev tunneling
- **Phase 3 (Deep Orchestration):** Architect sub-agent, autonomous bug correction, proactive indexing

### 2.5 Reference Implementation Artifacts

The `plan_docs/` directory includes scaffold reference code:
- **`notifier_service.py`** — FastAPI webhook receiver with HMAC validation, WorkItem triaging, and ITaskQueue interface
- **`orchestrator_sentinel.py`** — Background polling service with GitHubQueue, shell-bridge execution, and WorkItem lifecycle management
- **`interactive-report.html`** — React-based interactive architecture and development plan presentation dashboard

### 2.6 Known Issues from Plan Review (OS-APOW Plan Review)

The plan review identified 10 issues and 9 improvement recommendations in the reference implementations. Key findings that will impact project setup:

1. **I-1:** Divergent `WorkItem` models between sentinel and notifier — must unify in `src/models/`
2. **I-2:** Race condition in task claiming — needs assign-then-verify pattern
3. **I-3:** No jittered exponential backoff on poller — must add for rate limit resilience
4. **I-5:** Hardcoded secrets in notifier scaffold — must use `os.environ` from the start
5. **I-6:** No heartbeat implementation for long-running tasks
6. **I-10:** No environment reset between tasks (docker-compose down)

These should be tracked as TODO items or future issues during the `create-app-plan` assignment.

### 2.7 Project Structure (Planned)

```
workflow-orchestration-queue/
├── pyproject.toml               # uv dependencies and metadata
├── uv.lock                      # Deterministic lockfile
├── src/
│   ├── notifier_service.py      # FastAPI webhook ingestion
│   ├── orchestrator_sentinel.py # Background polling and dispatch
│   ├── models/
│   │   ├── work_item.py         # Unified WorkItem, Status, Types
│   │   └── github_events.py     # GitHub webhook payload schemas
│   └── interfaces/
│       └── i_task_queue.py      # Abstract operations (add, claim, update)
├── scripts/                     # Shell bridge execution layer
│   ├── devcontainer-opencode.sh
│   ├── gh-auth.ps1
│   └── update-remote-indices.ps1
├── local_ai_instruction_modules/ # Markdown logic workflows for LLM
│   ├── create-app-plan.md
│   ├── perform-task.md
│   └── analyze-bug.md
└── docs/                        # Architecture and user documentation
```

---

## 3. Assignment Execution Plan

### Assignment 1: `init-existing-repository`

**Title:** Initiate Existing Repository  
**Goal:** Create the working branch, import branch protection rules, set up a GitHub Project for issue tracking, import labels, rename workspace/devcontainer files, and open the initial setup PR.

#### Key Acceptance Criteria

- New branch `dynamic-workflow-project-setup` created (must be first — all work commits here)
- Branch protection ruleset imported from `.github/protected-branches_ruleset.json` (idempotent)
- GitHub Project created with Board template, linked to repository
- Project columns: Not Started, In Progress, In Review, Done
- Labels imported from `.github/.labels.json` via `scripts/import-labels.ps1`
- `devcontainer.json` `name` property renamed to `<repo-name>-devcontainer`
- `ai-new-app-template.code-workspace` renamed to `<repo-name>.code-workspace`
- PR created from branch to `main`

#### Project-Specific Notes

- The template repository already has `.github/protected-branches_ruleset.json` and `.github/.labels.json` bundled
- The repository name is `workflow-orchestration-queue-papa43`
- Authentication requires `GH_ORCHESTRATION_AGENT_TOKEN` (PAT with `administration: write`) for ruleset import
- The `scripts/test-github-permissions.ps1` script should be run to verify auth before proceeding
- PR number from this step is passed to `pr-approval-and-merge` as `$pr_num`

#### Prerequisites

- GitHub authentication with scopes: `repo`, `project`, `read:project`, `read:user`, `user:email`
- `administration: write` scope for branch protection rulesets
- GitHub CLI (`gh`) installed and authenticated

#### Dependencies

- None — this is the first assignment

#### Risks/Challenges

- **Ruleset import failure:** Requires `GH_ORCHESTRATION_AGENT_TOKEN` with `administration: write`; the default `GITHUB_TOKEN` in Actions has limited permissions. If PAT is unavailable, this step should be skipped gracefully with a warning.
- **PR creation failure:** Requires at least one commit pushed to the branch before PR can be created. The label import and file renames provide these commits.
- **Project creation:** May fail if GitHub Projects (V2) API access is not available with current token scopes.

#### Events Fired

- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### Assignment 2: `create-app-plan`

**Title:** Create Application Plan  
**Goal:** Analyze the plan documents and reference implementations in `plan_docs/` to produce a comprehensive application plan documented as a GitHub Issue with milestones, linked to the GitHub Project.

#### Key Acceptance Criteria

- Application template (plan_docs) thoroughly analyzed and understood
- Tech stack documented in `plan_docs/tech-stack.md`
- High-level architecture documented in `plan_docs/architecture.md`
- Application plan follows the 4-phase roadmap (Sentinel → Ear → Deep Orchestration)
- Plan addresses: testing, documentation, containerization, security, cost guardrails
- All risks and mitigations identified (API rate limits, LLM looping, concurrency, container drift, security injection)
- Plan documented as GitHub Issue using the `application-plan.md` template from `.github/ISSUE_TEMPLATE/`
- Milestones created for each phase and linked to issues
- Plan issue added to GitHub Project and assigned to "Phase 1: Foundation" milestone
- Labels applied: `planning`, `documentation`

#### Project-Specific Notes

- The `plan_docs/` directory contains rich, detailed documents that serve as the "filled-out application template":
  - **Architecture Guide v3:** 4-pillar architecture, ADRs, security model, self-bootstrapping lifecycle
  - **Development Plan v4:** Phased roadmap with user stories, acceptance criteria, risk assessment
  - **Implementation Specification v1:** Features, test cases, logging, containerization, deliverables
  - **Plan Review:** 7 strengths, 10 issues, 9 improvement recommendations — critical for risk identification
  - **Reference code:** `notifier_service.py` and `orchestrator_sentinel.py` scaffold implementations
  - **Dashboard:** `interactive-report.html` presentation artifact
- The plan review's 10 issues (I-1 through I-10) and 9 recommendations (R-1 through R-9) must be incorporated as explicit TODO items or tracked issues
- The `orchestration:plan-approved` label is NOT applied by this assignment — it is applied by the `post-script-complete` event after all assignments finish
- The `pre-assignment-begin` event fires `gather-context` before this assignment starts
- The `on-assignment-failure` event fires `recover-from-error` if the assignment fails

#### Prerequisites

- `init-existing-repository` completed (branch exists, labels imported, project created)
- GitHub Project available for linking the plan issue
- Labels available for application (`planning`, `documentation`)

#### Dependencies

- Depends on: `init-existing-repository` (for branch, labels, project)

#### Risks/Challenges

- **Template absence:** The assignment references `plan_docs/ai-new-app-template.md` but the actual directory uses different filenames. The agent must adapt and use the actual plan docs as input.
- **Issue template:** The assignment references `.github/ISSUE_TEMPLATE/application-plan.md` — must verify this template exists in the repo.
- **Plan scope:** The project is large (4 phases). The plan must be scoped to what's achievable and prioritize Phase 1 (Sentinel MVP).
- **Divergent models:** The reference code has divergent `WorkItem` models — the plan must address unification (I-1 from Plan Review).

#### Events Fired

- `pre-assignment-begin`: `gather-context`
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`
- `on-assignment-failure`: `recover-from-error`

---

### Assignment 3: `create-project-structure`

**Title:** Create Project Structure  
**Goal:** Create the actual project scaffolding — solution structure, configuration files, Docker setup, CI/CD foundation, documentation structure, and repository summary — based on the application plan.

#### Key Acceptance Criteria

- Solution/project structure created following the Python/uv tech stack
- All required directories and files established (`src/`, `src/models/`, `src/interfaces/`, `scripts/`, `local_ai_instruction_modules/`, `docs/`)
- `pyproject.toml` created with uv dependencies (FastAPI, Uvicorn, Pydantic, HTTPX, etc.)
- Version pinning files created (`.python-version`)
- Dockerfiles for each service component
- `docker-compose.yml` for local development (using Python stdlib for healthchecks, not curl)
- Configuration file templates (`.env.example`)
- Initial test project structure (`tests/`)
- CI/CD workflows in `.github/workflows/` with all actions pinned by SHA
- Documentation structure: README.md, `docs/`, ADRs
- Repository summary at `.ai-repository-summary.md` linked from README.md
- Solution builds successfully
- Docker configurations validated
- Initial commit with complete scaffolding pushed to branch

#### Project-Specific Notes

- **Python/uv stack:** Use `pyproject.toml` + `uv.lock` — NOT `global.json` (explicitly out of scope per spec)
- **No .NET artifacts:** This is a Python project; do not create `.sln`, `.csproj`, or `global.json` files
- **Docker healthchecks:** Use `python -c "import urllib.request; ..."` NOT `curl` (per assignment instructions)
- **Action SHA pinning:** All GitHub Actions must use full commit SHA, not version tags (per dynamic workflow directive)
- **Editable installs:** If using `uv pip install -e .`, ensure source directory is copied before install command in Dockerfile
- **Reference implementations:** The `notifier_service.py` and `orchestrator_sentinel.py` from `plan_docs/` should be adapted into `src/` with fixes for known issues (I-1 through I-10 from Plan Review)
- **Known issues to address during scaffolding:**
  - Unify WorkItem models into `src/models/work_item.py` (I-1)
  - Use `os.environ` for secrets, not hardcoded placeholders (I-5)
  - Create shared `httpx.AsyncClient` in constructors, not per-call (I-4)
  - Remove bare `except: pass` patterns (I-9)

#### Prerequisites

- `create-app-plan` completed (plan exists as issue, tech stack documented, architecture documented)
- Application plan with project structure defined
- Working branch with prior commits

#### Dependencies

- Depends on: `init-existing-repository` (branch, project infrastructure)
- Depends on: `create-app-plan` (plan with structure definition, tech stack)

#### Risks/Challenges

- **Scope creep:** The assignment creates scaffolding only — reference implementations from `plan_docs/` should be adapted, not copy-pasted with known bugs
- **Build validation:** Must verify the project actually builds/runs (`uv sync`, import checks)
- **CI/CD workflow correctness:** SHA pinning requires looking up current release SHAs for each action
- **Docker validation:** Dockerfiles and compose must be syntactically valid; healthcheck commands must work without curl

#### Events Fired

- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### Assignment 4: `create-agents-md-file`

**Title:** Create AGENTS.md File  
**Goal:** Create a comprehensive `AGENTS.md` at the repository root providing AI coding agents with precise context about the project — build steps, test commands, code conventions, project structure, and common pitfalls.

#### Key Acceptance Criteria

- `AGENTS.md` file exists at repository root
- Contains: project overview with purpose and tech stack
- Contains: setup/build/test commands verified to work
- Contains: code style and conventions section
- Contains: project structure / directory layout section
- Contains: testing instructions
- Contains: PR / commit guidelines
- Written in standard Markdown with agent-focused language
- All listed commands validated by running them
- File committed and pushed to working branch

#### Project-Specific Notes

- Must cross-reference with existing `AGENTS.md` (the template's version) and update for the actual project
- Must cross-reference with `.ai-repository-summary.md` (created in previous assignment)
- Must validate commands against the actual Python/uv project structure created in `create-project-structure`
- Key commands to document:
  - `uv sync` — install dependencies
  - `uv run python -m pytest` — run tests
  - `uv run uvicorn src.notifier_service:app --reload` — run notifier dev server
  - `uv run python -m src.orchestrator_sentinel` — run sentinel
  - `trunk check` — lint/format
- Architecture notes should reference the 4-pillar design (Ear/State/Brain/Hands)
- Common pitfalls should include: known issues from Plan Review, environment variable requirements, Docker network setup

#### Prerequisites

- `create-project-structure` completed (actual project files exist to validate commands against)
- Build and test tooling operational

#### Dependencies

- Depends on: `init-existing-repository` (branch)
- Depends on: `create-app-plan` (tech stack, architecture decisions)
- Depends on: `create-project-structure` (actual files, build system, test framework)

#### Risks/Challenges

- **Command validation:** All commands listed must actually work. If the project structure has issues, this assignment will expose them.
- **Stale template:** The existing `AGENTS.md` is template-focused; must be completely rewritten for the actual project.
- **Monorepo consideration:** This is a single-repo project (not a monorepo), so nested AGENTS.md files are not needed.

#### Events Fired

- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### Assignment 5: `debrief-and-document`

**Title:** Debrief and Document Learnings  
**Goal:** Produce a comprehensive debrief report capturing all work done, deviations from assignments, lessons learned, errors encountered, and recommendations for future phases.

#### Key Acceptance Criteria

- Detailed debrief report created following the 12-section structured template
- Report documented in `.md` file format
- All required sections complete: Executive Summary, Workflow Overview, Key Deliverables, Lessons Learned, What Worked Well, What Could Be Improved, Errors Encountered, Complex Steps, Suggested Changes, Metrics, Future Recommendations, Conclusion
- All deviations from assignments documented
- Report reviewed and approved
- Report committed and pushed to project repo
- Execution trace saved as `debrief-and-document/trace.md`

#### Project-Specific Notes

- The debrief must capture the unique aspects of this self-bootstrapping project
- **Plan Adjustment Mandate:** Flag any plan-impacting findings as ACTION ITEMS with recommendations (file new issue or update later phases)
- The execution trace should include: all `gh` CLI commands run, all `git` operations, all file modifications
- Suggested changes should address issues found in the reference implementations (I-1 through I-10) and recommendations (R-1 through R-9) from the Plan Review
- Metrics should include: total files created, lines of code, dependencies count, build time

#### Prerequisites

- All prior assignments completed (`init-existing-repository` through `create-agents-md-file`)
- Complete record of actions taken, commands run, and decisions made

#### Dependencies

- Depends on: all 4 prior assignments completed

#### Risks/Challenges

- **Incomplete trace:** If earlier assignments didn't capture detailed execution logs, the trace will be incomplete
- **Action item tracking:** Must clearly differentiate between "fixed during setup" and "needs future work" items

#### Events Fired

- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### Assignment 6: `pr-approval-and-merge`

**Title:** Pull Request Approval and Merge  
**Goal:** Complete the full PR approval and merge process for the setup PR created during `init-existing-repository`, including CI verification, code review, comment resolution, merge, branch cleanup, and issue closure.

#### Key Acceptance Criteria

**CI Verification:**
- All required CI/CD status checks pass before code review begins
- CI remediation loop executed (up to 3 attempts) if any check fails
- Escalation to orchestrator documented if CI cannot be fixed within 3 attempts

**Code Review:**
- Code review delegated to `code-reviewer` subagent (NOT self-review)
- Auto-reviewer comments (Copilot, CodeQL, etc.) waited for before comment resolution

**Review Comment Resolution:**
- `ai-pr-comment-protocol.md` workflow executed and logged
- `pr-review-comments` acceptance criteria satisfied
- GraphQL verification artifacts captured (`pr-unresolved-threads.json`)

**Approval & Merge:**
- Stakeholder/Delegating Agent approval obtained
- Merge performed using repository policies
- Merge result recorded (`result` output: `merged`, `pending`, or `failed`)

**Post-Merge:**
- Source branch deleted (if merge succeeded)
- Related setup issues closed or updated
- Run report updated with final status

#### Project-Specific Notes

- **Special handling:** Per the dynamic workflow, this is an automated setup PR — self-approval by the orchestrator is acceptable. No human stakeholder approval is required.
- **CI remediation loop (Phase 0.5):** MUST still be executed — if CI checks fail, attempt up to 3 fix cycles before escalating
- **PR number:** Passed from `init-existing-repository` output (`#initiate-new-repository.init-existing-repository`)
- **On successful merge:** Delete the setup branch and close any related setup issues
- **Protocol compliance:** Must read and acknowledge `ai-pr-comment-protocol.md` and `pr-review-comments.md` before proceeding

#### Prerequisites

- PR exists (created during `init-existing-repository`)
- All prior assignments completed and committed to the PR branch
- All local changes committed and pushed before merge

#### Dependencies

- Depends on: all 5 prior assignments completed
- Depends on: `$pr_num` from `init-existing-repository`

#### Risks/Challenges

- **CI failures:** The project is new; CI workflows may have issues with Python/uv setup, Docker builds, or linting configurations
- **Merge conflicts:** Unlikely since this is the first PR, but possible if `main` was updated during the workflow
- **Branch protection:** The ruleset imported in Assignment 1 may require specific checks to pass before merge is allowed

#### Events Fired

- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                   project-setup Dynamic Workflow                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─ PRE-SCRIPT-BEGIN ─────────────────────────────────────────────┐ │
│  │                                                                 │ │
│  │  [create-workflow-plan]  ← THIS ASSIGNMENT                     │ │
│  │    → Read dynamic workflow, all assignments, all plan_docs      │ │
│  │    → Produce this workflow execution plan                       │ │
│  │    → Commit as plan_docs/workflow-plan.md                       │ │
│  │    → Record: #events.pre-script-begin.create-workflow-plan     │ │
│  │                                                                 │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                              │                                       │
│                              ▼                                       │
│  ┌─ MAIN SCRIPT: initiate-new-repository ─────────────────────────┐ │
│  │                                                                 │ │
│  │  ┌──────────────────────────┐   ┌───────────────────────────┐  │ │
│  │  │ 1. init-existing-repo    │──▶│ post-assignment-complete   │  │ │
│  │  │    • Create branch       │   │  • validate-completion     │  │ │
│  │  │    • Import ruleset      │   │  • report-progress         │  │ │
│  │  │    • Create GH Project   │   └───────────────────────────┘  │ │
│  │  │    • Import labels       │                                   │ │
│  │  │    • Rename files        │   ┌───────────────────────────┐  │ │
│  │  │    • Create PR ($pr_num) │──▶│ post-assignment-complete   │  │ │
│  │  └──────────────────────────┘   │  • validate-completion     │  │ │
│  │                                  │  • report-progress         │  │ │
│  │  ┌──────────────────────────┐   └───────────────────────────┘  │ │
│  │  │ 2. create-app-plan       │                                   │ │
│  │  │    • Analyze plan_docs   │   ┌───────────────────────────┐  │ │
│  │  │    • Create tech-stack   │──▶│ post-assignment-complete   │  │ │
│  │  │    • Create architecture │   │  • validate-completion     │  │ │
│  │  │    • Create plan issue   │   │  • report-progress         │  │ │
│  │  │    • Create milestones   │   └───────────────────────────┘  │ │
│  │  └──────────────────────────┘                                   │ │
│  │                                  ┌───────────────────────────┐  │ │
│  │  ┌──────────────────────────┐   │ post-assignment-complete   │  │ │
│  │  │ 3. create-project-struct │──▶│  • validate-completion     │  │ │
│  │  │    • Create src/ layout  │   │  • report-progress         │  │ │
│  │  │    • pyproject.toml      │   └───────────────────────────┘  │ │
│  │  │    • Dockerfile(s)       │                                   │ │
│  │  │    • docker-compose.yml  │   ┌───────────────────────────┐  │ │
│  │  │    • CI/CD workflows     │──▶│ post-assignment-complete   │  │ │
│  │  │    • Tests structure     │   │  • validate-completion     │  │ │
│  │  │    • .ai-repo-summary    │   │  • report-progress         │  │ │
│  │  │    • Validate build      │   └───────────────────────────┘  │ │
│  │  └──────────────────────────┘                                   │ │
│  │                                  ┌───────────────────────────┐  │ │
│  │  ┌──────────────────────────┐   │ post-assignment-complete   │  │ │
│  │  │ 4. create-agents-md      │──▶│  • validate-completion     │  │ │
│  │  │    • Write AGENTS.md     │   │  • report-progress         │  │ │
│  │  │    • Validate commands   │   └───────────────────────────┘  │ │
│  │  │    • Cross-reference docs│                                   │ │
│  │  └──────────────────────────┘   ┌───────────────────────────┐  │ │
│  │                                  │ post-assignment-complete   │  │ │
│  │  ┌──────────────────────────┐   │  • validate-completion     │  │ │
│  │  │ 5. debrief-and-document  │──▶│  • report-progress         │  │ │
│  │  │    • 12-section report   │   └───────────────────────────┘  │ │
│  │  │    • Execution trace     │                                   │ │
│  │  │    • Action items        │   ┌───────────────────────────┐  │ │
│  │  └──────────────────────────┘   │ post-assignment-complete   │  │ │
│  │                                  │  • validate-completion     │  │ │
│  │  ┌──────────────────────────┐   │  • report-progress         │  │ │
│  │  │ 6. pr-approval-and-merge │──▶│                           │  │ │
│  │  │    • CI verification     │   └───────────────────────────┘  │ │
│  │  │    • Code review         │                                   │ │
│  │  │    • Resolve comments    │                                   │ │
│  │  │    • Merge PR            │                                   │ │
│  │  │    • Cleanup branch      │                                   │ │
│  │  └──────────────────────────┘                                   │ │
│  │                                                                  │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                              │                                       │
│                              ▼                                       │
│  ┌─ POST-SCRIPT-COMPLETE ─────────────────────────────────────────┐ │
│  │                                                                 │ │
│  │  Apply label `orchestration:plan-approved` to the plan issue   │ │
│  │  (created during create-app-plan)                               │ │
│  │    → Locate issue from #initiate-new-repository.create-app-plan │ │
│  │    → Apply label                                                │ │
│  │    → Record: #events.post-script-complete.plan-approved        │ │
│  │                                                                 │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 5. Event Assignments Summary

| Event | Assignment | Purpose |
|-------|------------|---------|
| **pre-script-begin** | create-workflow-plan | Create this execution plan |
| **pre-assignment-begin** (Assignment 2 only) | gather-context | Gather context before app plan creation |
| **post-assignment-complete** (after each) | validate-assignment-completion | Independent QA validation |
| **post-assignment-complete** (after each) | report-progress | Progress reporting and checkpointing |
| **on-assignment-failure** (Assignment 2 only) | recover-from-error | Error recovery |
| **post-script-complete** | (label application) | Apply `orchestration:plan-approved` to plan issue |

---

## 6. Open Questions

| # | Question | Impact | Resolution Needed Before |
|---|----------|--------|--------------------------|
| 1 | Does `.github/ISSUE_TEMPLATE/application-plan.md` exist in the template repo? The `create-app-plan` assignment references it as the plan issue template. If absent, the agent must create it or use an alternative format. | `create-app-plan` | Assignment 2 |
| 2 | Does `.github/protected-branches_ruleset.json` exist in the template repo? Required for ruleset import in `init-existing-repository`. | `init-existing-repository` | Assignment 1 |
| 3 | Is `GH_ORCHESTRATION_AGENT_TOKEN` (PAT with `administration: write`) available in the environment? Required for branch protection ruleset import. Without it, ruleset import will fail. | `init-existing-repository` | Assignment 1 |
| 4 | The assignment references `plan_docs/ai-new-app-template.md` but the actual files use different names (OS-APOW prefixed). The agent must adapt and use the actual plan docs as input. | `create-app-plan` | Assignment 2 |
| 5 | Should the reference implementations (`notifier_service.py`, `orchestrator_sentinel.py`) from `plan_docs/` be adapted into `src/` during `create-project-structure`, or should they remain as reference only? The Plan Review identified 10 issues that should be fixed if they're used as starting code. | `create-project-structure` | Assignment 3 |
| 6 | What is the target repository for the sentinel to poll? The Implementation Spec says it's configurable via `.env`, but a default `GITHUB_ORG`/`GITHUB_REPO` must be established. | `create-project-structure` | Assignment 3 |
| 7 | Will the existing `AGENTS.md` (template-focused) be overwritten or merged? The `create-agents-md-file` assignment should produce a project-specific version. | `create-agents-md-file` | Assignment 4 |
| 8 | The `pr-approval-and-merge` assignment requires the `ai-pr-comment-protocol.md` and `pr-review-comments.md` files. Are these available in the local instruction modules or do they need to be fetched from the remote agent-instructions repository? | `pr-approval-and-merge` | Assignment 6 |

---

## 7. Risk Register

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Permissions errors during ruleset import | Medium | High | Verify PAT has `administration: write` scope before starting |
| Build failures due to uv/pyproject.toml misconfiguration | Medium | Medium | Validate pyproject.toml syntax, test `uv sync` early |
| CI remediation exceeds 3 attempts | Low | High | Escalate to orchestrator with structured failure report |
| Plan documents contain conflicting information | Low | Medium | Cross-reference all docs, flag inconsistencies in plan issue |
| Missing environment variables for notifier | Medium | Medium | Crash at startup if WEBHOOK_SECRET not set (per R-6) |
| Long-running subagent delegation appears as hang | Medium | Medium | Implement heartbeat (R-1) — may need to defer to Phase 1 |

---

## 8. Success Criteria

The project-setup workflow is considered successful when:

1. ✅ Repository has complete project structure matching tech stack
2. ✅ Application plan documented in GitHub Issue with all phases
3. ✅ GitHub Project created with proper columns and linked issues
4. ✅ All labels imported and milestones created
5. ✅ AGENTS.md provides clear guidance for AI agents
6. ✅ Build and test commands validated and working
7. ✅ Debrief report captures all learnings and deviations
8. ✅ Setup PR merged, branch deleted, issues closed
9. ✅ `orchestration:plan-approved` label applied to plan issue
10. ✅ All actions in GitHub workflows pinned to commit SHA

---

*This workflow execution plan was produced by the `create-workflow-plan` assignment as part of the `pre-script-begin` event of the `project-setup` dynamic workflow.*  
**Plan Status:** Ready for Review
