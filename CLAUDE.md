# Project Context

<!-- ============================================================
     CUSTOMIZE THIS FILE FOR YOUR PROJECT
     This is the single source of truth that all AI agents read
     before doing any work. Update it to match your actual project.
     ============================================================ -->

## Project Overview
<!-- Describe what your project does and its purpose -->
A sample web application designed to demonstrate automated multi-agent development using GitHub Actions and Claude AI agents.

## Tech Stack
<!-- List your actual tech stack. Agents use this to write appropriate code. -->
- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript (strict mode)
- **UI**: React 19, Tailwind CSS
- **Backend**: Supabase (Auth, Database, Storage)
- **Database**: PostgreSQL (via Supabase)
- **Testing**: Jest, React Testing Library
- **Linting**: ESLint

## Build & Test Commands
<!-- IMPORTANT: Agents use these exact commands to build, test, and lint your project.
     Update them to match your project's package manager and scripts. -->

```bash
# Install dependencies
npm install

# Linting - must pass with no errors
npm run lint

# Run all tests - all must pass
npm run test

# Build verification - must succeed
npm run build

# Start development server
npm run dev
```

## Project Structure
<!-- Describe your project's directory layout so agents know where to put files -->
```
app/                    # Pages and routes
components/             # Reusable UI components
lib/                    # Utilities, clients, helpers
types/                  # TypeScript type definitions
__tests__/              # Test files
public/                 # Static assets
```

## Development Guidelines

### Coding Standards
<!-- Add your project-specific coding conventions here -->
- Follow the language's standard style guide
- Use meaningful variable and function names
- Write self-documenting code with comments for complex logic
- Handle errors gracefully with proper error messages
- Write type-safe code (avoid `any` types in TypeScript)

### Git Workflow

**Branch Strategy**:
- `main` - Production environment (stable, deployed code)
- `develop` - Development environment (integration branch)
- `feature/<feature-name>` - Feature branches (created from `develop`)
- `fix/<bug-name>` - Bug fix branches (created from `develop`)

**Merge Strategy**:
- Feature branches merge to `develop` (NOT to `main`)
- Pull requests should target the `develop` branch as the base
- Only humans merge from `develop` to `main` for production releases

**Commit Messages**: Use conventional commits format
- `feat:` for new features
- `fix:` for bug fixes
- `refactor:` for code refactoring
- `test:` for test additions/changes
- `docs:` for documentation updates
- `chore:` for build/tooling changes

### Testing Requirements
<!-- Adjust testing standards to match your project -->
- Every new feature MUST have tests
- Every bug fix SHOULD have a regression test
- All tests must pass before PR approval
- Test both success and error cases
- Test edge cases (empty states, null values, boundary conditions)

## Multi-Agent Development Process

This project uses AI agents for development automation:

- **Scrum Master**: Manages the Kanban board, creates tickets, assigns work, tracks status
- **Planner (Product Manager)**: Analyzes feature requests and creates implementation plans
- **Fullstack Developer**: Implements features end-to-end
- **QA Tester**: Reviews code quality and runs test suites

### Kanban Board Workflow
1. Work items are tracked on a Kanban board (GitHub issue labeled `kanban`)
2. Stages: Backlog → Planning → Developing → Testing → Human Review → Done
3. Scrum Master coordinates movement between stages
4. All status updates posted to relevant issues for transparency

### Agent Communication Protocol
1. Feature requests start on the Kanban board as backlog items
2. Human approves backlog items to move to Planning
3. Scrum Master creates feature issues, triggering the Planner
4. Planner creates detailed plans, awaits human approval
5. Human approves plan, Fullstack Dev implements
6. PRs are automatically reviewed by QA Tester
7. Failed tests trigger bug fix workflow
8. Passing tests trigger human review for merge

## Notes for AI Agents

### Critical Git Workflow Rules
- **ALWAYS** create feature branches from `develop` branch (NOT from `main`)
- **ALWAYS** name feature branches with `feature/` prefix (e.g., `feature/user-authentication`)
- **ALWAYS** target `develop` as the base branch when creating pull requests
- **NEVER** create PRs that merge directly to `main` - only humans do this
- The `main` branch is production - it must remain stable
- The `develop` branch is development - all features integrate here

### How to Find Build/Test Commands
- **ALWAYS** read the "Build & Test Commands" section above before running any commands
- **ALWAYS** check `package.json`, `Makefile`, `Cargo.toml`, `pyproject.toml`, or equivalent for available scripts
- **DO NOT** assume commands — verify they exist first

### Example Git Commands for Agents
```bash
# Correct way to create a feature branch
git checkout develop
git pull origin develop
git checkout -b feature/my-feature

# When opening PR - base branch: develop
gh pr create --base develop --title "feat: description" --body "..."
```

### General Guidelines
- Always read this file before starting work
- Follow the established patterns in the codebase
- Ask clarifying questions if requirements are unclear
- Prioritize code quality, security, and maintainability
