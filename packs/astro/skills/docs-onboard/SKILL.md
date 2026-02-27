---
name: docs-onboard
description: "Use when onboarding onto a module and need a quick summary of its structure, patterns, and key files."
user-invocable: true
argument-hint: "[module-name]"
allowed-tools: Read, Bash, Glob, Grep
---

Create a quick summary of a module for onboarding.

Module: $ARGUMENTS

## Steps

1. Map the structure:
```bash
find src/modules/$ARGUMENTS -type f | head -40
find src/pages/ -type f | head -40
find src/components/ -type f | head -40
find src/islands/ -type f | head -40
```

2. Summarize in quick format:

### Endpoints
List the consumed endpoints (grep in services).

### Pages
List pages and their routes (1 line each).

### Components
List .astro components and their responsibilities (1 line each).

### Islands
List interactive islands, their framework, and hydration strategy.

### Content Collections
List collections and their schemas (if applicable).

### Data Flow
```
Service (fetch) -> Adapter (parse) -> Frontmatter (await) -> Component (render)
```

### Points of Attention
- Anything outside the ARCHITECTURE.md pattern?
- Islands that could be .astro components?
- Missing Content Collection schemas?
- Legacy SPA patterns not yet migrated?

3. Output: concise summary that a new developer can understand the module in 2 minutes.
