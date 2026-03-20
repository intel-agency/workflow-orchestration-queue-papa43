# Workflow Execution Plan: Project Setup

**Generated:** 2026-03-20
**Workflow:** project-setup
**Repository:** workflow-orchestration-queue-papa43

---

## 1. Overview

- **Workflow Name:** project-setup
- **Workflow File:** `ai_instruction_modules/ai-workflow-assignments/dynamic-workflows/project-setup.md`
- **Project Name:** workflow-orchestration-queue (OS-APOW)
- **Brief Description:** Headless agentic orchestration platform that transforms AI from passive co-pilot to autonomous background production service, managing task discovery and execution via GitHub Issues and labels.
- **Total Assignments:** 6 main assignments + 2 event handlers
- **High-level Summary:** Initialize the freshly cloned repository, create application plan from seeded documents, set up project scaffolding for a Python FastAPI-based orchestration system, document the repository, and create final debrief documentation.

---

## 2. Project Context Summary

### Key Facts from plan_docs/

| Aspect | Details |
|--------|---------|
| **Project Name** | workflow-orchestration-queue (OS-APOW) |
| **Purpose** | Headless agentic orchestration platform for autonomous AI-driven development |
| **Primary Language** | Python 3.12+ |
| **Frameworks** | FastAPI, Pydantic, HTTPX, Uvicorn |
| **Package Manager** | uv (Rust-based, fast dependency management) |
| **Containerization** | Docker, DevContainers |
| **Target Platform** | Linux (devcontainer), GitHub Actions |

### Key Components (Architecture)

1. **Work Event Notifier ("The Ear")** - FastAPI webhook receiver for GitHub events
2. **Work Queue ("The State")** - GitHub Issues and Labels as distributed state
3. **Sentinel Orchestrator ("The Brain")** - Persistent polling service
4. **Opencode Worker ("The Hands")** - DevContainer-based AI execution

### Repository Details

- **Template Source:** intel-agency/workflow-orchestration-queue-papa43
- **Current State:** Fresh clone with plan_docs/ already seeded
- **Devcontainer:** Pre-built image ready

### Special Constraints

- Self-bootstrapping system designed to build itself
- Uses existing devcontainer-opencode.sh shell bridge
- "Markdown as a Database" approach for state visibility
- Polling-first resiliency model

### Known Risks

- GitHub API rate limiting (mitigated by GitHub App tokens)
- LLM "looping" / hallucination (mitigated by max_steps timeout)
- Concurrency collisions (mitigated by GitHub Assignees as locks)

---

## 3. Assignment Execution Plan

### Assignment 1: init-existing-repository

| Field | Content |
|-------|---------|
| **Assignment** | `init-existing-repository`: Initialize Existing Repository |
| **Goal** | Set up the freshly cloned repository with GitHub Project, labels, and proper file naming |
| **Key Acceptance Criteria** | GitHub Project created and linked, Labels imported, Workspace/devcontainer files renamed, Branch created |
| **Project-Specific Notes** | This is a Python project, not .NET - adjust accordingly. Template repo already has .labels.json. |
| **Prerequisites** | GitHub authentication with repo, project scopes |
| **Dependencies** | None (first assignment) |
| **Risks / Challenges** | May need to handle existing files vs new creation |
| **Events** | post-assignment-complete → validate-assignment-completion, report-progress |

### Assignment 2: create-app-plan

| Field | Content |
|-------|---------|
| **Assignment** | `create-app-plan`: Create Application Plan |
| **Goal** | Create comprehensive application plan from plan_docs/ |
| **Key Acceptance Criteria** | Application template analyzed, Plan documented in issue, Milestones created, Labels applied, `implementation:ready` label added |
| **Project-Specific Notes** | Plan docs already contain OS-APOW Development Plan v4, Architecture Guide v3, Implementation Spec v1. Need to create tech-stack.md and architecture.md in plan_docs/. |
| **Prerequisites** | Repository initialized |
| **Dependencies** | Outputs from init-existing-repository (Project, labels) |
| **Risks / Challenges** | No ai-new-app-template.md file exists - use Implementation Specification as primary source |
| **Events** | pre-assignment-begin → gather-context, post-assignment-complete → report-progress |

### Assignment 3: create-project-structure

