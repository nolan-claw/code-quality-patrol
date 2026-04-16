---
name: code-quality-patrol
version: 1.0.0
description: Systematic code quality inspection for PRs and codebases — lint, type safety, test coverage, dead code
---

# Code Quality Patrol

Systematic code quality inspection for pull requests and codebases. Run before merging, after Ralph dispatches, or on a schedule.

## Quick Patrol (per PR)

```bash
# From repo root on the bridge
ssh matchpoint@HOST bash -lc "cd /home/matchpoint/repositories/REPO && \
  export PATH=/home/matchpoint/.local/bin:\$PATH && \
  pnpm build 2>&1 | tail -20 && \
  pnpm lint 2>&1 | tail -20 && \
  pnpm test 2>&1 | tail -20"
```

## Full Patrol (comprehensive)

### 1. Build Health
- `pnpm build` — must pass with zero errors
- Check for warnings that indicate stale types or missing deps
- Turbo cache misses signal dirty state — run `pnpm install` first

### 2. Type Safety
- `tsc --noEmit` — zero errors
- No `as any` casts (grep for them)
- No `@ts-ignore` or `@ts-expect-error` without documented reason
- Zod schemas for all API boundaries

### 3. Lint
- `pnpm lint` — zero errors
- Watch for: unused imports, inconsistent type assertions, missing return types on public functions

### 4. Test Coverage
- `pnpm test` — all pass
- New features MUST have tests
- Ralph-generated code often has weak tests — verify coverage on critical paths
- Integration tests > unit tests for API routes

### 5. Dead Code
- Unreferenced exports
- Commented-out code blocks
- Unused dependencies in package.json

### 6. Security
- No hardcoded secrets or API keys
- No `eval()` or `Function()` constructors
- Dependencies: check for known vulnerabilities (`pnpm audit`)

## Post-Dispatch Checklist

After Ralph completes a dispatch, run this checklist before considering the PR ready:

```bash
# On the worktree branch
pnpm build                    # Must pass
pnpm lint                     # Must pass
pnpm test                     # Must pass
git diff main..BRANCH --stat  # Review scope of changes
git diff main..BRANCH -- '*.ts' | grep -E '(as any|@ts-ignore|eval)'  # Red flags
```

## Common Ralph-Generated Issues

Ralph is fast but introduces patterns that need cleanup:

| Issue | Detection | Fix |
|-------|-----------|-----|
| `as any` type casts | `grep -r "as any" src/` | Replace with proper Zod schemas |
| Missing error handling | `grep -r "catch.*{}" src/` | Add proper error logging |
| Stale imports | `pnpm lint` catches most | Remove unused imports |
| Test only happy path | Manual review | Add error case tests |
| Hardcoded values | `grep -rE "(localhost|127\.0\.0\.1|hardcoded)" src/` | Use env vars |
| Missing Zod validation at API boundary | Check route handlers | Add schema.parse() |

## Reporting Format

```
PATROL: PR #XX — TITLE
Build: PASS/FAIL
Lint: PASS/FAIL (N warnings)
Tests: PASS/FAIL (M/N passing)
Type Safety: CLEAN/N issues found
Security: CLEAN/N issues found
Verdict: SHIP/FIX NEEDED
```
