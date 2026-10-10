# DevFlow — Design System and UX/UI Specification

| Field | Value |
|---|---|
| Status | Draft v0.1 |
| Derived from | `mvp.md` §4, §5, §8, §9, §10, §11, §14, §17; requirement IDs from [prd.md](prd.md) |
| Related | [architecture.md](architecture.md) · [rules.md](rules.md) · [memory.md](memory.md) · [security.md](security.md) · [phases.md](phases.md) |

## 0. Evidence status and how to read this document

- **VERIFIED existing design conventions: none.** No repository, components, stylesheet, tokens, fonts or brand guidelines were available. Nothing here claims an existing palette, font, component library or breakpoint.
- `mvp.md` §17 supplies *candidate content* for screens (not layouts). It supplies **no** colors, typography, spacing or brand direction. Everything visual in §4 is **PROPOSED** and needs approval; the only measured claim is contrast ratios, computed with the WCAG 2.x formula (§4.2).
- Where `mvp.md` is silent, this document specifies behaviour and layout constraints, not pixel-perfect mockups. No personas, user research or user accessibility needs are asserted.
- Status tags: **REQ** (from `mvp.md`), **PROPOSED**, **OPEN**.

## 1. Design goals derived from requirements

Each goal states what must be observable in the UI. Anything not traceable to a requirement is omitted.

| ID | Goal | Source | Observable consequence |
|---|---|---|---|
| DG-01 | Every number is explainable | NFR-010, FR-061, FR-070 | Each metric value has a "Definition" and a "View records" control reachable by keyboard; each alert lists rule, threshold, observed value, window, evidence |
| DG-02 | Missing is never zero | BR-01, FR-054 | Unavailable metrics render a state label and reason text, never `0`, `–` alone, or an empty chart point plotted at zero |
| DG-03 | Data completeness is always visible | BR-09, FR-090, FR-064 | Sync state and coverage appear on Overview and in every record view that depends on partial data |
| DG-04 | Active ≠ completed | BR-02, FR-084 | Active intervals use the label "Waiting" / "Open" with an elapsed-so-far value and are excluded from completed statistics; never share a style with completed values |
| DG-05 | Link method is never hidden | BR-04, FR-043 | Every task↔PR relationship shows its match status and method |
| DG-06 | Team-level framing | BR-05, RISK-09 | No screen ranks, sorts totals by, or scores individuals; alert text never attributes delay to a person |
| DG-07 | Corrections are auditable and reversible-in-record | FR-045, FR-120 | Every state-changing action that is audited says so and shows who/when in history |
| DG-08 | Cohort and unit are always stated | BR-06, BR-12 | Every metric card and chart shows cohort, window, time mode (elapsed/working) and n |

## 2. Product-specific constraints on the UI

1. Core vocabulary is fixed by `mvp.md` and must appear verbatim: connection states `Connecting`, `Syncing`, `Connected`, `Needs attention`, `Disconnected`; insight states `open`, `acknowledged`, `resolved`; match states `confirmed_manual`, `matched_exact_key`, `suggested`, `unmatched`, `rejected`. UI labels in §5 are PROPOSED display text for those enums.
2. Forbidden UI language (REQ §9.8, §10.3, §15.4): "productivity", "score", "leaderboard", "ranking", "underperforming", "slow reviewer", "caused the delay".
3. No decimal-precision time displays (REQ §9.6 "avoid decorative precision"); see §4.5.
4. Unavailable deployment-based measures (merge-to-production, deployment frequency) get **no card or chart**. They appear only in Settings → Metric definitions with status "Unavailable — requires a deployment source" (PROPOSED; OPEN D-03).

## 3. Information architecture and navigation (PROPOSED)

Top-level destinations, in order: **Overview · Tasks · Analytics · Bottlenecks · Integrations & Data Quality · Settings**.

| Item | Rule | Basis |
|---|---|---|
| Tasks | Shown only if the chosen task source gives DevFlow task data | OPEN OQ-01; `mvp.md` §9.4 |
| Analytics | Shown only if OQ-18 resolves to "in MVP" | `mvp.md` §17.3 not in §7.1 Must list |
| Lifecycle explorer, Metric evidence, Bottleneck detail | Reached by drill-in, not a top-level item | `mvp.md` §8.2–8.4 journeys |
| Navigation form | Horizontal top bar (≤6 items, no nesting). Below 640 px it collapses to a single "Menu" button opening the same list. No sidebar | Fewest items, no hidden hierarchy |
| Account/workspace | Workspace name + account menu (sign out) at far end of top bar | FR-001 |
| Integration attention indicator | If any integration is not `Connected`, the "Integrations & Data Quality" item shows a text badge ("Needs attention" / "Syncing"), not only a colored dot | DG-03 |
| Scope selectors | Repository selector + date-window selector appear on Overview, Tasks, Analytics, Bottlenecks. Selection is held in URL query parameters so reload, back/forward and shared links preserve it | PROPOSED; needed for evidence links (DG-01) |
| Default date window | A workspace setting (FR-111). Initial default value: OPEN D-05 | `mvp.md` §22 Q7 related |

Route paths are not specified (stack OPEN, OQ-11).

## 4. Design tokens (all PROPOSED; no existing tokens verified)

Minimum system required to implement the screens. Token names are stack-neutral CSS custom properties; if the chosen styling system differs, keep the semantics and values.

