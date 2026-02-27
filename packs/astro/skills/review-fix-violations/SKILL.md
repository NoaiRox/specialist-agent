---
name: review-fix-violations
description: "Use when architecture violations were found and need automated fixing by priority."
user-invocable: true
argument-hint: "[module]"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

Identify and fix `docs/ARCHITECTURE.md` violations in the specified module.

Module: $ARGUMENTS

## Steps

1. Run `/review-check-architecture` to find violations.

2. For each violation found, fix in priority order:

   **Critical (fix first):**
   - try/catch in services -> remove, move error handling to page frontmatter or endpoint
   - Transformation in services -> move to adapter
   - `set:html` without sanitization -> remove or sanitize
   - `dangerouslySetInnerHTML` in islands -> remove or sanitize
   - Hardcoded secrets -> move to env vars

   **Important:**
   - `client:load` where `client:idle` or `client:visible` would suffice -> change directive
   - Islands that could be .astro components -> convert to .astro (zero JS)
   - `any` types -> type correctly
   - Client-side data fetching -> move to frontmatter
   - Raw `<img>` tags -> use `<Image />` from `astro:assets`

   **Improvements:**
   - Console.log -> remove
   - TODO/FIXME -> resolve or create issue
   - Missing Content Collection schemas -> add

3. Validate after each fix:
```bash
npx astro check && npx astro build && npx vitest run --passWithNoTests
```

4. Report what was fixed and what remains.
