---
name: ian-fe
description: Generate or modify Ian-style iMES Vue 2 and Element UI pages from design images, wireframes, or requirements. Use for CRUD screens, drill-down reports, scanner workstations, and their frontend API modules.
---

# Ian frontend development

**Version: 1.2.0**

**Author: Ian Wu**

For internal use only by FOREVERWIN INFORMATION TECHNOLOGY CO., LTD. To report an issue, email <wuguanru@semi-tech.com>.

Use the rules below as the implementation brief. Work in the user's target project. Do not introduce a new UI framework or copy example modules into the output. Communicate plans, issues, and delivery in Traditional Chinese unless the user requests another language.

## Entry: route by the requested outcome

Choose one path from the user's request. Editing an existing page does not automatically mean a minor adjustment; file count alone does not determine the path.

- **微調 (minor adjustment):** A bounded change to existing behavior or presentation, e.g. move a permission condition onto its button or adjust one search label. Note the target paths in one sentence, inspect the affected code and its direct dependencies, then make the smallest change. Work locally without delegation or the six-step workflow. Review the diff against the baseline, check the affected interaction and report the result. Skip unrelated templates and page-wide checks.
- **新建或編輯完整頁面 (full page):** A new page, a page-wide redesign, or a change that requires coordinating a new flow across search, table, dialog and/or API contracts. Follow the full workflow below, even on an existing page. If inspection shows a minor request needs this coordination, switch paths and explain why; switching never expands write permission.

Both paths must follow the write scope and all template, component and API rules applicable to the change. If the intended outcome is unclear and the choice would materially change the work, ask one focused question; otherwise proceed without asking the user to select a mode.

## Write scope: applies to every step and every agent

- Before writing, record the workspace baseline (staged, unstaged and untracked changes). In the plan, list the exact paths to create or edit. By default, only the requested page module's needed files and its dedicated API file are eligible. Honor any narrower user limit. A plan or delegation cannot grant permission beyond the user's scope.
- Everything else is read-only unless the user explicitly authorizes a specific expansion. This includes the shared Axios wrapper and interceptors, shared components, other business modules, global routes, permissions, i18n, styles, dependencies and build configuration. Importing or inspecting a file is not permission to change it. For a missing URL, leave the operation unwired in the view; do not change Axios.
- Give each worker only its assigned subset of the allowlist, including on repair. Apply the boundary to creates, edits, deletions and renames, including indirect writes from generators, formatters and `lint --fix`. Use non-writing checks when a command's changes cannot be confined.
- If a fix needs an out-of-scope change, leave that file untouched, finish independent in-scope work and report the blocked behavior, file and reason. Only explicit user authorization can expand the list. Do not duplicate shared infrastructure inside the module to bypass this rule.

## Full-page execution workflow

1. **Receive requirements.** Identify the target page, new vs. existing behavior, and the requested controls, actions, dialogs and API contracts. For each action trace input → local guard → request, if any → success and failure UI. Do not infer editability, permissions, persistence or payload keys from a screenshot alone.
2. **Read rules.** Apply the implementation rules below. For an edit, inspect the actual page and its imports first. If an interface or convention needs evidence, inspect a behaviorally similar module in the target project. Check `src/components` for suitable controls; verify props, events, slots, registration and actual usage before reuse. Do not invent import paths if source is unavailable.
3. **Plan.** Produce a short file-level plan: page type, file owners, shared field names, props/events, API exports, pending contracts and acceptance checks. Separate confirmed behavior from assumptions. Continue directly to implementation without asking for plan approval. If part of a contract is unknown, isolate that part and implement the known scope.
4. **Generate with subagents.** Give each worker these rules, its files, the relevant design constraints, agreed interfaces and acceptance checks. Use one worker for a small page; parallelize only disjoint file ownership (e.g. API vs. search UI vs. dialog). Assign one owner to page integration. If the host cannot launch subagents, execute the same assignments locally; never claim delegation that did not occur. No approval gate between planning, generation and verification.
5. **Verify and repair.** The lead reviews the integrated changes against the design, the rules and the checks below. Run applicable existing target-project checks. On an implementation failure, send the responsible worker the file, violated rule, evidence and expected result; return to step 4, then recheck the fix and affected integrations. If an external dependency or unavailable environment blocks verification, record that limitation rather than cycling or claiming success.
6. **Deliver.** Report changed files, implemented interactions, checks actually run and their results. List pending endpoints, unresolved requirements and untested flows separately. A correctly guarded missing URL is pending integration, not a failed implementation or a successful live test.

## Choose the file structure by behavior

Use English kebab-case for module directories, business component files and API filenames. Use `index.vue` for the page and `init.js` for named exports of defaults, column definitions and picker settings: `initSearch`, `initForm`, `tableOptions`, domain-specific `*Options`. Initialize mutable state with the project's `copyJSON` helper or confirmed equivalent; do not mutate shared defaults.

