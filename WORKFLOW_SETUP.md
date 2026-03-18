# Multi-Agent GitHub Actions Workflow Setup

This document explains the automated multi-agent development workflow for this project.

## Overview

This project uses AI agents with specialized roles to automate the software development lifecycle through GitHub Actions. A Scrum Master agent manages a Kanban board while other agents handle planning, development, and testing.

## Architecture

### Agents

Four specialized AI agents collaborate on development:

1. **Scrum Master** (`.claude/agents/scrum_master.md`)
   - Manages the Kanban board (a GitHub issue)
   - Creates tickets and feature issues
   - Assigns work to agents
   - Tracks status through all stages

2. **Planner / Product Manager** (`.claude/agents/product_manager.md`)
   - Analyzes feature requests
   - Creates implementation plans
   - Defines acceptance criteria

3. **Fullstack Developer** (`.claude/agents/fullstack_dev.md`)
   - Implements features end-to-end (reads CLAUDE.md for tech stack)
   - Writes tests using the project's testing framework
   - Creates pull requests

4. **QA Tester** (`.claude/agents/tester.md`)
   - Reviews code quality
   - Runs lint, tests, and build
   - Reports bugs or approves PRs

### Workflows

Nine GitHub Actions workflows orchestrate the agents:

| # | Workflow | Purpose |
|---|---------|---------|
| 00 | `00-kanban-management.yml` | Scrum Master manages Kanban board transitions |
| 01 | `01-issue-to-plan.yml` | Planner analyzes feature requests |
| 02 | `02-plan-to-implementation.yml` | Fullstack Dev implements features |
| 03 | `03-pr-review.yml` | QA Tester reviews PRs |
| 04 | `04-fix-bugs.yml` | Fullstack Dev fixes reported bugs |
| 05 | `05-final-approval.yml` | Marks PRs ready for human merge |
| 06 | `06-clarification-request.yml` | Notifies human when agents need input |
| 07 | `07-human-review-checkpoint.yml` | Notifies human to review plan |
| 08 | `08-plan-revision.yml` | Planner revises plan based on feedback |

## Development Flow

```
Human adds item to Kanban backlog
         │
         ▼ (human comments "approve <ticket>")
Scrum Master creates feature issue (#feature-request)
         │
         ▼
Planner creates implementation plan
         │
         ▼ (human adds "approved-plan" + "@fullstack-dev please implement")
Scrum Master moves to Developing
         │
         ▼
Fullstack Dev implements + creates PR
         │
         ▼
Scrum Master moves to Testing
         │
         ▼
QA Tester reviews PR
    ┌────┴────┐
    │         │
 bugs-found  tests-passed
    │         │
    ▼         ▼
 Dev fixes  Scrum Master moves to Human Review
    │         │
    └──→ QA   ▼
         re-  Human merges PR
         reviews    │
                    ▼
              Scrum Master moves to Done
```

**Branch Strategy**: `main` (prod) ← `develop` (dev) ← `feature/*` branches

## Setup Instructions

### Prerequisites

1. **GitHub Repository** with Actions enabled
2. **Claude Code OAuth Token** or **Anthropic API Key**

### Installation Steps

#### 1. Add Claude OAuth Token to Repository Secrets

1. Go to your GitHub repository Settings
2. Click **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `CLAUDE_CODE_OAUTH_TOKEN`
5. Value: Your Claude Code OAuth token
6. Click **Add secret**

#### 2. Configure Repository Permissions

Go to: `Settings` → `Actions` → `General` → `Workflow permissions`
Select: **Read and write permissions**

#### 3. Create Required Labels

Create these labels in your repository (Settings → Labels):

