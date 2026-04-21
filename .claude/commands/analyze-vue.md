# Analyze Vue Component

Analyze Vue 3 component(s) for performance issues and code reuse opportunities, then produce a prioritized report with concrete fix suggestions.

## Usage

```
/analyze-vue [path]
```

- `path` — a specific `.vue` file (e.g. `client/src/views/Dashboard.vue`) or a directory (e.g. `client/src/views`). If omitted, analyze all `.vue` files under `client/src/`.

## Steps

### 1 — Discover files

If a specific file was given, read that file.  
If a directory was given, glob for `**/*.vue` inside it and read each file.  
If nothing was given, glob `client/src/**/*.vue` and read all files.

### 2 — For each component, check these categories

**Performance**
- Template expressions that should be `computed` properties (any non-trivial expression called in the template that doesn't depend on event args)
- `watch` callbacks that recalculate derived state — these should usually be `computed`
- Missing `v-memo` on expensive list items that re-render often
- Missing `v-once` on static content inside dynamic lists
- `v-if` + `v-for` on the same element (always a bug — `v-if` should wrap the `v-for`)
- Large components (>300 lines) that do multiple unrelated things — split candidates
- Props passed straight through to a child without transformation (pass-through props suggest the parent shouldn't exist or should use `v-bind="$attrs"`)
- API calls or data fetching happening inside `computed` (side effects in computed)
- Missing `shallowRef` / `shallowReactive` for large read-only data structures

**Code Reuse**
- Identical or near-identical `<script setup>` logic blocks across multiple components — extract to a composable in `client/src/composables/`
- Repeated template markup patterns (same card/table/modal structure copy-pasted) — extract to a component
- Hardcoded values (colours, limits, URLs) that appear in multiple files — move to constants
- Data-fetching logic inline in components that could share a `useXxx()` composable

**Correctness / Best Practices**
- `v-for` without `:key`, or using `index` as key on a list that can reorder
- Mutating props directly instead of emitting events
- `async setup()` without `<Suspense>` wrapping the parent
- Missing `onUnmounted` cleanup for event listeners, intervals, or subscriptions set up in `onMounted`

### 3 — Score and prioritize

For each finding assign a severity:
- 🔴 **High** — causes bugs or measurable performance degradation
- 🟡 **Medium** — code smell or reuse opportunity with clear benefit
- 🟢 **Low** — minor improvement, style, or optional refactor

### 4 — Output format

Print a report structured like this:

```
## Vue Component Analysis

### [filename] (e.g. views/Dashboard.vue)

🔴 **[Issue title]**
Line ~XX: [what the problem is and why it matters]
Fix: [concrete code snippet showing the corrected pattern]

🟡 **[Issue title]**
...

---
### Composable extraction opportunities
[Only if 2+ components share logic]
Suggested composable: `useXxx.js`
Affected files: [list]
What to extract: [description]

---
### Summary
| Severity | Count |
|----------|-------|
| 🔴 High  | N     |
| 🟡 Medium| N     |
| 🟢 Low   | N     |

Top 3 highest-impact fixes: [numbered list]
```

Keep suggestions specific — reference the actual variable/function names from the code, not generic advice.