### 4.1 Color — single light theme

Dark theme is not required by `mvp.md`; deferred (OPEN D-06).

| Token | Value | Semantic use |
|---|---|---|
| `--color-bg` | `#FFFFFF` | Page background |
| `--color-surface-subtle` | `#F6F7F9` | Table header rows, code/ID chips, disabled areas |
| `--color-border` | `#8A93A2` | Control and table borders that must meet 3:1 (inputs, buttons, checkboxes) |
| `--color-divider` | `#D0D5DD` | Decorative row separators only (not required to meet 3:1) |
| `--color-text` | `#1A1F29` | Primary text |
| `--color-text-muted` | `#4A5361` | Secondary text, captions, labels |
| `--color-accent` | `#1F5FBF` | Links, primary buttons, focus ring, selected nav item |
| `--color-ok` / `--color-ok-bg` | `#1B6B3A` / `#E8F4EC` | `Connected`, `resolved`, `confirmed_manual`, `matched_exact_key`, Completed, Complete coverage |
| `--color-warn` / `--color-warn-bg` | `#7A4E00` / `#FFF4DB` | `Needs attention` (non-error cause), `suggested`, Partial coverage, Active/Waiting, `acknowledged` |
| `--color-error` / `--color-error-bg` | `#B3261E` / `#FDECEA` | Failed sync/processing, Invalid, `Disconnected` with error, `rejected` |
| `--color-info` / `--color-info-bg` | `#1F5FBF` / `#E8F0FB` | `Connecting`, `Syncing`, `open` insight, informational notices |
| `--color-neutral` / `--color-neutral-bg` | `#4A5361` / `#EEF0F3` | `Disconnected` (user-initiated), `unmatched`, Unavailable, Unknown coverage |

Rules:
- Color never carries meaning alone: every status has a visible text label and an icon shape distinct per status family (check, clock/spinner, triangle, cross, dash). 
- Status color pairs are used for chips and notices only. Body text stays `--color-text` on `--color-bg` or `--color-surface-subtle`.
- No gradients, translucency, or decorative color.

### 4.2 Contrast (computed with the WCAG relative-luminance formula)

| Pair | Ratio | Requirement met |
|---|---|---|
| `--color-text` on `--color-bg` / `--color-surface-subtle` | 16.51 / 15.40 | ≥4.5:1 text |
| `--color-text-muted` on bg / subtle | 7.77 / 7.25 | ≥4.5:1 text |
| `--color-accent` on bg / subtle | 6.09 / 5.68 | ≥4.5:1 text, ≥3:1 focus ring |
| White on `--color-accent` (primary button) | 6.09 | ≥4.5:1 text |
| ok text on ok-bg | 5.79 | ≥4.5:1 |
| warn text on warn-bg | 6.59 | ≥4.5:1 |
| error text on error-bg | 5.72 | ≥4.5:1 |
| info text on info-bg | 5.30 | ≥4.5:1 |
| neutral text on neutral-bg | 6.81 | ≥4.5:1 |
| `--color-border` on bg | 3.10 | ≥3:1 non-text UI |

These were computed during drafting; re-run if any value changes.

### 4.3 Typography

| Role | Spec |
|---|---|
| Font family | System UI sans stack (`system-ui, -apple-system, "Segoe UI", Roboto, sans-serif`); monospace stack (`ui-monospace, "SFMono-Regular", Menlo, Consolas, monospace`) for delivery IDs, task keys, PR numbers. Rationale: no webfont dependency, no invented brand font. |
| Page title | 24 px / 32 px, weight 600 |
| Section heading | 18 px / 28 px, weight 600 |
| Body | 14 px / 20 px, weight 400 |
| Dense table/body | 13 px / 20 px |
| Caption, labels | 12 px / 16 px, `--color-text-muted` |
| Metric value | 28 px / 36 px, weight 600, `font-variant-numeric: tabular-nums` |
| Timestamps/durations in tables | tabular numerals; no truncation |

Text may be resized to 200% without loss of content (§12).

### 4.4 Spacing, layout, shape

| Token | Value |
|---|---|
| Spacing scale | 4, 8, 12, 16, 24, 32, 48 px (4 px base). Component padding uses 8/12/16; section gaps 24/32. |
| Content max width | 1200 px, left-aligned container with 16 px (<1024) / 24 px (≥1024) side padding |
| Breakpoints | `compact` <640 px · `medium` 640–1023 px · `wide` ≥1024 px (PROPOSED; nothing verified) |
| Radius | 4 px on controls, chips, cards. No larger radii |
| Border | 1 px `--color-divider` for cards/tables; 1 px `--color-border` for form controls |
| Elevation | None for cards and tables. Menus/popovers/dialogs only: 1 px `--color-border` plus `0 2px 8px rgba(0,0,0,0.12)`. Dialog backdrop: `rgba(26,31,41,0.5)` |
| Control height | Inputs/buttons 36 px min (target size ≥24×24 CSS px per WCAG 2.2 AA); primary actions 40 px |
| Grid | `wide`: metric cards in rows of up to 3; `medium`: 2; `compact`: 1 |

### 4.5 Value formatting rules (PROPOSED; needed so DG-02 and "no decorative precision" are testable)