| Field | Content |
|-------|---------|
| **Assignment** | `create-project-structure`: Create Project Structure |
| **Goal** | Create actual project scaffolding based on application plan |
| **Key Acceptance Criteria** | Solution structure created, Project files established, CI/CD foundation, Documentation structure, Initial commit made |
| **Project-Specific Notes** | Python project using pyproject.toml (not .NET). Structure should include src/, tests/, scripts/, docs/ directories per Implementation Spec. |
| **Prerequisites** | Application plan documented |
| **Dependencies** | Outputs from create-app-plan |
| **Risks / Challenges** | Adapt .NET-focused assignment to Python ecosystem |
| **Events** | None specified |

### Assignment 4: create-repository-summary

| Field | Content |
|-------|---------|
| **Assignment** | `create-repository-summary`: Create Repository Summary |
| **Goal** | Create .ai-repository-summary.md file for AI agent onboarding |
| **Key Acceptance Criteria** | Summary file exists at root, Build/test commands documented, Project layout described |
| **Project-Specific Notes** | Focus on Python/FastAPI commands (uv, pytest, etc.), document devcontainer usage |
| **Prerequisites** | Project structure created |
| **Dependencies** | Outputs from create-project-structure |
| **Risks / Challenges** | None significant |
| **Events** | None specified |

### Assignment 5: create-agents-md-file

| Field | Content |
|-------|---------|
| **Assignment** | `create-agents-md-file`: Create AGENTS.md File |
| **Goal** | Create/update AGENTS.md with project-specific agent instructions |
| **Key Acceptance Criteria** | AGENTS.md exists at root, Contains verified commands, Follows agents.md specification |
| **Project-Specific Notes** | Template already has AGENTS.md - update with project-specific context |
| **Prerequisites** | Project structure and repository summary |
| **Dependencies** | Outputs from create-repository-summary |
| **Risks / Challenges** | Ensure consistency with existing AGENTS.md content |
| **Events** | None specified |

### Assignment 6: debrief-and-document

| Field | Content |
|-------|---------|
| **Assignment** | `debrief-and-document`: Debrief and Document Learnings |
| **Goal** | Create comprehensive debrief report with lessons learned |
| **Key Acceptance Criteria** | Detailed report created, All deviations documented, Execution trace saved, Report committed |
| **Project-Specific Notes** | Capture all deviations from assignments, especially Python vs .NET adaptations |
| **Prerequisites** | All previous assignments complete |
| **Dependencies** | All prior assignment outputs |
| **Risks / Challenges** | Thorough documentation required |
| **Events** | None specified |

---

## 4. Sequencing Diagram

```
pre-script-begin: create-workflow-plan
    │
    ▼
┌─────────────────────────────────┐
│ init-existing-repository        │
│   ↓ post-assignment-complete    │
│   validate-assignment-completion│
│   report-progress               │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│ create-app-plan                 │
│   ↓ post-assignment-complete    │
│   validate-assignment-completion│
│   report-progress               │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│ create-project-structure        │
│   ↓ post-assignment-complete    │
│   validate-assignment-completion│
│   report-progress               │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│ create-repository-summary       │
│   ↓ post-assignment-complete    │
│   validate-assignment-completion│
│   report-progress               │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│ create-agents-md-file           │
│   ↓ post-assignment-complete    │
│   validate-assignment-completion│
│   report-progress               │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│ debrief-and-document            │
│   ↓ post-assignment-complete    │
│   validate-assignment-completion│
│   report-progress               │
└─────────────────────────────────┘
    │
    ▼
post-script-complete (if defined)
```

---

## 5. Open Questions

1. **Template Adaptation**: The `create-project-structure` assignment references .NET-specific tooling. Should we follow Python conventions (pyproject.toml, src/ layout) instead?

2. **Primary App Spec**: No `ai-new-app-template.md` file exists in plan_docs/. Should we use `OS-APOW Implementation Specification v1.md` as the primary app specification?

3. **GitHub Project**: Should the project be created in the intel-agency organization or the user's personal account?

---

## 6. Approval

- [ ] Workflow plan reviewed
- [ ] Open questions resolved
- [ ] Plan approved for execution

**Approved by:** _[pending]_
**Date:** _[pending]_