- **CRUD master data:** `index.vue` owns `searchForm`, `tableData`, pagination, selected record and mode. `ui/search-ui.vue` binds the search fields and emits `search` and `clear`; the parent resets or queries. Use one `dialog/main.vue` for requested `new`, `view`, `edit`, `copy` modes. Get persisted details by `handle` when that is the contract; clear the original handle when copying. On successful save emit `afterSave` for the parent to refresh. On close emit `update:visible`, restore form defaults and clear validation on `$nextTick`.
- **Read-only report:** Keep search separate. A report `searchReset` can clear conditions, results and page without querying again. Use a business-named dialog per distinct detail source/column set, not per clickable cell. Pass the current row and `visible`; load on open, clear on close. Do not add CRUD controls to a read-only report.
- **Workstation:** Keep scanning, focus, row selection, messages, signals and task progression coordinated by the page. Reuse existing display/input controls; do not split every visual box into a component or add list pagination by default. Stop polling/timers and remove listeners on teardown.

## Vue template, layout and style

- Write Vue 2 SFCs: template → script → optional scoped style. Use Options API sections as needed: `components`, `props`, `data()`, `computed`, `watch`, lifecycle hooks, `methods`. Use two spaces, single-quoted JS strings, usually no semicolons, one attribute per line for multi-attribute tags. Preserve project `.sync`, `slot="label"`, `v-slot:header` syntax; do not migrate to Vue 3.
- Build layout rows and columns with `el-row` and `el-col`; use `gutter`, `span`, `justify` and alignment props instead of nested layout divs. Reserve `div` for a real purpose, e.g. the single `execution` root or a scroll viewport. On a list page put search → optional actions → table → right-aligned pagination in `basic-container`, with business dialogs outside it under the same root.
- Bind `el-form` with `ref`, `:model`, `@submit.native.prevent`; match `el-form-item` prop to the bound field. Search rows often use gutter 15 and span 8, not fixed requirements. Use existing `tooltip` and confirmed `$t(...)` keys; retain confirmed domain labels if no key is supplied. Never manufacture translation keys.
- For an actual lookup, use the project's `el-input` suffix icon with `@click.stop` and `input-select-dialog` if available. Supply `modal-obj`, `modal-fetch-fun`, `modal-show.sync`; handle `inputDialogSelected` to fill the intended field. Verify any custom directive or globally registered component exists. Workstation-specific examples include `text-msg`, `traffic-signal`, `qr-code`, `Scanner`; only use them for matching interactions.
- Put normal column definitions (`prop`, `label`, `width`, `minWidth`) in `init.js`; render using `v-for="col in tableOptions"` and `:key="col.prop"`. Use slots for headers, clickable quantities or row actions. Typical tables use borders, centered cells and overflow tooltips. Paginated lists start with `current: 1`, `size: 20`, `total: 0`; reset to page one on search/page-size change, keep filters on page change.
- Use small primary add buttons and mini circular row-action buttons when applicable. For edit dialogs use the existing draggable dialog directive, footer buttons and protection against accidental backdrop/Escape dismissal.
- Prefer component props over custom CSS. A single local alignment/spacing may be inline; repeated or nested styling belongs in `<style scoped>` or `<style lang="scss" scoped>`. Use table style callbacks for dynamic status. Preserve visible link affordance for report drill-downs. Do not globalize a page-specific height, negative margin or `!important`.

## Custom classes and sanitized template patterns

Start with no new CSS class. Prefer Element UI layout props and verified existing styles. Do not invent wrappers or switch to inline CSS merely to avoid this rule. Preserve existing project and framework class names (e.g. `el-icon-search`); do not rename them to fit a local convention. A `div` needed as a Vue 2 root does not by itself need a class.

Only introduce a local class for a specific visual or interaction requirement that those options cannot meet. Name every new custom class **lowerCamelCase** (e.g. `resultViewport`, not `result-viewport`, `ResultViewport` or `result_viewport`), including tokens supplied through `:class` or table class callbacks. Keep its rule scoped to the component.

The following fragments illustrate boundaries, not mandatory markup or complete pages. Bind only fields, actions, labels and component interfaces confirmed for the target. Do not add sample features just because they appear here.

**Component templates are allowlists.** For each component use only the parameters shown in its applicable template. Omit unused ones; do not add props, pass-through attributes, listeners or slots merely because a library supports them. If a requirement needs an interface not in the template, report the mismatch rather than extending it silently. In every Vue SFC script, only imports may precede `export default`; no functions, variables, constants or other declarations outside it. This does not change the named-export formats for `init.js` and API files.

**`init.js` — matched search, form and column keys.** Keep only definitions used by the page. The parent initializes `searchForm` with `copyJSON(initSearch)` and passes it to the search component.