| Value | Format |
|---|---|
| Duration < 1 h | `42 min` |
| 1 h ≤ duration < 48 h | `19 h 20 min` (omit minutes if 0: `19 h`) |
| Duration ≥ 48 h | `3 d 5 h` |
| Sample size | `n = 24` |
| Count | integer, no abbreviations |
| Timestamp | `YYYY-MM-DD HH:mm` followed by explicit offset `(UTC+05:30)`; the column header or timeline header states the display timezone; UTC offset is never omitted. Event detail also exposes the exact UTC instant |
| Unavailable | text `Unavailable` + reason; never `0`, `0 min`, `—` alone, or `N/A` alone |
| Percent (coverage) | integer percent with numerator and denominator available on hover/focus and in text for screen readers (`18 of 24 PRs linked`) |

### 4.6 Motion

Only functional motion: loading indicator and expand/collapse of detail rows (≤150 ms). With `prefers-reduced-motion: reduce`, the indicator is replaced by static text ("Loading…") and no transition runs. No decorative animation.

### 4.7 Iconography

Small inline icon per status family (check, clock, spinner, triangle-exclamation, cross, dash, external-link) at 16 px beside text. No illustrations, no empty-state artwork. Icon set choice OPEN (follows styling decision, OQ-11).

## 5. Status vocabulary (maps enums to labels, color and icon)

| Domain | Enum | Label shown | Color family | Icon |
|---|---|---|---|---|
| Connection | `Connecting` | Connecting | info | spinner |
| | `Syncing` | Syncing | info | spinner |
| | `Connected` | Connected | ok | check |
| | `Needs attention` | Needs attention | warn (error if credentials revoked) | triangle |
| | `Disconnected` | Disconnected | neutral | dash |
| Sync completeness | complete / partial / unknown | Fully synced / Partial sync / Sync status unknown | ok / warn / neutral | check / triangle / dash |
| Metric result | completed | (value shown) | — | — |
| | active | Waiting (PR) / Open | warn | clock |
| | unavailable | Unavailable | neutral | dash |
| | invalid / pending reconciliation | Needs reconciliation | error | triangle |
| Data coverage | complete / partial / unknown | Complete data / Partial data / Coverage unknown | ok / warn / neutral | check / triangle / dash |
| Link match | `confirmed_manual` | Confirmed by user | ok | check |
| | `matched_exact_key` | Matched by task key | ok | check |
| | `suggested` | Suggested — needs confirmation | warn | triangle |
| | `unmatched` | Not linked | neutral | dash |
| | `rejected` | Rejected | error | cross |
| Insight | `open` | Open | info | circle |
| | `acknowledged` | Acknowledged | warn | check-circle |
| | `resolved` | Resolved by user / No longer detected (see FR-103) | ok | check |

"Resolved by user" and "No longer detected" are two distinct labels sharing the `resolved`-style chip so FR-103 is visible; the underlying representation is OPEN (OQ-09).

## 6. Shared components (only those the screens need)

| Component | Responsibility | Behaviour |
|---|---|---|
| StatusChip | Render a vocabulary entry from §5 | Text + icon + color; never color alone; not interactive |
| CompletenessNotice | Banner shown when scope is partial, failed, or sync status unknown | Placed directly above the data it qualifies; text names the repository and the specific limitation; `role="status"`; contains a link to Integrations & Data Quality. Not dismissible while the condition holds |
| MetricCard | One metric for the chosen cohort and window | Fields in order: metric name (links to its definition) · value or state · cohort label · window · time mode (`elapsed`/`working`) · `n = …` · CoverageChip · "View records" button. Whole-card click is not used; the two named controls are |
| EvidenceTable | List of PRs/tasks behind a metric or alert | Columns defined per screen; sortable on duration and timestamps only (never by person); row opens Lifecycle explorer |
| Timeline | Ordered events for one PR/task | `<ol>`; each item: event name, occurred_at, received_at (when different or on expand), actor reference as source fact, source link; gap items are visibly distinct (dashed marker, text "No event recorded for: …") |
| IntervalMarker | Shows metric start/end inside a timeline | Labels "Start of MET-01 interval" / "End …" with definition version |
| InsightRow / InsightDetail | List item and full view of an alert | See S-06 |
| LinkControl | Show/modify a task↔PR relationship | Shows StatusChip + method; actions Confirm / Change / Remove depending on state and role |
| ThresholdField | Numeric setting with unit | Label states unit; helper text states default and what it affects |
| ConfirmDialog | Destructive or audited actions | Names the exact object and consequence; primary button text repeats the action verb ("Disconnect GitHub"); focus starts on the cancel button |
| InlineMessage | Form and action results | `role="alert"` for errors; `role="status"` for success; adjacent to the control, not a transient toast |

## 7. Screens

Common to all authenticated screens: workspace isolation (NFR-001) means a record the user cannot access shows the **Not found** state (identical to nonexistent), never "forbidden" details. Loading state of every data region: a labelled placeholder (text "Loading <region>…"), never an empty table or zero values.

### S-01 Sign-in and account (FR-001) — PROPOSED minimal

