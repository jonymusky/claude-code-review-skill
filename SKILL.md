---
name: pr-review
description: "CodeRabbit-style PR review with severity grouping, diff suggestions, and a single consolidated AI agent prompt block. Use when asked to review a PR, review code, check quality, or find bugs/security issues."
user_invocable: true
---

# PR Code Review Skill

Performs thorough code reviews on Pull Requests with findings grouped by severity, concrete diff suggestions, and a consolidated "Prompt for AI Agents" block that can be copy-pasted to fix all issues at once.

## When to Use

When user asks to:
- Review a PR (by number or URL)
- Review code changes
- Check code quality / Find bugs or security issues
- Get PR feedback

## How to Review

### 1. Gather PR Context

```bash
# Get PR metadata and file list
gh pr view <PR_NUMBER> --json title,body,state,headRefName,baseRefName,files,commits

# Get the full diff
gh pr diff <PR_NUMBER>
```

If the PR has many files (>30), use subagents in parallel to review backend and frontend separately.

### 2. Review Checklist

For each changed file, check:

**Security (Critical)**
- Hardcoded secrets or fallback credentials
- PII in logs (names, emails, phones, URLs)
- Cross-tenant / authorization bypass
- Injection vulnerabilities (SQL, XSS, CSS, command)
- Token reuse / replay attacks
- Unvalidated file paths or S3 keys
- Missing input validation at system boundaries

**Code Quality (Medium)**
- Error handling patterns (project-specific: asyncHandler, try/catch)
- Database query performance (.lean(), .select(), indexes)
- Logging conventions (centralized logger vs console.log)
- Controller thickness (business logic belongs in services)
- Type safety (avoid `as any` casts)
- Validation completeness (backend must enforce what frontend validates)

**Frontend**
- API client usage (project-specific patterns)
- Responsive design
- i18n (no hardcoded user-facing strings)
- SSR guards for browser APIs
- Component size and reusability
- Accessibility (aria labels, roles)

**Conventions** (adapt to each project's CLAUDE.md or coding guidelines)
- File naming, imports, code style
- Test coverage for new endpoints
- API documentation files (Bruno, Swagger, etc.)
- Migration patterns

### 3. Format the Review

Post a **single comment** on the PR using this exact template structure:

```markdown
## 🔍 Code Review — PR #<NUMBER>

**Actionable comments posted: <N>** | **Nitpick comments: <N>**

---

<details>
<summary>🔴 Critical / High findings (<N>)</summary><blockquote>

<details>
<summary>path/to/file.ts (<N>)</summary><blockquote>

`<line or range>`: **Short title of the finding.**

Explanation of why this is a problem and what could go wrong.

\`\`\`diff
- old code
+ suggested fix
\`\`\`

</blockquote></details>

</blockquote></details>

---

<details>
<summary>🟡 Medium findings (<N>)</summary><blockquote>

<!-- Same nested structure as above -->

</blockquote></details>

---

<details>
<summary>🧹 Nitpick comments (<N>)</summary><blockquote>

<!-- Same nested structure, can use shorter descriptions -->

</blockquote></details>

---

<details>
<summary>✅ Things done well</summary><blockquote>

- Bullet list of positive patterns found in the PR

</blockquote></details>

---

<details>
<summary>📊 Summary</summary>

| Category | 🔴 Critical/High | 🟡 Medium | 🟢 Nitpick | Total |
|----------|-----------------|---------|-----------|-------|
| Security | X | X | X | X |
| Convention | X | X | X | X |
| Frontend | X | X | X | X |
| **Total** | **X** | **X** | **X** | **X** |

</blockquote></details>

---

<details>
<summary>🤖 Prompt for all review comments with AI agents</summary>

\`\`\`
Verify each finding against the current code and only fix it if needed.

Inline comments:

In @path/to/file.ts:
- Around line X: Description of what to fix. Be specific about the change:
  what to add, remove, or replace. Include function/variable names.

In @path/to/other-file.ts:
- Around line Y: Another fix description.

---

Nitpick comments:

In @path/to/file.ts:
- Around line Z: Nitpick description.
\`\`\`

</details>

🤖 _Review by Claude Code_
```

### Key Rules

1. **ONE comment** — never split the review across multiple comments.
2. **Severity grouping** — Critical/High, Medium, Nitpick. Each group is a collapsible `<details>` block.
3. **Concrete diffs** — always include a code suggestion, not just a description.
4. **Single AI prompt block** — ALL prompts consolidated at the end, separated into "Inline comments" (must fix) and "Nitpick comments" (nice to have). Do NOT put individual prompt blocks per finding.
5. **Positives** — always include a "Things done well" section. Reviews should be balanced.
6. **Summary table** — categories x severities with counts.
7. **Adapt to the project** — read CLAUDE.md or equivalent guidelines before reviewing. Use project-specific conventions in your checks.
8. **File:line references** — every finding must reference the exact file and line/range.
9. **No false positives** — verify each finding against the actual code. If unsure, skip it.
10. **Post via gh CLI** — use `gh pr comment <PR_NUMBER> --body "$(cat <<'EOF' ... EOF)"` to post.

### For Large PRs

When a PR has many files (>30) or touches multiple areas:

1. Launch parallel subagents — one for backend, one for frontend
2. Each subagent reads the diff via `gh pr diff <PR_NUMBER>` and reviews its area
3. Merge findings into one unified comment with the template above
4. De-duplicate findings that both agents may have caught
