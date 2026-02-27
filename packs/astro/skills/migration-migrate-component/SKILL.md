---
name: migration-migrate-component
description: "Use when a component needs migration to the target architecture or framework version."
user-invocable: true
argument-hint: "[component-file]"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

Migrate a SPA component (React/Vue/Svelte) to the Astro architecture following `docs/ARCHITECTURE.md` section 5.

Component: $ARGUMENTS

## Steps

1. Read `docs/ARCHITECTURE.md` section 5.

2. Analyze the component:
   - Count lines
   - List: props, state, effects, event handlers, data fetching
   - Determine: interactive or presentational?
   - Map who imports this component

3. If **presentational** (no interactivity) -> Convert to `.astro`:
   - Props -> `Astro.props` with `Props` interface
   - JSX/template -> Astro template syntax
   - CSS-in-JS/scoped styles -> scoped `<style>`
   - Remove all client-side JavaScript
   - Remove data fetching (move to page frontmatter)
   - Place in `src/components/`

4. If **interactive** (needs JS) -> Convert to island:
   - Keep framework code (React/Vue/Svelte)
   - Remove data fetching (move to page frontmatter, pass as props)
   - Keep only UI interactivity (state, event handlers)
   - Ensure props are serializable
   - Choose hydration strategy (`client:visible` > `client:idle` > `client:load`)
   - Place in `src/islands/`

5. Update pages that reference this component:
   - Add `client:*` directive if island
   - Move data fetching to page frontmatter

6. Validate:
```bash
npx astro check
npx astro build
npx vitest run --passWithNoTests
```