| Aspect | Spec |
|---|---|
| Purpose | Authenticate; sign out; reach account management |
| Access | Unauthenticated (sign-in); authenticated (account) |
| Entry | Any protected URL while signed out redirects here and returns to the original URL after sign-in |
| Content | Controls depend on the authentication mechanism — **OPEN (OQ-11)**. Account area: display name, sign-out. Further account management scope: OPEN |
| Actions | Primary: Sign in. Secondary (account menu): Sign out |
| Validation / errors | Failure message states what failed without revealing whether an account exists (security.md §4) |
| States | Loading (button disabled with text "Signing in…"), error (InlineMessage), success (redirect) |
| Responsive/keyboard | Single column at all widths; Enter submits; error message moved into focus order |
| Requirements | FR-001 |

### S-02 Workspace setup and Connect GitHub (UC-01, `mvp.md` §8.1)

| Aspect | Spec |
|---|---|
| Purpose | Create workspace, authorize GitHub, select repositories, start sync |
| Access | Signed-in user without a workspace; owner/authorized role afterwards (OQ-12) |
| Entry | First sign-in with no workspace; "Connect GitHub" from Integrations & Data Quality |
| Structure | Three numbered steps with visible current step: (1) Workspace name (2) Connect GitHub (3) Select repositories. After step 3 the user lands on S-08 where progress is shown |
| Step 2 content | Before leaving to GitHub, list the permissions DevFlow requests and one sentence of purpose each. **The list is OPEN until the minimal permission set is chosen (security.md §5).** Primary: "Continue to GitHub". Secondary: Cancel |
| Step 3 content | Checklist of repositories the installation exposes (name = display only; identity is the stable ID). "Select all" is not provided by default. Primary: "Start import". Disabled until ≥1 selected |
| Validation | Workspace name: required; trimmed; length bounds PROPOSED 1–80 characters (limit to confirm). Repository selection: ≥1 |
| States | Authorizing (waiting for GitHub return), authorization cancelled/denied (InlineMessage: "GitHub authorization was not completed. Nothing was connected." + retry), no repositories available (explain installation scope and link to change it on GitHub), repository list load error (retry), success → S-08 in `Syncing` |
| Responsive | Single column at all widths |
| Keyboard | Steps are sequential forms; focus moves to the step heading on step change |
| Requirements | FR-002, FR-010, FR-011, FR-012, FR-016, FR-110 |

### S-03 Overview (UC-02, `mvp.md` §9.6, §17.1)

| Aspect | Spec |
|---|---|
| Purpose | Show workflow health for a repository and window with data completeness |
| Access | Any signed-in workspace user |
| Entry | Default landing after sign-in; first destination after initial sync reaches a usable state |
| Hierarchy (top to bottom) | 1. Scope bar (repository selector, date-window selector, last successful sync) 2. CompletenessNotice (if applicable) 3. Metric cards: MET-01 first-review wait, MET-02 time to merge, "Open PRs currently waiting for first review" (count; labelled as PRs, never combined with tasks) 4. Current bottleneck alerts (max 5 rows then "View all in Bottlenecks") 5. Data-health summary: last sync, unmatched PR count, missing-data count (FR-064) |
| Primary actions | MetricCard "View records" → S-10; alert row → S-06 detail |
| Secondary | Metric name → definition (S-09 Metric definitions section) ; "Integrations & Data Quality" link |
| Metric card states | Value (completed population, with median and p75 if sample sufficient — OQ-17); `n` always; **insufficient sample** shows median only or `Unavailable — fewer than the required number of completed PRs in this window` (threshold OPEN OQ-17); **unavailable** shows reason naming the missing event/integration (FR-061); **no PRs in window** shows "No PRs in this window for this cohort" (not zero minutes). Active PRs are reported in the "waiting" count, not mixed into the median |
| Trend | Shown on a card only when both comparable windows have enough comparable data (FR-063): text "Change vs previous window: …" with both medians and both `n`; otherwise omitted with no placeholder |
| Empty states | No repository connected → CTA to S-02/S-08. Repository connected, import not yet complete → CompletenessNotice "Partial sync" + cards for imported data only. No alerts → "No open alerts for this window." |
| Error states | Metric query failed → card-level error with retry; other cards remain. Whole-page failure → single error with retry and link to S-08 |
| Loading | Per-region placeholders |
| Responsive | `wide` 3 cards per row; `medium` 2; `compact` 1, scope bar fields stack |
| Keyboard/a11y | Scope selectors are native-semantics controls; changing them updates URL and moves focus to the results heading; live region announces "Updated for <repo>, <window>" |
| Data dependencies / failure | Metric results (FR-053), insights, integration status. If metric results are stale versus a pending recompute, notice states "Recalculating after new data" and shows the previous value labelled with its calculation time |
| Requirements | FR-060–FR-064, FR-054, FR-081 |

### S-10 Metric evidence (records behind a metric) (`mvp.md` §8.2 steps 4–6, §7.1 #14)

| Aspect | Spec |
|---|---|
| Purpose | Show exactly which records produced a metric value |
| Entry | "View records" on MetricCard, or chart point drill-down |
| Header | Metric name, definition version (link), cohort, window, time mode, `n`, coverage, calculated_at (with timezone) |
| EvidenceTable columns (MET-01/02) | PR (number + title) · repository · start event time · end event time · duration · state (Completed / Waiting / Needs reconciliation / Unavailable) · task link status. Default sort: duration descending. Excluded records are listed under a labelled "Excluded from this metric" section with the exclusion reason (e.g., "Draft, never marked ready", "Bot review only") |
| Actions | Row → S-07; export CSV (SHOULD, FR-131, OQ-14) |
| States | Empty ("No records in this cohort for this window"), loading, error with retry, partial data (CompletenessNotice) |
| Responsive | Table scrolls horizontally inside its own container at `compact`; first column sticky |
| Keyboard | Sortable headers are buttons; row link is the PR title |
| Requirements | FR-061, FR-053, FR-056 |

