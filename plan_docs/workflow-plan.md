# Workflow Execution Plan: project-setup

**Generated:** 2026-04-06  
**Workflow:** project-setup  
**Repository:** intel-agency/workflow-orchestration-queue-papa43

---

## 1. Overview

| Field | Value |
|-------|-------|
| **Workflow Name** | project-setup |
| **Workflow File** | `ai_instruction_modules/ai-workflow-assignments/dynamic-workflows/project-setup.md` |
| **Project Name** | OS-APOW (workflow-orchestration-queue) |
| **Total Assignments** | 6 main assignments |
| **Events** | 3 event types (pre-script-begin, post-assignment-complete, post-script-complete) |

### Purpose

The **project-setup** dynamic workflow initializes a new repository with the complete project infrastructure, including:
- Repository configuration (labels, project board, branch protection)
- Application planning based on seeded documents
- Project scaffolding and structure
- AI agent documentation (AGENTS.md)
- Debrief and lessons learned
- Final PR approval and merge

### High-Level Summary

This workflow transforms a seeded repository containing planning documents (`plan_docs/`) into a fully initialized project with:
- GitHub Project board for issue tracking
- Comprehensive application plan documented as a GitHub Issue
- Complete project structure matching the tech stack
- AGENTS.md for AI coding agent guidance
- All changes consolidated into a single setup PR

---

## 2. Project Context Summary

### Project Description

**OS-APOW (workflow-orchestration-queue)** is a headless agentic orchestration platform that transforms GitHub Issues into automated Execution Orders. It shifts AI from a passive co-pilot role to an autonomous background production service.

### Technology Stack

| Component | Technology |
|-----------|------------|
| **Language** | Python 3.12+ |
| **Web Framework** | FastAPI |
| **Package Manager** | uv (Rust-based, replaces pip/poetry) |
| **Validation** | Pydantic |
| **HTTP Client** | HTTPX (async) |
| **ASGI Server** | Uvicorn |
| **Containerization** | Docker, Docker Compose |
| **Dev Environment** | DevContainers |
| **CLI Scripts** | PowerShell Core (pwsh), Bash |

### Architecture (4-Pillar System)

1. **The Ear (Work Event Notifier)** - FastAPI webhook receiver for GitHub events
2. **The State (Work Queue)** - GitHub Issues as distributed state management ("Markdown as a Database")
3. **The Brain (Sentinel Orchestrator)** - Async Python background service for polling and dispatch
4. **The Hands (Opencode Worker)** - Isolated DevContainer executing AI-generated code

### Key Constraints

- **Security:** HMAC webhook verification, network isolation, credential scrubbing, ephemeral tokens
- **Resource Limits:** Worker containers capped at 2 CPUs / 4GB RAM
- **Polling-First:** Webhooks are optimization; polling provides resiliency
- **Shell-Bridge:** All worker interaction via `./scripts/devcontainer-opencode.sh`

### Known Risks (from Plan Review)

| Risk ID | Issue | Impact |
|---------|-------|--------|
| I-1 | Divergent WorkItem models between sentinel and notifier | Data inconsistency |
| I-2 | Race condition in task claiming (no assign-then-verify) | Duplicate work |
| I-3 | No jittered exponential backoff on poller | Rate limiting issues |
| I-6 | No heartbeat implementation for long-running tasks | Appears hung |
| I-7 | Cost guardrails story has no implementation | Budget overrun |
| I-10 | No environment reset between tasks | State contamination |

---

## 3. Assignment Execution Plan

### Assignment 1: init-existing-repository

| Field | Content |
|-------|---------|
| **Short ID** | `init-existing-repository` |
| **Goal** | Initialize the repository with branch, project board, labels, branch protection ruleset, and create the initial setup PR |

**Key Acceptance Criteria:**
- New branch created (prefix: `dynamic-workflow-project-setup`)
- Branch protection ruleset imported from `.github/protected-branches_ruleset.json`
- GitHub Project created with columns: Not Started, In Progress, In Review, Done
- Labels imported from `.github/.labels.json`
- Workspace and devcontainer files renamed to match repo name
- PR created from branch to `main`

**Project-Specific Notes:**
- Branch name will be: `dynamic-workflow-project-setup`
- Requires `administration: write` scope for ruleset import
- Use `GH_ORCHESTRATION_AGENT_TOKEN` (PAT) for ruleset API calls
- File renames: `ai-new-app-template.code-workspace` → `workflow-orchestration-queue-papa43.code-workspace`

**Prerequisites:**
- GitHub CLI authenticated with scopes: `repo`, `project`, `read:project`, `read:user`, `user:email`
- `administration: write` scope on target repository

**Dependencies:** None (first assignment)

