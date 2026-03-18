# Communication Protocol

This document explains how AI agents communicate with each other and with humans in the multi-agent development workflow.

## Core Principles

1. **Transparency**: All agents post status updates to the original issue so everyone can follow progress
2. **Traceability**: Every action, decision, and handoff is documented in the issue
3. **Human Visibility**: Humans can see the entire development story in one place (the original issue)
4. **Clear Handoffs**: Agents explicitly mention the next agent/human when passing work
5. **Kanban Sync**: The Scrum Master keeps the Kanban board updated at every stage transition

## Kanban Board Flow

```
Backlog → Planning → Developing → Testing → Human Review → Done
```

The Kanban board is a GitHub issue labeled `kanban`. The Scrum Master updates it at every stage transition.

## Communication Flow

### 0. Creating Backlog Items

**Trigger**: Human comments `@scrum-master <description or request>` on the Kanban issue

**Agent**: Scrum Master

**Actions**:
- Analyzes the human's request (project description, feature list, or high-level goal)
- Breaks it down into discrete, well-scoped backlog tickets with priorities
- Updates the Kanban issue body (adds items under Backlog section)
- Posts a summary comment listing what was added
- Human reviews and comments `approve <ticket-title>` to move items to Planning

### 1. Backlog → Planning

**Trigger**: Human comments `approve <ticket>` on the Kanban issue

**Agent**: Scrum Master

**Actions**:
- Creates a feature issue with `feature-request` label
- Updates Kanban board (Backlog → Planning)
- Posts status comment on Kanban issue

### 2. Planning (Feature Request → Implementation Plan)

**Trigger**: `feature-request` label on new issue

**Agent**: Planner (Product Manager)

**Communication Pattern**:
```markdown
## Planner - Starting Analysis
[Posted at start of work]

## Implementation Plan
[Posted after analysis complete]

## Planner - Plan Complete
**Next Steps**: @human Please review and approve
[Posted at end, adds `awaiting-human-review` label]
```

**Human Action Required**: Review plan, then add `approved-plan` label and comment `@fullstack-dev please implement`

### 3. Planning → Developing

**Trigger**: `approved-plan` label added

**Agent**: Scrum Master updates Kanban, Fullstack Dev starts implementation

**Communication Pattern**:
```markdown
## Fullstack Developer - Starting Implementation
**What I'll do**: [summary]
[Posted to original issue at start of work]

## Fullstack Developer - Pull Request Created
**PR**: #[NUMBER]
**What was implemented**: [list]
**Next Steps**: @qa-tester - Ready for review
[Posted to original issue after PR created]
```

### 4. Developing → Testing

**Trigger**: PR opened targeting `develop`

**Agent**: Scrum Master updates Kanban, QA Tester reviews

**Communication Pattern**:
```markdown
## QA Tester - Starting Review
**What I'll test**: [checklist]
[Posted to ORIGINAL ISSUE]

# If bugs found:
## QA Tester - Issues Found
**Summary**: [counts by severity]
**Next Steps**: @fullstack-dev - Please fix

# If tests pass:
## QA Tester - All Tests Passed
**Next Steps**: @human - Ready for merge
```

**Labels Added**:
- `bugs-found` → triggers bug fix workflow
- `tests-passed` → triggers human review

### 5. Bug Fix Loop

**Trigger**: `bugs-found` label on PR

**Agent**: Fullstack Developer

**Communication Pattern**:
```markdown
## Fullstack Developer - Bugs Fixed
**Fixes applied**: [list]
**Next Steps**: @qa-tester - Please re-review PR #[NUMBER]
[Posted to ORIGINAL ISSUE]
```

### 6. Testing → Human Review

**Trigger**: `tests-passed` label added

**Agent**: Scrum Master updates Kanban

### 7. Human Review → Done

**Trigger**: PR merged to `develop`

**Agent**: Scrum Master updates Kanban, marks ticket as done

---

## Where Agents Post

| Agent | Start of Work | During Work | End of Work | Results |
|-------|---------------|-------------|-------------|---------|
| **Scrum Master** | Kanban Issue | Kanban Issue | Kanban Issue | Kanban Issue |
| **Planner** | Feature Issue | Feature Issue | Feature Issue | Feature Issue |
| **Fullstack Dev** | Feature Issue | Feature Issue | Feature Issue | PR + Feature Issue |
| **QA Tester** | **Feature Issue** | Feature Issue | **Both PR and Issue** | **Both PR and Issue** |

### Key Rule for QA Tester

**CRITICAL**: The QA Tester MUST post to the original feature issue, not just the PR, because:
1. The issue is where the human is watching
2. The issue shows the complete development story
3. All other agents post to the issue
4. The human needs to know when testing is complete

---

## Label-Driven Workflow

| Label | Triggers | Added By | Removed By |
|-------|----------|----------|------------|
| `kanban` | Marks the Kanban board issue | Human | - |
| `feature-request` | Planner analysis | Scrum Master | - |
| `awaiting-human-review` | Human notification | Planner | Human |
| `needs-revision` | Planner revision | Human | Planner |
| `approved-plan` | Developer implementation | Human | - |
| `bugs-found` | Developer bug fix | QA Tester | Developer |
| `tests-passed` | Human merge notification | QA Tester | - |
| `ready-for-merge` | Final notification | Automation | - |
| `needs-clarification` | Human notification | Any agent | Human |

---

## Example Complete Flow

**Kanban Issue #1: Project Board**

1. **Human**: Adds "User authentication" to backlog, comments `approve User authentication`

2. **Scrum Master** → Kanban #1:
   ```
   Moving to Planning
   Created feature issue #2
   ```
   Board updated: Backlog → Planning

3. **Planner** → Issue #2:
   ```
   Starting Analysis
   Implementation Plan [detailed]
   Plan Complete → @human please review
   ```
   Adds: `awaiting-human-review`

4. **Human**: Reviews plan, adds `approved-plan`, comments `@fullstack-dev please implement`

5. **Scrum Master** → Kanban #1:
   ```
   Moving to Developing
   ```
   Board updated: Planning → Developing

6. **Fullstack Dev** → Issue #2:
   ```
   Starting Implementation
   [implements code]
   Pull Request Created - PR #3
   @qa-tester ready for review
   ```

7. **Scrum Master** → Kanban #1:
   ```
   Moving to Testing
   ```
   Board updated: Developing → Testing (PR #3)

8. **QA Tester** → Issue #2 + PR #3:
   ```
   Starting Review
   All Tests Passed
   @human ready for merge
   ```
   Adds `tests-passed` to PR #3

9. **Scrum Master** → Kanban #1:
   ```
   Moving to Human Review
   ```
   Board updated: Testing → Human Review

10. **Human**: Reviews and merges PR #3 to develop

11. **Scrum Master** → Kanban #1:
    ```
    Complete!
    ```
    Board updated: Human Review → Done

---

## For Humans: How to Monitor Progress

1. **Watch the Kanban Issue**: Single source of truth for all work status
2. **Watch Feature Issues**: Detailed development story for each feature
3. **Check Labels**: Labels show current workflow state
4. **Look for @ Mentions**: Agents will mention you when input is needed
5. **Review PR Links**: Agents post PR links in issues

---

## For Agents: Quick Reference

**Always**:
- Post to the original issue (find issue number from PR or context)
- Use the templates from your agent configuration
- Mention the next person/agent in your handoff
- Add/remove labels as specified in your configuration
- The Scrum Master updates the Kanban board at every stage transition

**Never**:
- Post only to the PR (QA Tester must post to both)
- Start work without posting a "Starting" message
- Complete work without posting a handoff message
- Forget to add the appropriate labels