### S-04 Tasks (`mvp.md` §17.2) — conditional on OQ-01

| Aspect | Spec |
|---|---|
| Purpose | Inspect tasks, linked PRs, link method/status, unmatched or suggested links, and correct links |
| Existence | Not built until the task source is chosen. Path A: tasks are GitHub issues (distinguish issue from PR visibly in every row). Path B: native tasks — keep fields minimal per `mvp.md` §9.4 |
| Columns | Task key · title (source link) · task state · linked PRs (count) · link StatusChips · source link. Never sorted or grouped by assignee |
| Filters | Repository, date window, task status (SHOULD, FR-130); link status |
| Actions | Open task/PR link controls (S-07); confirm/change/remove per I-05 |
| States | Empty ("No tasks in this scope"), "No task-key format configured — exact-key matching is off" (when FR-115 unset, links only manual), loading, error |
| Task lead time (MET-03) | Only displayed when reliable state transitions exist; with Path A the UI must label any open→close duration as "Issue open to close", never "lead time" (`mvp.md` §10.2C) |
| Requirements | FR-040–FR-046, FR-130 |

### S-05 Analytics (`mvp.md` §17.3) — candidate, OQ-18

If included: metric selector (MET-01, MET-02 only), cohort and window, median/p75 with `n`, one trend chart (§9), drill-down to S-10, definition and coverage panel. If OQ-18 is "not in MVP", drill-down via S-03/S-10 satisfies FR-061 and this screen is deferred.

### S-06 Bottlenecks (UC-03, `mvp.md` §8.3, §9.8, §9.10, §17.4)

| Aspect | Spec |
|---|---|
| Purpose | List rule-based alerts, inspect evidence, acknowledge or resolve |
| Access | Any signed-in workspace user may view; acknowledge/resolve permission follows OQ-12 (OPEN; default PROPOSED: all workspace users) |
| List | Filters: status (`open`, `acknowledged`, `resolved`; default `open`), rule, severity. Row: rule name · severity · observed value vs threshold · window · affected-item count · status chip · created / last updated. Severity labels: OPEN until OQ-09 |
| Detail (InsightDetail) | Order: 1. Plain statement (template below) 2. Rule ID/version, threshold, observed value, window, affected count 3. CompletenessNotice for the repository (RULE-01 requires "whether qualifying review events were checked" and sync/coverage status) 4. EvidenceTable of affected PRs/tasks, with ready-for-review time, evaluation time and elapsed-so-far; links to source GitHub items 5. Status history (actor, timestamp, note) 6. Actions |
| Statement template, RULE-01 | "`<n>` PR(s) have been waiting for a qualifying human review for longer than the configured threshold of `<threshold>` in `<repository>`. Evaluated at `<timestamp>`. Last successful sync: `<timestamp>`." Illustrative values in `mvp.md` §11.2 are **not** defaults. Never use: "reviewers are unproductive", "caused the release delay" |
| Active vs completed | Active intervals show "Waiting for `<elapsed so far>`" with warn styling; completed durations never appear in RULE-01 evidence |
| Actions | Acknowledge, Resolve (each with optional note), see I-04. Both disabled with reason when the user lacks permission or the insight is `resolved` |
| Auto-clear vs user resolve | If the condition is no longer detected, status label "No longer detected" with timestamp; if the user resolves, "Resolved by `<role/name>`" with timestamp (FR-103). Reopen display: when a resolved insight's condition recurs, it returns to `open` and history shows a "Reopened" entry. **The rule for reopening (immediately, after cooldown, only if auto-cleared) is OPEN (OQ-09).** |
| States | Empty ("No alerts match these filters"), none-ever ("No bottleneck rules have produced an alert in this window"), loading, error, rule evaluation unavailable because data incomplete ("Alerts may be incomplete: <reason>") |
| Responsive | List becomes stacked rows at `compact`; Detail is a full page |
| Requirements | FR-080–FR-084, FR-100–FR-103 |

### S-07 Lifecycle explorer (`mvp.md` §9.7, §17.5)

| Aspect | Spec |
|---|---|
| Purpose | Answer "what happened, and when?" for one PR (and its linked task if any) |
| Entry | Row links from S-10, S-06, S-04; direct lookup |
| Header | PR number/title, repository, current source state (open / draft / merged / closed), source link (shown only if the user is authorized per `mvp.md` §9.7), link StatusChip(s) |
| Body | Timeline (§6). Columns: event · occurred at · received at · actor reference · source. Order = occurrence order, ties broken by event semantics; late/out-of-order events are marked "Received after later events" |
| Metric overlay | IntervalMarkers for MET-01/MET-02 with definition version; events that were used are tagged "Used in MET-01" |
| Gaps | Where a required event is missing: dashed item "No `ready for review` event recorded" and the metric row states `Unavailable`. No inferred timeline step is ever drawn |
| Review rounds | Collapsed groups per review round (SHOULD); dismissed reviews appear with "Dismissed" and are not counted unless the definition says so |
| Filters | In list entry points (SHOULD, FR-072) |
| States | Loading, not found, partial data (CompletenessNotice), reconciliation pending ("Source and stored state differ; under reconciliation") |
| Responsive | Single column timeline at all widths; metadata columns collapse into labelled lines at `compact` |
| Keyboard | Timeline is an ordered list; expandable event details use disclosure buttons |
| Requirements | FR-070–FR-072, FR-025, FR-071, FR-043 |

