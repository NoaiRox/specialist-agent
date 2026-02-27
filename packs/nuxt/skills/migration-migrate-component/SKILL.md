---
name: migration-migrate-component
description: "Use when a component needs migration to the target architecture or framework version."
user-invocable: true
argument-hint: "[component-file]"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

Migrate a Nuxt component from Nuxt 2 / Options API to Nuxt 3 `<script setup lang="ts">` following `docs/ARCHITECTURE.md` section 5.

Component: $ARGUMENTS

## Steps

1. Read `docs/ARCHITECTURE.md` section 5.

2. Analyze the component:
   - Count lines (template, script, style)
   - List: props, emits, data, computed, watch, methods, mixins, asyncData, fetch
   - Map who imports this component

3. Convert to `<script setup lang="ts">`:
   - `props` -> `defineProps<T>()`
   - `emits` -> `defineEmits<T>()`
   - `data()` -> `ref()` / `reactive()`
   - `computed:` -> `computed()`
   - `watch:` -> `watch()`
   - `methods:` -> `function`
   - `mounted()` -> `onMounted()`
   - `mixins: []` -> extract to composable `useXxx()`
   - `asyncData()` -> `useAsyncData()`
   - `fetch()` -> `useFetch()`
   - `$axios` -> `$fetch`
   - `this.$route` -> `useRoute()`
   - `this.$router` -> `useRouter()`
   - `this.$store` -> `useState()` / Pinia
   - Remove explicit imports (use auto-imports)

4. Apply composition pattern if there is prop drilling.

5. Decompose if > 200 lines.

6. Validate:
```bash
nuxi typecheck
nuxi build
npx vitest run --passWithNoTests
```