```js
export const initSearch = { code: '' }
export const initForm = { code: '', description: '' }
export const tableOptions = [
  { prop: 'code', label: '代碼', minWidth: '120' },
  { prop: 'description', label: '說明', minWidth: '150' }
]
```

**`ui/search-ui.vue` — search emits intent; the parent owns query and reset.** When lookups are needed, put all `input-select-dialog` instances first, as the first children of the template root, before forms, containers or other content. Omit pickers for ordinary text inputs.

```vue
<template>
  <div>
    <el-form ref="searchForm" :model="searchForm" @submit.native.prevent>
      <el-row :gutter="15">
        <el-col :span="8">
          <el-form-item prop="code">
            <span slot="label">代碼</span>
            <el-input v-model="searchForm.code" clearable />
          </el-form-item>
        </el-col>
        <el-col :span="8" style="text-align: right">
          <el-button type="primary" size="small" icon="el-icon-search" @click="$emit('search')">查詢</el-button>
          <el-button size="small" icon="el-icon-delete" @click="$emit('clear')">清除</el-button>
        </el-col>
      </el-row>
    </el-form>
  </div>
</template>

<script>
export default {
  props: {
    searchForm: { type: Object, default: () => ({}) }
  }
}
</script>
```

**CRUD actions — include only requested operations.** The add button goes above the table; the other four belong in its action column. Bind each required `v-if` directly on its `el-button`, never on the enclosing grid, column or slot. `canAdd`, `canEdit` and `canDelete` below represent confirmed permission predicates, not new permission keys. Apply the same placement to view or copy if their actual permissions differ.

```vue
<el-button v-if="canAdd" type="primary" size="small" icon="el-icon-plus" @click="handleClick('new')">新增</el-button>
```

```vue
<el-table-column label="操作">
  <template v-slot="scope">
    <!-- 查看 -->
    <el-button type="warning" size="mini" icon="el-icon-view" circle @click="handleClick('view', scope.row)" />
    <!-- 修改 -->
    <el-button v-if="canEdit" type="primary" size="mini" icon="el-icon-edit" circle @click="handleClick('edit', scope.row)" />
    <!-- 複製 -->
    <el-button v-if="canAdd" type="success" size="mini" icon="el-icon-document-copy" circle @click="handleClick('copy', scope.row)" />
    <!-- 刪除 -->
    <el-button v-if="canDelete" type="danger" size="mini" icon="el-icon-delete" circle @click="handleDel(scope.row)" />
  </template>
</el-table-column>
```

**Single business dialog in `index.vue`:** use the direct `main` component name (`<main/>` shorthand), not a business-specific alias. Preserve the bindings needed by `dialog/main.vue` (`:visible.sync`, `:model`, `:type`, `@afterSave`). Check component resolution: lowercase `main` is a native HTML element in standard Vue 2; if so, import and register `Main` and write `<Main ... />` in the SFC so the dialog actually mounts. Only distinguish dialogs with descriptive aliases when there are two or more.

**Report — a detail cell selects the parent row; a read-only dialog receives it.** In the table column's slot, use an existing text action rather than styling an ordinary cell as a link. On open, the detail component loads its own records.

```vue
<template v-slot="scope">
  <el-button type="text" @click="showDialog(scope.row)">
    {{ scope.row[col.prop] }}
  </el-button>
</template>
```

```vue
<DetailDialog
  :visible.sync="detailVisible"
  :modal-obj="currentRow" />
```

**Workstation — keep scan input and feedback together.** Enter triggers the page's scan handler, not form submission. Substitute a verified target-project feedback component if one fits. Do not introduce polling or device integration unless required.

```vue
<el-row :gutter="15">
  <el-col :span="8">
    <el-input
      ref="scanInput"
      v-model="input"
      :disabled="isLoading"
      @keyup.enter.native="handleScan" />
  </el-col>
  <el-col :span="16">{{ message }}</el-col>
</el-row>
```

## Naming and comments

- Use camelCase. UI events: `handleSearch`, `handleSave`, `handleDel`, `handleSelectionChange`; use `handleClick(type, row)` when actions open the same dialog in different modes. Page-level `getList()` builds a payload, calls API `fetchList(payload)` and updates the list. Use `getCurrent(row)` for details or meaningful workstation verbs such as `loadTask`.
- Name helpers by effect (`showDialog`, `closeDialog`, `clearSelection`, `dataFormatter`), predicates by question (`isRowSelectable`), and state by role (`customerVisible`, `tableLoading`). Use computed properties for derived state, watchers for side effects such as loading on open. Match emitted and listened-to names exactly; do not reproduce incidental typos.
- Write short Traditional Chinese template block comments such as `<!-- 搜尋欄位 -->`. Use `// API` or `// TOOL` when grouping methods helps navigation. Comment non-obvious reasons, e.g. `// 複製時清空 handle`, and annotate opaque column keys with the business label where useful. Put a short Chinese action comment above API functions. Do not narrate obvious lines.