### S-08 Integrations & Data Quality (`mvp.md` §9.9, §17.6)

| Aspect | Spec |
|---|---|
| Purpose | Establish whether analytics can be trusted; manage connection |
| Access | View: any workspace user. Connect/disconnect/scope change/reconcile: authorized role (OQ-12) |
| Hierarchy | 1. Integration summary: StatusChip, installation account, repository scope, connected timestamp 2. Sync block: sync type, start time, progress **only if measurable** (otherwise "Progress not available"), last successful sync, last reconciliation, fully/partial indicator 3. Problems list (plain-language error summary, time, what the user can do) 4. Data quality metrics: unmatched PRs, missing required events blocking a metric (names the metric), delayed/failed/awaiting-reconciliation counts where known, link coverage over window (SHOULD) 5. Actions |
| Actions | Primary depends on state: `Needs attention` → resolve instruction or "Retry sync"/"Reconcile" (only if safe, FR-091/FR-133); Secondary: "Change repositories", "Disconnect" (ConfirmDialog) |
| `Needs attention` copy | States cause category in words (e.g., "GitHub rate limit reached; sync will resume", "Permission missing: `<permission>`", "Access revoked in GitHub"), last successful sync, and that existing data is preserved. Exact cause list follows GitHub error handling (architecture.md §8) |
| Processing failures | Authorized admins see failed-event list with retry status (SHOULD). Non-admins see counts only |
| States | `Connecting`, `Syncing` (progress), `Connected`, `Needs attention`, `Disconnected` (retention note: existing data is kept or deleted per policy — wording OPEN until OQ-10), empty (no integration → CTA S-02), loading, load error |
| Live updates | Status change is announced to assistive tech via `role="status"`; polling or push mechanism OPEN (architecture) |
| Responsive | Blocks stack at `compact`; tables scroll within container |
| Requirements | FR-012, FR-013, FR-018, FR-019, FR-090, FR-091, FR-113 |

### S-09 Settings (`mvp.md` §9.11, §17.7)

Sections on one page with in-page anchors (no nested nav): **Repository scope · Metric definitions · Alert thresholds and date window · Task-key convention · Bot handling · Timezone and working calendar (only if OQ-06 includes working time) · Retention and deletion · Integration**.

| Section | Spec |
|---|---|
| Repository scope | Same checklist as S-02 step 3; changes audited; removing a repository asks what happens to its data (wording OPEN, OQ-10) |
| Metric definitions | Read-only list of every metric (including unavailable ones such as deployment-based), showing name, definition version, start event, end event, inclusions/exclusions, unit, elapsed/working, default cohort, status behaviour, and availability |
| Thresholds and window | ThresholdFields per enabled rule, each showing unit, default, what it affects, and "Last changed by/at". Save is explicit (button), not on blur. Saved changes create an audit record and state that dependent insights are re-evaluated |
| Task-key convention | Prefix/pattern field (SHOULD, FR-115) with live example match against sample text; exact semantics OPEN with matcher design |
| Bot handling | Checkbox "Exclude bots from qualifying reviews" (default on, per `mvp.md` §10.2A); helper text: "Bot detection depends on GitHub account type; unclassifiable accounts are shown as unknown." |
| Retention and deletion | Displays the documented policy (OPEN, OQ-10) and the delete-data action (ConfirmDialog; requires typing the workspace name — PROPOSED because the action is irreversible) |
| Validation | See §8. Unsaved changes: navigating away prompts once |
| Access | Edit: authorized role (OQ-12). Others: read-only with explanation |
| Requirements | FR-110–FR-117, FR-114, FR-120 |

## 8. Form validation and feedback patterns (PROPOSED)

| Field | Rule | Message pattern |
|---|---|---|
| Workspace name | Required, trimmed, 1–80 chars | "Enter a workspace name (1–80 characters)." |
| Repository selection | ≥1 | "Select at least one repository." |
| Time-threshold field | Number > 0, unit shown in label (hours); upper bound OPEN | "Enter a number greater than 0 (hours)." |
| Count-threshold field | Integer ≥ 1 | "Enter a whole number of 1 or more." |
| Link reason / insight note | Optional (mandatory-ness: OQ-08); length counter; limit confirmed with backend | "Note is too long (`n`/`limit`)." |
| Task-key convention | Must compile under the chosen matcher | "This pattern isn't valid: `<reason>`" |

Feedback rules: errors appear next to the field and in a summary at the top when ≥2; the form is not cleared on error; success text states what changed and what was recalculated ("Threshold saved. Open alerts will be re-evaluated."); server rejections caused by authorization show "You don't have permission to change this." with the required role named once OQ-12 resolves.

## 9. Interactions and state specifications