| Label | Color | Description |
|-------|-------|-------------|
| `kanban` | `#0075ca` | Kanban board issue |
| `feature-request` | `#a2eeef` | Feature request for planning |
| `awaiting-human-review` | `#fbca04` | Waiting for human review |
| `needs-revision` | `#e4e669` | Plan needs revision |
| `approved-plan` | `#0e8a16` | Plan approved for implementation |
| `bugs-found` | `#d73a4a` | Bugs found in PR |
| `tests-passed` | `#0e8a16` | All tests passed |
| `ready-for-merge` | `#0e8a16` | Ready for human merge |
| `needs-clarification` | `#fbca04` | Agent needs human input |
| `priority-high` | `#b60205` | High priority |
| `priority-medium` | `#fbca04` | Medium priority |
| `priority-low` | `#0e8a16` | Low priority |

#### 4. Create the Kanban Board Issue

Create a new issue with:
- Title: "Kanban Board"
- Label: `kanban`
- Body:
```markdown
## Kanban Board

### Backlog

### Planning

### Developing

### Testing

### Human Review

### Done
```

#### 5. Push Workflow Files

Commit and push all files to the repository:

```bash
git add .github/ .claude/ CLAUDE.md COMMUNICATION_PROTOCOL.md WORKFLOW_SETUP.md README.md .gitignore
git commit -m "feat: add multi-agent GitHub Actions workflow with Kanban board"
git push origin main
```

## Usage Guide

### Creating Work Items

1. **Comment `@scrum-master <your request>`** on the Kanban issue to have the Scrum Master create backlog items
   - Example: `@scrum-master Create backlogs for a todo app with user auth, CRUD operations, and search`
   - The Scrum Master will break it down into discrete tickets, assign priorities, and update the board
2. **Comment `approve <ticket-title>`** to move a backlog item to Planning
3. The Scrum Master will create a feature issue and the Planner will start automatically

### Reviewing Plans

1. When notified, review the Planner's implementation plan in the feature issue
2. If approved: Add `approved-plan` label and comment `@fullstack-dev please implement`
3. If changes needed: Add `needs-revision` label and post feedback

### Monitoring Progress

- **Kanban Board**: Check the Kanban issue for overall project status
- **Feature Issues**: Check individual issues for detailed progress
- **PRs**: Check pull requests for code changes and test results
- **Actions Tab**: Check workflow runs for execution logs

### Merging

When a PR has the `ready-for-merge` label:
1. Review the implementation
2. Review the QA report
3. Merge the PR to `develop` (NOT `main`)
4. Only merge `develop` → `main` for production releases

## Labels Reference

| Label | Purpose | Applied By | Triggers |
|-------|---------|------------|----------|
| `kanban` | Marks Kanban board issue | Human | Workflow 00 |
| `feature-request` | Triggers Planner | Scrum Master | Workflow 01 |
| `awaiting-human-review` | Plan needs approval | Planner | Workflow 07 |
| `needs-revision` | Plan needs changes | Human | Workflow 08 |
| `approved-plan` | Start development | Human | Workflow 00 + 02 |
| `bugs-found` | Code has issues | Tester | Workflow 04 |
| `tests-passed` | Tests passed | Tester | Workflow 00 + 05 |
| `ready-for-merge` | Ready for merge | Automation | Human review |
| `needs-clarification` | Agent needs input | Any agent | Workflow 06 |

## Cost Considerations

- Each workflow run uses Claude API tokens
- Scrum Master uses Opus (more capable, higher cost) for board management
- Planner uses Opus for analysis
- Developer and Tester use Sonnet (cost-effective for implementation and testing)
- Adjust `--max-turns` in workflows to control token usage

## Troubleshooting

**Workflow doesn't trigger**: Check labels are spelled correctly, workflow files are on the default branch

**Authentication error**: Verify `CLAUDE_CODE_OAUTH_TOKEN` is set in repository secrets

**Agent doesn't follow instructions**: Check agent config files are readable, CLAUDE.md is up to date, increase `--max-turns`

**Kanban board not updating**: Check the Scrum Master workflow logs, verify the kanban issue has the `kanban` label