## Frontend guards vs. backend authority

- Enforce only specified rules determinable from current UI state. Match required fields to `el-form` rules and run `$refs.form.validate` before saving. Apply `maxlength`, uppercase conversion, mutually exclusive fields or date dependencies only when specified. Do not impose one report's search prerequisites on all lists.
- In view mode disable editing, hide save and guard picker triggers. Use confirmed permission keys for button visibility; server authorization is still authoritative. Confirm destructive actions with `$confirm`; cancellation sends nothing.
- Use early returns for missing input, no selection or an in-flight action. A loaded option list may give immediate scan feedback, but cannot prove that a device, relationship or task is still valid. Leave uniqueness, referential integrity, concurrent conflicts, authorization and final state transitions to the backend.
- Pair `showLoading()` with `hideLoading()` in `finally`, or use a local `tableLoading` flag with `v-loading`. Only refresh/close on success. Preserve editable input on failure; account for the shared request layer's error reporting without swallowing errors or showing duplicate alerts.

## API modules

- **Mandatory API file template:** allow only `import request from '@/router/axios'`, an optional named import of the required service prefix from `@/api/prefix`, comments, and one named `export function` per endpoint. No other imports, re-exports, constants, helpers or executable module-level code. Each function body must consist of exactly one `return request({ ... })` with an inline object literal: `url`, confirmed `method`, and `params` or `data` as required by the contract. No guards, validation, transforms, retries, caching, mocks, `async/await`, promise handlers or UI logic. Build payloads and handle responses in the caller. Use the actual API filename and prefix, not ones inferred from the view name.
- CRUD names: `fetchList(params)`, `getById(params)`, `saveObj(data)`, `delObj(handle)`. Typical contracts use GET with `params`, POST with `data`, DELETE with `params: {handle}`. Reports often use `rpt`, `fetchList(query)`, `getMaterialReceipt(query)` or `moFetch(query)`; master data/workstations often use `iep`. Domain actions may use `link`, `unlink`, `startScrewTask`. Determine HTTP method and parameter placement from the contract, not the verb in the function name.
- Construct `payload` in the caller. A paged response commonly exposes `records` and `total` under `res.data.data`; read single-record and task responses by their own schema. Do not assume a label is a server field or invent response data.
- Preserve existing API filenames, paths, export names and call signatures, including legacy casing; naming conventions apply to new modules only. Do not rename or rewrite unrelated endpoints. All new or changed endpoint implementations must use the mandatory template above; legacy names are not permission for alternate implementation styles.
- **URL missing:** create the named function in the same template with `url: '', // IAN TO DO`. Do not wrap the empty URL in a prefix. Do not import or call the pending function until configured. Skip automatic loads; for a manual action stop in its page method with `return // IAN TO DO：待 API URL 確認後串接` before loading, confirmation or success effects. Do not add `el-alert`, toast, disabled state or other UI solely for a missing URL. Example for a confirmed GET query whose route alone is pending:

```js
import request from '@/router/axios'

// 查詢清單
export function fetchList(params) {
  return request({
    url: '', // IAN TO DO
    method: 'get',
    params
  })
}
```

If method or parameter placement is unknown, omit the unknown fields and leave the operation unwired; do not copy GET from the example as a default.

## Full-page verification checklist for step 5

- Inspect every added `class`, `:class`, class callback and CSS selector. For each new custom class, identify the requirement and why existing layout props/styles do not suffice; remove it if unnecessary. Check lowerCamelCase for each new custom token, not framework or pre-existing names.
- Compare actual changes (including staged, unstaged, untracked, deleted and renamed paths) against the baseline and write allowlist. Review diffs for files that were already dirty. Never reset, clean or revert someone else's changes. Report unexpected changes; do not mark a scope violation as passed or widen the list to make a repair pass.
- Compare visible controls, labels, ordering and actions to the design; check grid layout, component reuse, naming, comments, styles and file boundaries. No unrequested business actions.
- Resolve imports; trace `init.js` keys → model/column props → payload. Match child props and emitted events to parent listeners. Check a drill-down uses the clicked row and correct detail function.
- Exercise applicable paths: search resets page; view cannot write; copy cannot overwrite the source; cancel sends nothing; save failure preserves input and releases loading; reopen does not retain stale data; repeated scans cannot double-submit; leaving the page stops listeners/polling.
- Reject an API file with an unapproved import, extra declaration or any function body other than a single direct request return. For each missing endpoint verify its URL/comment and check that no caller invokes it (including lifecycle hooks and watchers). Keep it on the pending list.
- Use available target-project lint/build/tests and UI checks. Distinguish observed results from static inspection and unavailable tests in delivery.