| ID | Interaction | Initial | Trigger | System response | Validation / errors | Loading / completion | Empty / failure | Permission | Persistence / navigation |
|---|---|---|---|---|---|---|---|---|---|
| I-01 | Connect GitHub (UC-01) | No integration, S-02 step 2 | "Continue to GitHub" | Redirect to GitHub authorization; on return, create Integration `Connecting`, load repositories | Cancelled/denied → message, no integration created; missing permission → FR-019 explanation | "Waiting for GitHub…" until return; then step 3 | No repositories → explanation; failure → retry | Authorized role | Step state kept on return; leaves nothing half-connected on cancel |
| I-02 | Select repositories and start import | Step 3, none selected | "Start import" | Persist scope, create resumable sync job, set `Syncing`, go to S-08 | ≥1 repo | Progress block on S-08 | Job enqueue failure → "Import could not start" + retry, scope kept | Authorized role | Scope change audited (FR-120) |
| I-03 | Inspect a metric | Overview card | "View records" | Open S-10 with same repository/window/cohort in URL | Unavailable metric → page shows reason instead of table | Table placeholder | No records / error states per S-10 | Workspace user | URL preserves scope; Back returns to Overview with scroll position |
| I-04 | Acknowledge / resolve insight | Detail, status `open` | Acknowledge or Resolve button | Dialog with optional note → set status, record actor/timestamp, keep original evidence unchanged (FR-101), update history | Note too long; insight changed concurrently → "This alert changed. Reload to see its current state." | Buttons show "Saving…", disabled | Failure → InlineMessage, status unchanged | Per OQ-12 | New status visible in list and detail without full reload |
| I-05 | Confirm / change / remove a link | Task or PR with a link or suggestion | Confirm / Change / Remove | Dialog states PR, task, current status; optional reason (OQ-08 may make mandatory); on confirm, write link + audit, schedule recompute, show "Recalculating" on affected metrics | Target already linked → show conflict; remove of `confirmed_manual` explains consequence | Link shows new StatusChip immediately with "Recalculating…" until done | Failure → unchanged, error shown | Authorized per OQ-08 | History visible on the link (who, when, reason) |
| I-06 | Recover from sync failure (UC-05) | `Needs attention` | Open S-08 / "Retry sync" or "Reconcile" | Resume from checkpoint; show resumed state; on completion record result and show last successful sync | Retry unavailable → button disabled with reason | `Syncing` then `Connected` or `Needs attention` with updated message | Repeated failure → message persists with last error time | Authorized role for actions | Last known valid data stays visible, labelled partial/stale (BR-09) |
| I-07 | Disconnect integration | `Connected` / `Needs attention` | "Disconnect" | ConfirmDialog stating that DevFlow will stop importing and describing what happens to existing data (wording after OQ-10) → revoke/stop → `Disconnected` | — | Button "Disconnecting…" | Revocation call fails → `Needs attention` with explanation; no further calls are made | Authorized role | Audited; Overview shows Disconnected notice |
| I-08 | Change threshold or window | Settings loaded with current values | Save | Validate, persist, audit, re-evaluate insights, show confirmation text | §8 rules | "Saving…" | Failure → values kept in form, error shown | Authorized role | Window change affects scope bar default only |
| I-09 | Change scope selectors | Any scoped screen | Select repository or window | Reload data for scope; update URL | Invalid window (start after end) rejected inline | Region placeholders | Empty scope states | Workspace user | Scope retained when navigating between scoped screens |
| I-10 | Delete workspace data | Settings → Retention | "Delete data" | ConfirmDialog requiring workspace name typed; perform deletion per policy | Name mismatch keeps button disabled | "Deleting…" then confirmation text of what was deleted | Partial failure lists what remains | Authorized role | Audited if audit records survive per OQ-10 |

## 10. Charts (only if S-05 is built, or a trend appears on a card)

- Allowed: one line chart of median (and p75) per comparable window; one distribution view of completed durations. Nothing else.
- Time axis windows use the cohort named on the card. Y axis in hours starting at 0, with unit label; no dual axes; no smoothing.
- Windows with insufficient data are **gaps with an annotation**, never zero points.
- Each point shows `n` in its tooltip and in the companion table.
- Every chart has an adjacent table with identical values, reachable by keyboard; color is not the only differentiator (distinct line styles for median vs p75).
- Active intervals are not plotted in completed-duration series.
- Stage-share percentages are shown only when a non-overlapping partition is defined (BR-07); otherwise stages are listed separately.

## 11. Responsive layout rules

| Rule | `compact` <640 | `medium` 640–1023 | `wide` ≥1024 |
|---|---|---|---|
| Navigation | Menu button | Top bar | Top bar |
| Metric cards per row | 1 | 2 | up to 3 |
| Scope bar | Stacked fields | Inline | Inline |
| Tables | Scroll in own container, first column sticky; page body never scrolls sideways | Same if needed | Fit |
| Dialogs | Full-width, max height with internal scroll | Centered, 480 px max | Centered, 480 px max |
| Timeline | Vertical list; metadata as labelled lines | Vertical | Vertical with columns |

Required: no loss of content or function at 320 px width and at 200% text zoom (WCAG reflow).

## 12. Accessibility and keyboard (PROPOSED; NFR-013 awaits approval)