**Risks / Challenges:**
- Permissions errors if PAT lacks `administration: write`
- Ruleset import may fail if name already exists (idempotency check required)
- PR creation requires at least one commit on branch

**Events:** None defined for this assignment

---

### Assignment 2: create-app-plan

| Field | Content |
|-------|---------|
| **Short ID** | `create-app-plan` |
| **Goal** | Create a comprehensive application plan based on the seeded planning documents in `plan_docs/` |

**Key Acceptance Criteria:**
- Application template analyzed and understood
- Plan documented in GitHub Issue using template from `.github/ISSUE_TEMPLATE/application-plan.md`
- `plan_docs/tech-stack.md` created with languages, frameworks, tools, packages
- `plan_docs/architecture.md` created with high-level design
- Milestones created based on phases
- Issue linked to GitHub Project and assigned to "Phase 1: Foundation" milestone
- Labels applied: `planning`, `documentation`

**Project-Specific Notes:**
- Primary spec: `plan_docs/OS-APOW Implementation Specification v1.md`
- Supporting docs: Architecture Guide v3, Development Plan v4, Plan Review
- Tech stack: Python 3.12+, FastAPI, uv, Pydantic, HTTPX, Docker
- Phases: 0 (Seeding), 1 (Sentinel MVP), 2 (Ear/Webhook), 3 (Deep Orchestration)
- **IMPORTANT:** Planning only - no code implementation in this assignment

**Prerequisites:**
- Branch created (from Assignment 1)
- `plan_docs/` directory exists with planning documents

**Dependencies:**
- Output from Assignment 1: branch, GitHub Project

**Risks / Challenges:**
- Plan documents are comprehensive - need to synthesize without losing critical details
- Known issues from Plan Review should be incorporated as risks in the plan
- Must NOT create actual project files - only planning documents

**Events:**
- `pre-assignment-begin`: gather-context
- `on-assignment-failure`: recover-from-error
- `post-assignment-complete`: report-progress

---

### Assignment 3: create-project-structure

| Field | Content |
|-------|---------|
| **Short ID** | `create-project-structure` |
| **Goal** | Create the actual project scaffolding including solution structure, Docker configuration, CI/CD foundation, and documentation |

**Key Acceptance Criteria:**
- Solution/project structure created following Python/FastAPI patterns
- `pyproject.toml` and `uv.lock` for dependency management
- `src/` directory with notifier_service.py, orchestrator_sentinel.py, models/, interfaces/
- Dockerfile and docker-compose.yml created
- `.github/workflows/` directory with basic CI workflow
- Documentation structure: README.md, docs/, ADRs
- `.ai-repository-summary.md` created
- All GitHub Actions pinned to commit SHA
- Build validated successfully

**Project-Specific Notes:**
- Use `uv` for all package management (not pip/poetry)
- Structure follows Implementation Spec §Project Structure:
  ```
  workflow-orchestration-queue/
  ├── pyproject.toml
  ├── uv.lock
  ├── src/
  │   ├── notifier_service.py
  │   ├── orchestrator_sentinel.py
  │   ├── models/
  │   └── interfaces/
  ├── scripts/
  │   ├── devcontainer-opencode.sh
  │   ├── gh-auth.ps1
  │   └── update-remote-indices.ps1
  └── local_ai_instruction_modules/
  ```
- Healthcheck in docker-compose must use Python stdlib (no curl)
- Editable installs require `COPY src/` before `uv pip install -e .`

**Prerequisites:**
- Application plan exists with defined tech stack
- Branch from Assignment 1

**Dependencies:**
- Output from Assignment 2: tech-stack.md, architecture.md, plan issue

**Risks / Challenges:**
- Build may fail if pyproject.toml is misconfigured
- Docker healthcheck syntax must avoid curl (use Python urllib)
- CI workflow actions must be SHA-pinned

**Events:** None defined for this assignment

---

### Assignment 4: create-agents-md-file

| Field | Content |
|-------|---------|
| **Short ID** | `create-agents-md-file` |
| **Goal** | Create an AGENTS.md file at repository root providing AI coding agents with context and instructions |

**Key Acceptance Criteria:**
- `AGENTS.md` exists at repository root
- Contains: Project Overview, Setup Commands, Project Structure, Code Style, Testing Instructions
- All listed commands have been validated by running them
- File uses standard Markdown with agent-focused language
- Complements (not duplicates) README.md and .ai-repository-summary.md

**Project-Specific Notes:**
- Key commands to document:
  - Install: `uv sync`
  - Build: `uv build` or `uv run python -m build`
  - Test: `uv run pytest`
  - Lint: `uv run ruff check .`
  - Run: `uv run uvicorn src.notifier_service:app --reload`
