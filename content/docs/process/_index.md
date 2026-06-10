---
title: Process
weight: 80
---

# Process

This document describes how we work on documentation in the W3DS TRL7 project.

The process is intentionally **simple at the start** and will be **improved iteratively** based on real experience.

---

## Goal

- Produce complete and implementation-ready documentation by June 1
- Build an effective, scalable documentation process
- Identify and eliminate bottlenecks early

---

## Core Principles

- Documentation is the primary product at this stage  
- We follow an **Agile approach** (iterations over perfection)  
- We start simple and improve the process over time  
- GitHub is the single source of truth  
- All important decisions must be documented  
- ChatGPT is a mandatory tool for writers  

---

## Workflow

Each piece of work follows this pipeline:
Idea → Issue → Draft → PR → Review → Merge
### Step descriptions:

- **Idea** — raw thought, requirement, or problem  
- **Issue** — structured task in GitHub  
- **Draft** — initial version of documentation  
- **PR (Pull Request)** — proposed change to repository  
- **Review** — validation by experts and team  
- **Merge** — accepted and finalized  

---

## AI-assisted workflow

AI  is used at two mandatory stages:
Input → AI structuring → Draft → AI validation → PR
Writers must:
- structure raw input using AI  
- validate drafts using AI before PR  

AI does not make decisions — only assists.

---

## Agile Process

We work in **weekly sprints**.

Each task:
- is a small, meaningful piece of documentation  
- must result in a PR  

We prioritize:
- progress  
- fast feedback  
- continuous improvement  

---

## Sprint Structure

### Tuesday — Sprint Sync (Planning + Review)

**Purpose:**
- close previous sprint  
- align on next sprint  

**Agenda:**

**1. Previous Sprint Review**
- what was completed  
- demos / highlights (if any)  
- blockers and issues  
- what did not work  

**2. Process Feedback**
- suggestions for improvement  
- decisions on workflow changes  

**3. Next Sprint Planning**
- sprint goal  
- priority areas (vision / use-cases / requirements / etc.)  
- task selection (Issues)  
- ownership (who does what)  
- clarifications  

---

### Wednesday–Monday — Working Phase

**Activities:**
- creating / refining Issues  
- writing and updating documents  
- submitting PRs  
- reviewing and merging PRs  

**Principles:**
- async-first (GitHub + chat)  
- small, incremental PRs  
- visibility of progress in Issues  

---

### Mid-Sprint Check (Optional, Async)

When needed (no fixed day):
- short updates in chat:
  - progress  
  - blockers  
  - requests for help  

---

### Continuous Review (Async)

- PRs reviewed continuously (not only at end of sprint)  
- discussions happen in PRs / Issues  
- decisions documented in repo  

---

### Sprint Output

By next Tuesday:
- merged PRs  
- updated documentation sections  
- open Issues with clear status  

---

### Notes

- Single weekly call = decision point, not working session  
- Everything else happens asynchronously  
- Goal - reduce coordination overhead and keep momentum  
---

## Definition of Done

A task is complete when:

- documented in repository  
- structured and validated with AI  
- reviewed by team  
- merged via PR  

---

## Communication Rules

- Async-first (chat + GitHub). We use Slack. Join our workgroup: https://join.slack.com/t/w3dstrl7/shared_invite/zt-3u0xtw192-mFt1hRMWGQT2eaPGtWJODw 
- GitHub = source of truth  
- English is required for:
  - documentation  
  - issues  
  - PRs  
  - decisions  
- Calls are used only when async is inefficient  
- Every call must produce a written summary  

---

## Roles

- **Writers** — create documentation  
- **Experts / Clients** — validate correctness  
- **Developers** — validate feasibility  
- **Leader** — prioritization and final decisions  

---

## Artifacts

### GitHub Issues
Used for:
- requirements  
- ideas  
- tasks  

### Pull Requests
Used for:
- proposing changes  
- discussion and review  

### Documentation
Stored in `/docs`

### Process knowledge
Stored in `/process`

---

## Continuous Improvement

We actively improve the process.

After each sprint we capture:

- problems  
- root causes  
- improvements  

Stored in:
/process/lessons-learned.md

---

## Guidelines

- Keep texts concise and clear  
- Avoid ambiguity  
- Use verifiable statements  
- Prefer structure over free-form text  
- Do not aim for perfection in early iterations  

---

## Responsibility

Each participant is responsible for:
- contributing to documentation  
- following the process  
- improving the process through feedback  
