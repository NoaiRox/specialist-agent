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
find modules/$ARGUMENTS -type f | head -40
find pages/ -path "*$ARGUMENTS*" -type f 2>/dev/null
find server/api/$ARGUMENTS -type f 2>/dev/null
```

2. Summarize in quick format:

### Endpoints
List the consumed endpoints (grep in services and server/api/).

### Main Components
List components and their responsibilities (1 line each).

### State
- What is in useState? (simple shared state)
- What is in Pinia? (complex client state)
- What is in useAsyncData/useFetch? (server state)

### Data Flow
```
Service ($fetch) -> Adapter (parse) -> Composable (useAsyncData) -> Component (UI)
```

### Server API Routes
List Nitro routes in server/api/ (1 line each).

### Points of Attention
- Anything outside the ARCHITECTURE.md pattern?
- Components > 200 lines?
- Legacy code not yet migrated?
- Explicit imports where auto-imports should work?

3. Output: concise summary that a new developer can understand the module in 2 minutes.
