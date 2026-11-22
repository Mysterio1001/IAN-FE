---
name: ian-fe
description: Generate or modify Ian-style iMES Vue 2 and Element UI pages from design images, wireframes, or requirements. Use for CRUD screens, drill-down reports, scanner workstations, and their frontend API modules.
---

# Ian frontend development

**Version: 1.0.0**

**Author: Ian Wu**

For internal use only by FOREVERWIN INFORMATION TECHNOLOGY CO., LTD. To report an issue, email <wuguanru@semi-tech.com>.

Use the rules below as the implementation brief. Work in the user's target project. Do not introduce a new UI framework or copy example modules into the output. Communicate plans, issues, and delivery in Traditional Chinese unless the user requests another language.

## Execution workflow

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

- Keep transport separate. Import the target project's `request` from `@/router/axios` and the confirmed service prefix from `@/api/prefix` for known routes. Use named exported functions returning the request promise. Keep loading, messages, state resets and rendering in the view. Import using the target project's actual API path; do not derive the filename or service prefix mechanically from the view name.
- CRUD names: `fetchList(params)`, `getById(params)`, `saveObj(data)`, `delObj(handle)`. Typical contracts use GET with `params`, POST with `data`, DELETE with `params: {handle}`. Reports often use `rpt`, `fetchList(query)`, `getMaterialReceipt(query)` or `moFetch(query)`; master data/workstations often use `iep`. Domain actions may use `link`, `unlink`, `startScrewTask`. Determine HTTP method and parameter placement from the contract, not the verb in the function name.
- Construct `payload` in the caller. A paged response commonly exposes `records` and `total` under `res.data.data`; read single-record and task responses by their own schema. Do not assume a label is a server field or invent response data.
- **URL missing:** still create the API file and named function for each known operation. Keep the URL literally empty and mark it `url: '', // IAN TO DO`. Do not wrap an empty string in a prefix, guess a route, or report fake success. Guard the request inside the function so even automatic page loads cannot send an empty URL. Example for a confirmed GET query whose route alone is pending:

```js
import request from '@/router/axios'

// 查詢清單
export function fetchList(params) {
  const config = {
    url: '', // IAN TO DO
    method: 'get',
    params
  }
  if (!config.url) return Promise.reject(new Error('查詢 API 尚未設定'))
  return request(config)
}
```

Surface the pending state in the caller without a network request. If method or parameter placement is also unknown, leave the operation guarded until the contract is supplied; do not copy GET from the example as a default.

## Verification checklist for step 5

- Compare visible controls, labels, ordering and actions to the design; check grid layout, component reuse, naming, comments, styles and file boundaries. No unrequested business actions.
- Resolve imports; trace `init.js` keys → model/column props → payload. Match child props and emitted events to parent listeners. Check a drill-down uses the clicked row and correct detail function.
- Exercise applicable paths: search resets page; view cannot write; copy cannot overwrite the source; cancel sends nothing; save failure preserves input and releases loading; reopen does not retain stale data; repeated scans cannot double-submit; leaving the page stops listeners/polling.
- For every missing endpoint verify the export exists, the literal URL/comment is present and request is unreachable until configured. Keep it on the pending list.
- Use available target-project lint/build/tests and UI checks. Distinguish observed results from static inspection and unavailable tests in delivery.