- Cross-reference with existing AGENTS.md in template repo
- Include monorepo guidance if applicable

**Prerequisites:**
- Project structure created (Assignment 3)
- Build/test tooling validated

**Dependencies:**
- Output from Assignment 3: project structure, validated build commands

**Risks / Challenges:**
- Commands must be validated - agents will attempt to execute them
- Must avoid duplicating content from README.md

**Events:** None defined for this assignment

---

### Assignment 5: debrief-and-document

| Field | Content |
|-------|---------|
| **Short ID** | `debrief-and-document` |
| **Goal** | Capture key learnings, insights, and areas for improvement from the workflow execution |

**Key Acceptance Criteria:**
- Detailed debrief report created following structured template (12 sections)
- Report includes: Executive Summary, Workflow Overview, Key Deliverables, Lessons Learned
- All deviations from assignments documented
- Execution trace saved as `debrief-and-document/trace.md`
- Report reviewed and approved by stakeholder
- Report committed and pushed

**Project-Specific Notes:**
- Document any plan-impacting findings as ACTION ITEMS
- Flag issues that require new GitHub issues to be filed
- Review whether Phase 1 implementation plan is still valid given learnings
- Known issues from Plan Review (I-1 through I-10) should be tracked if encountered

**Prerequisites:**
- All previous assignments complete
- Working branch has all commits

**Dependencies:**
- Outputs from all previous assignments

**Risks / Challenges:**
- Must capture all deviations, even minor ones
- Action items must be filed as GitHub issues

**Events:** None defined for this assignment

---

### Assignment 6: pr-approval-and-merge

| Field | Content |
|-------|---------|
| **Short ID** | `pr-approval-and-merge` |
| **Goal** | Complete the PR approval and merge process, including CI verification, comment resolution, and merge execution |

**Key Acceptance Criteria:**
- All CI/CD status checks pass before code review
- CI remediation loop executed (up to 3 attempts) if checks fail
- Code review delegated to code-reviewer subagent (NOT self-review)
- All review comments resolved via pr-review-comments assignment
- GraphQL verification artifacts captured
- Stakeholder approval obtained
- PR merged, source branch deleted
- Related issues closed

**Project-Specific Notes:**
- PR number from Assignment 1 (init-existing-repository)
- This is an automated setup PR - self-approval by orchestrator is acceptable
- Must still execute CI remediation loop (Phase 0.5)
- Follow `ai-pr-comment-protocol.md` for comment resolution
- Use `scripts/query.ps1` for managing PR review threads

**Prerequisites:**
- All assignments complete
- PR exists with all commits pushed

**Dependencies:**
- `$pr_num` from Assignment 1

**Risks / Challenges:**
- CI may fail requiring remediation (up to 3 attempts)
- Merge conflicts if main branch changed during setup
- Long-running CI may appear as hang

**Events:** None defined for this assignment

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PROJECT-SETUP WORKFLOW EXECUTION                         │
└─────────────────────────────────────────────────────────────────────────────┘

┌─ PRE-SCRIPT-BEGIN ─────────────────────────────────────────────────────────┐
│  create-workflow-plan (CURRENT)                                             │
│  └─ Output: plan_docs/workflow-plan.md                                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─ ASSIGNMENT 1 ─────────────────────────────────────────────────────────────┐
│  init-existing-repository                                                   │
│  ├─ Create branch: dynamic-workflow-project-setup                           │
│  ├─ Import branch protection ruleset                                        │
│  ├─ Create GitHub Project                                                   │
│  ├─ Import labels                                                           │
│  ├─ Rename workspace/devcontainer files                                     │
│  └─ Create PR → Output: $pr_num                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
┌─ POST-ASSIGNMENT ─────────────┐  ┌─────────────────────────────────────────┐
│  validate-assignment-complete │  │  report-progress                        │
│  └─ QA validation             │  │  └─ Progress update, checkpoint state   │
└───────────────────────────────┘  └─────────────────────────────────────────┘
                                    │
                                    ▼
┌─ ASSIGNMENT 2 ─────────────────────────────────────────────────────────────┐
│  create-app-plan                                                            │
│  ├─ [pre] gather-context                                                    │
│  ├─ Analyze plan_docs/                                                      │
│  ├─ Create tech-stack.md                                                    │
│  ├─ Create architecture.md                                                  │
│  ├─ Create plan issue → Output: plan_issue_number                           │
│  ├─ Create milestones                                                       │
│  └─ [post] report-progress                                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
       validate-assignment-complete      report-progress
                                    │
                                    ▼