| Requirement | Specification |
|---|---|
| Standard | Target WCAG 2.2 AA |
| Focus | Visible 2 px `--color-accent` outline with 2 px offset on every interactive element; never removed |
| Order | Skip-to-content link first; DOM order matches visual order; scope bar precedes results |
| Controls | Native elements (links, buttons, inputs, select) wherever possible; custom widgets must implement the corresponding ARIA pattern and keyboard keys |
| Dialogs | Focus moves into dialog (to Cancel for destructive), is trapped, `Esc` closes, focus returns to the trigger |
| Status | Sync/connection changes and "Updated for …" use `role="status"`; form errors use `role="alert"` |
| Tables | Proper header cells, caption naming the metric/alert; sortable headers expose current sort state |
| Timeline | `<ol>`; gap items have text, not just styling |
| Color | Never sole carrier (§4.1); contrast per §4.2 |
| Target size | ≥24×24 CSS px; 36 px default control height |
| Zoom/reflow | 200% text, 320 px viewport (§11) |
| Motion | `prefers-reduced-motion` honored (§4.6); no auto-updating content that moves |
| Timestamps | Include explicit offsets in text, readable by screen readers |

## 13. Traceability

| PRD IDs | Screens / interactions |
|---|---|
| FR-001–FR-004 | S-01, S-02, S-08 access, I-01 |
| FR-010–FR-016 | S-02, S-08, I-01, I-02, I-07 |
| FR-017–FR-030 | Surfaces on S-07 (timeline), S-08 (failures); not directly user-facing |
| FR-040–FR-046 | S-04, S-07, I-05 |
| FR-050–FR-058 | S-03, S-09 Metric definitions, S-10, §4.5 |
| FR-060–FR-064 | S-03, I-03, I-09 |
| FR-070–FR-072 | S-07 |
| FR-080–FR-084, FR-100–FR-103 | S-06, I-04 |
| FR-090–FR-091 | S-08, I-06 |
| FR-110–FR-117 | S-09, I-08, I-10 |
| FR-120 | S-06 history, S-04/S-07 link history, S-09 "Last changed" |
| FR-130–FR-133 | S-04/S-10 filters and export, S-08 notices, S-08 reconcile |
| BR-01–BR-14, NFR-010, NFR-013 | §1, §2, §4.5, §5, §12 |

## 14. Design-specific acceptance criteria

| ID | Criterion (testable) |
|---|---|
| DAC-01 | With a fixture PR missing its ready-for-review event, MET-01 renders "Unavailable" with the reason; no `0` or `0 min` appears anywhere in the DOM for that metric |
| DAC-02 | Every MetricCard shows name, value-or-state, cohort, window, time mode, `n`, coverage chip, and working "View records" and definition links |
| DAC-03 | An active (waiting) PR appears as "Waiting for …" and is not included in the completed median on S-03; it appears in S-10 with state "Waiting" |
| DAC-04 | During a partial sync, S-03 and S-08 each show a Partial sync notice; the text "Fully synced" is absent until the sync job completes |
| DAC-05 | Each of the five connection states renders label + icon; removing all color (grayscale) leaves states distinguishable |
| DAC-06 | A RULE-01 alert detail shows rule ID/version, threshold, observed value, window, affected count, evaluation timestamp, sync/coverage status, and links to affected records |
| DAC-07 | No string in the UI matches the forbidden-language list (§2.2) — enforced by an automated string check on UI copy |
| DAC-08 | Acknowledge and resolve each record actor and timestamp visible in history; original evidence is unchanged afterward |
| DAC-09 | A `suggested` link is never displayed with a confirmed style; Confirm/Reject actions are available to permitted roles |
| DAC-10 | All screens operate by keyboard alone: every action reachable, focus visible, dialogs trap and restore focus |
| DAC-11 | At 320 px width and 200% zoom, no horizontal page scroll and no clipped controls |
| DAC-12 | Contrast checker passes for every token pair in §4.2 on rendered components |
| DAC-13 | `prefers-reduced-motion: reduce` produces no animated indicators |
| DAC-14 | Changing repository or window updates the URL; reloading the page restores the same scope |
| DAC-15 | A user from workspace B requesting a record ID from workspace A sees the same Not found state as for a nonexistent ID |
| DAC-16 | Timestamps in all tables display an explicit UTC offset |

## 15. Unresolved design decisions

| ID | Decision | Depends on |
|---|---|---|
| D-01 | Visual direction beyond §4 (brand, logo, typeface) | Product owner; §4 is a functional minimum, not a brand |
| D-02 | Authentication UI (S-01) | OQ-11 |
| D-03 | Whether unavailable deployment metrics are shown only in Settings or also as a labelled stub on Overview | `mvp.md` §9.6 vs §17.1 wording overlap; PROPOSED here: Settings only |
| D-04 | Task source UI (S-04 columns, native task editing if Path B) | OQ-01 |
| D-05 | Default date window | OQ-07 / product decision |
| D-06 | Dark theme | Not required by `mvp.md`; deferred |
| D-07 | Whether S-05 Analytics ships | OQ-18 |
| D-08 | Severity labels and levels for alerts | OQ-09 |
| D-09 | Role-based visibility of actions | OQ-12 |
| D-10 | Real-time vs polled status updates | architecture (OQ-11) |
| D-11 | Permission list shown before GitHub authorization | Final GitHub App permission set |
| D-12 | Notification channels (in-product only vs email/Slack) | OQ-13 |
