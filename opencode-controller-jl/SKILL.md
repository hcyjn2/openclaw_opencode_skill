---
name: opencode-controller
description: Orchestrate OpenCode (oh-my-opencode) for Flutter, Web, and LLM Application development. Two-step workflow — plan first, execute after user approval.
metadata: { "openclaw": { "emoji": "🏛️", "requires": { "bins": ["opencode"] }, "os": ["darwin"] } }
---

# OpenCode Controller

## Core Rule

**OpenClaw does NOT write code. All planning and coding happens inside OpenCode.**

## Session Management

1. Start OpenCode in the project directory
2. Open session selector: `/sessions`
3. If the current project has an existing session: **select it** (resume context)
4. Never create a new session without user approval

## Workflow

```
1. Start OpenCode
2. Reuse existing project session (or create if new project)
3. Create feature branch
4. Select Plan agent
5. Generate and validate plan
6. Report plan to user → WAIT for approval
7. If approved: switch to Build agent
8. Implement the approved plan
9. Run verification suite (test, lint, build)
10. Report results
11. If pass: commit and report completion
12. If fail: return to step 4 (Plan agent)
```

## Agent Control

Switch agents using `/agents`. Available agents:

| Agent | When to Use |
|-------|-------------|
| **Plan** | Always start here. Analyze task, create step-by-step plan. No code generation. |
| **Build** | Only after plan is approved. Implements the plan. |

### Plan Agent Rules

- Ask OpenCode to analyze the task and produce a clear step-by-step plan
- Allow OpenCode to ask clarification questions
- Review the plan carefully
- If the plan is incorrect or incomplete: ask OpenCode to revise
- **Do NOT allow code generation in Plan mode**

Send structured requirements to OpenCode:
```
Plan the following task (do NOT start implementation yet, only create the plan):

## Objective
[Clear description]

## Context
- Project type: [Flutter / Web / LLM Application]
- Affected areas: [files, components, modules]

## Requirements
1. [Specific requirement]

## Acceptance Criteria
- [What success looks like]
```

### Build Agent Rules

- Switch to Build using `/agents`
- Ask OpenCode to implement the approved plan
- If OpenCode asks a question during Build:
  1. **Immediately switch back to Plan**
  2. Answer and confirm the revised plan
  3. Switch back to Build

## Git Workflow

### Before Implementation

Create a feature branch before any coding:
```
git checkout -b feature/<descriptive-name>
```

Branch naming examples:
- `feature/add-login-screen`
- `feature/fix-cart-calculation`
- `feature/refactor-api-layer`

### After Verification Passes

Stage and commit:
```
git add -A
git commit -m "type(scope): description"
```

Commit types: `feat`, `fix`, `refactor`, `style`, `test`, `docs`

Examples:
- `feat(auth): add Google OAuth login`
- `fix(cart): resolve checkout calculation error`

Rules:
- Single commit per feature
- Never commit to main directly
- **Always verify before commit**

## Verification Suite

After Build completes, run in order:

### Flutter Projects
```
flutter test
flutter analyze
flutter build
```

### Web Projects
```
npm test
npm run lint
npm run build
```

### Pass Criteria

All steps must pass:
- Tests: 0 failures
- Lint: 0 errors (warnings acceptable)
- Build: successful

### Conditional: Security Scan

Run security checks when:
- Handling user input/data
- Touching auth/payments/sensitive data
- Database/API changes

Skip for:
- UI-only changes (styling, widgets)
- Pure refactoring with no new inputs

## Failure Handling

| Failure | Response |
|---------|----------|
| Tests fail | Report which tests failed → return to Plan → fix → re-run |
| Lint errors | Report specific errors → return to Plan → fix → re-run `analyze` |
| Build fails | Report build error → return to Plan → check syntax/imports → fix → re-run |
| Plan unclear | Ask OpenCode to rewrite → do NOT switch to Build → get user confirmation |
| OpenCode asks questions in Build | Immediately switch to Plan → answer → confirm → switch back to Build |

## Status Reporting

After each major step, report:
```
PHASE: <phase_name>
STATUS: SUCCESS | NEEDS_INPUT | FAILED
SUMMARY: <one line>
```

## Important Rules

1. **NEVER skip user review.** Always return the plan for approval before executing.
2. **NEVER execute Build without explicit user approval** ("yes", "approved", "go", "execute").
3. **NEVER write code yourself.** All code is written by OpenCode.
4. **Keep plans focused.** Break large requests into smaller, reviewable chunks.
5. **Report progress.** Monitor OpenCode output and report completion or issues.
6. **Run verification before commit.** Never commit unverified code.

## Project Detection

| Marker | Project Type | Verification Commands |
|--------|-------------|----------------------|
| `pubspec.yaml` | Flutter | `flutter test`, `flutter analyze`, `flutter build` |
| `package.json` | Web | `npm test`, `npm run lint`, `npm run build` |
| `pyproject.toml` or `requirements.txt` | LLM/Python | `pytest`, `ruff check .`, `python -m build` |