┌─ ASSIGNMENT 3 ─────────────────────────────────────────────────────────────┐
│  create-project-structure                                                   │
│  ├─ Create pyproject.toml, uv.lock                                          │
│  ├─ Create src/ structure (notifier, sentinel, models, interfaces)          │
│  ├─ Create Dockerfile, docker-compose.yml                                   │
│  ├─ Create .github/workflows/                                               │
│  ├─ Create README.md, docs/                                                 │
│  ├─ Create .ai-repository-summary.md                                        │
│  └─ Validate build                                                          │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
       validate-assignment-complete      report-progress
                                    │
                                    ▼
┌─ ASSIGNMENT 4 ─────────────────────────────────────────────────────────────┐
│  create-agents-md-file                                                      │
│  ├─ Gather project context                                                  │
│  ├─ Validate build/test commands                                            │
│  ├─ Create AGENTS.md                                                        │
│  └─ Cross-reference with existing docs                                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
       validate-assignment-complete      report-progress
                                    │
                                    ▼
┌─ ASSIGNMENT 5 ─────────────────────────────────────────────────────────────┐
│  debrief-and-document                                                       │
│  ├─ Create debrief report (12 sections)                                     │
│  ├─ Document all deviations                                                 │
│  ├─ Save execution trace                                                    │
│  └─ File action items as GitHub issues                                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
       validate-assignment-complete      report-progress
                                    │
                                    ▼
┌─ ASSIGNMENT 6 ─────────────────────────────────────────────────────────────┐
│  pr-approval-and-merge                                                      │
│  ├─ Phase 0.5: CI Verification & Remediation (max 3 attempts)               │
│  ├─ Phase 0.75: Code Review Delegation                                      │
│  ├─ Phase 1: Resolve Review Comments (pr-review-comments)                   │
│  ├─ Phase 2: Secure Approval                                                │
│  └─ Phase 3: Merge & Wrap-Up → Output: result="merged"                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─ POST-SCRIPT-COMPLETE ─────────────────────────────────────────────────────┐
│  Apply label: orchestration:plan-approved                                   │
│  └─ To plan issue from Assignment 2                                         │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Event Assignments Summary

| Event | Assignment | Purpose |
|-------|------------|---------|
| **pre-script-begin** | create-workflow-plan | Create this execution plan |
| **post-assignment-complete** | validate-assignment-completion | Independent QA validation |
| **post-assignment-complete** | report-progress | Progress reporting and checkpointing |
| **post-script-complete** | (label application) | Apply `orchestration:plan-approved` to plan issue |

---

## 6. Open Questions

### Critical (Resolve Before Execution)

1. **Reference Implementation:** Should the existing scaffold code (`orchestrator_sentinel.py`, `notifier_service.py` mentioned in Plan Review) be used as a starting point, or should the project structure be created fresh?

2. **Plan Review Issues:** Should the known issues (I-1 through I-10) from the Plan Review be addressed during Phase 1 implementation, or deferred to later phases? Specifically:
   - I-2 (Race condition in task claiming)
   - I-6 (No heartbeat implementation)
   - I-7 (No cost guardrails)

3. **GitHub App Configuration:** Should webhook/GitHub App setup be included in project-setup, or is that out of scope for Phase 0/1?

### Medium Priority

4. **Target Repository:** Is the orchestration system being built in the current repository (`intel-agency/workflow-orchestration-queue-papa43`), or will it be deployed to a separate repository?

5. **Existing Scripts:** The plan references `scripts/devcontainer-opencode.sh`, `scripts/gh-auth.ps1`, etc. Are these already present in the template, or do they need to be created as part of project-setup?

6. **Test Fixtures:** Should test fixtures and sample webhook payloads be created during project-setup?

### Low Priority

7. **Documentation Depth:** How detailed should the initial ADRs (Architecture Decision Records) be?

8. **CI Workflow Scope:** Should the initial CI workflow include security scanning, or just build/test/lint?

---

## 7. Risk Register

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Permissions errors during ruleset import | Medium | High | Verify PAT has `administration: write` scope before starting |
| Build failures due to uv/pyproject.toml misconfiguration | Medium | Medium | Validate pyproject.toml syntax, test `uv sync` early |
| CI remediation exceeds 3 attempts | Low | High | Escalate to orchestrator with structured failure report |
| Plan documents contain conflicting information | Low | Medium | Cross-reference all docs, flag inconsistencies in plan issue |
| Missing environment variables for notifier | Medium | Medium | Crash at startup if WEBHOOK_SECRET not set (per R-6) |
| Long-running subagent delegation appears as hang | Medium | Medium | Implement heartbeat (R-1) - may need to defer to Phase 1 |

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

**Plan Status:** Ready for Review  
**Next Step:** Stakeholder approval, then commit to `plan_docs/workflow-plan.md`
