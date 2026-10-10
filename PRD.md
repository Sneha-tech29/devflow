# DevFlow — Product Requirements Document (PRD)

| Field | Value |
|---|---|
| Status | Draft v0.1, derived from `mvp.md` v0.1 (last updated 2026-10-10) |
| Source of truth | `mvp.md` (REQUIREMENT). Where this PRD adds structure, IDs or defaults, they are labelled. |
| Repository evidence | **None.** No source code, manifests, schemas, configs or prior docs were available when this was written. Nothing here is VERIFIED against code. Re-run the inspection step when a repository exists. |
| Related | [architecture.md](architecture.md) · [design.md](design.md) · [rules.md](rules.md) · [memory.md](memory.md) · [security.md](security.md) · [phases.md](phases.md) |

Labels: **REQUIREMENT** (stated in `mvp.md`), **PROPOSED** (recommendation, not approved), **ASSUMPTION** (needs validation), **OPEN QUESTION** (decision needed), **VERIFIED** (repository evidence; currently none).

## 1. Product overview

DevFlow is a GitHub-first engineering workflow intelligence platform for small software teams. It connects work items to pull-request (PR) activity and turns the event history into transparent delivery metrics and evidence-backed bottleneck insights (`mvp.md` §2). The MVP focuses on traceability, metric transparency and basic bottleneck detection. It is not employee surveillance and not automated performance ranking.

### 1.1 Problem statement (REQUIREMENT, `mvp.md` §3)

Tasks, code, reviews and releases live in separate tools. Teams cannot tell which stage delays delivery, cannot agree on metric definitions, and cannot trust dashboards that do not show where a number came from. Missing links, late webhooks and incomplete imports can distort analytics without the user knowing.

MVP problem to solve: for a team using GitHub, reliably collect supported PR and review events, connect them to work items where possible, calculate a small set of precisely defined metrics, show the underlying evidence, and flag simple explainable bottlenecks.

### 1.2 Product principles (REQUIREMENT, `mvp.md` §5)

Evidence before inference · Missing is not zero · Definitions before dashboards · Source-of-truth clarity · Idempotent ingestion · Reproducible calculations · Human-confirmable linking · Privacy by design · Team improvement, not surveillance · Graceful uncertainty.

## 2. Goals and success criteria

Goals (REQUIREMENT, `mvp.md` §6.1): connect GitHub with least privilege; import and normalize PR, review and merge events; associate tasks with PRs through a transparent matching hierarchy; calculate a small documented metric set; show task/PR timelines and the events behind each value; detect simple explainable bottlenecks; expose sync health, unmatched activity and completeness; validate against a ground-truth dataset.

Success criteria. The targets below come from `mvp.md` §6.2 and are **proposed validation targets, not measured claims**. No numeric performance, adoption or time-saving targets are defined, and none should be added without pilot evidence (`mvp.md` §6.3).

| ID | Area | Target | Verified by |
|---|---|---|---|
| SC-01 | Metric correctness | 100% of approved deterministic ground-truth fixtures produce expected results | Fixture suite (phases.md Phase 3) |
| SC-02 | Duplicate handling | Reprocessing a duplicate delivery has no duplicate effect | NFR-002 tests |
| SC-03 | Recalculation | Same normalized event set + same definition version ⇒ same result | NFR-003 tests |
| SC-04 | Traceability | Every displayed metric has a visible definition and inspectable source-event evidence | FR-061, FR-070 acceptance |
| SC-05 | Data uncertainty | Missing required events show as unavailable/incomplete, never zero | FR-054 acceptance |
| SC-06 | Alert explainability | Every alert shows rule, threshold, observed value, window, evidence | FR-081 acceptance |
| SC-07 | Task matching | Matching method stored; ambiguous matches never treated as confirmed | FR-043, FR-046 acceptance |
| SC-08 | Sync visibility | Last successful sync and current sync error are visible | FR-090 acceptance |
| SC-09 | Security | Credentials and repository data protected per [security.md](security.md) | Security verification |
| SC-10 | Usability | A pilot user can connect a repository and reach a first useful dashboard without developer help (subject to GitHub authorization) | Pilot validation (OQ-15) |

## 3. Target users and roles

User types from `mvp.md` §4. No personas, research findings or accessibility needs beyond the source material are asserted.

| Type | Needs (from `mvp.md` §4.1) |
|---|---|
| Engineering lead / team lead | Where work is waiting; whether review queue is worsening; inspect events behind an insight before acting |
| Developer | Which linked PRs wait for review; what blocks a task; whether the dashboard's reading of a PR is correct |
| Engineering manager / project owner | Trend-level delivery and data quality, not individual scores |

Access roles: `mvp.md` §9.1 says a basic **owner/member** model is SHOULD; §22 Q12 asks whether it is needed at all. **OPEN QUESTION OQ-12.** Until decided, this PRD uses: *signed-in workspace user* (all MVP behaviour) and *authorized role* (actions `mvp.md` restricts to "authorized roles": integration management, destructive actions, link correction "if permissions allow", administrative replay/reconcile). If OQ-12 resolves to owner-only, "authorized role" = owner.

## 4. Core use cases and journeys (REQUIREMENT, `mvp.md` §4.3, §8)

| ID | Journey | Source | Primary FRs |
|---|---|---|---|
| UC-01 | First-time setup: account, workspace, connect GitHub, select repositories, historical sync, states `Connecting / Syncing / Connected / Needs attention / Disconnected` | §8.1 | FR-001, FR-002, FR-010–FR-016, FR-090 |
| UC-02 | View project health: choose repository + date window, see metrics with sample size and coverage, open contributing records, see why a metric is unavailable | §8.2 | FR-060, FR-061, FR-054 |
| UC-03 | Investigate a bottleneck: list alerts, inspect threshold and evidence, acknowledge/resolve with optional note | §8.3 | FR-080–FR-084, FR-100–FR-103 |
| UC-04 | Link a task to a PR: view links and method, confirm/change/remove, audit, recalculation | §8.4 | FR-040–FR-046, FR-120 |
| UC-05 | Recover from sync failure: detect, `Needs attention`, readable summary, retry from checkpoint, dedupe, recalculate, record result | §8.5 | FR-018, FR-028, FR-029, FR-030, FR-090 |
| UC-06 | Disconnect integration and delete data | §7.1 #17, §9.11 | FR-013, FR-113, FR-114 |

## 5. Functional requirements

Priority follows `mvp.md` §1.1: **MUST** = required for MVP; **SHOULD** = strongly recommended; **MAY** = optional. IDs are new (PROPOSED numbering); wording is from `mvp.md`.

### 5.1 Authentication and workspace (`mvp.md` §9.1)

| ID | Requirement | Pri |
|---|---|---|
| FR-001 | Secure sign-in mechanism, sign-out, and account management. Mechanism is OPEN (OQ-11). | MUST |
| FR-002 | Associate connected repositories and all derived data with a workspace; enforce workspace-level isolation on the server for every read and write. | MUST |
| FR-003 | Workspace membership with a basic owner/member role model. | SHOULD |
| FR-004 | Restrict integration management and destructive actions to authorized roles. | SHOULD |

Acceptance: changing an identifier in a request cannot return another workspace's data; every API read/write checks workspace access; automated tests cover cross-workspace attempts.

### 5.2 GitHub integration (`mvp.md` §9.2)

| ID | Requirement | Pri |
|---|---|---|
| FR-010 | Use a GitHub App or equivalently scoped secure authorization; request only permissions required by MVP features. | MUST |
| FR-011 | Let users select which repositories to connect. | MUST |
| FR-012 | Record installation/account identifiers, repository identifiers, connection status, last successful sync. | MUST |
| FR-013 | Support disconnect and handling of revoked/expired access. | MUST |
| FR-014 | Handle GitHub API pagination and rate limits. | MUST |
| FR-015 | Persist sync checkpoints so long syncs resume. | MUST |
| FR-016 | Never expose installation tokens, access tokens, webhook secrets or other credentials to the browser. | MUST |
| FR-017 | Support at least these source events: PR opened/created; PR marked ready for review (if source supports it); review submitted; review dismissed/invalidated where relevant to a metric; PR synchronized on new commits (if needed for timeline); PR closed; PR merged incl. merge timestamp; repository/integration lifecycle status needed to detect disconnection. Exact event names and fields must be checked against current GitHub docs at implementation time. | MUST |
| FR-018 | Reconcile stored state with the GitHub REST API after missed deliveries or inconsistencies; show latest successful reconciliation time. | SHOULD |
| FR-019 | Explain clearly when a permission is missing. | SHOULD |

### 5.3 Event ingestion and processing (`mvp.md` §9.3)

| ID | Requirement | Pri |
|---|---|---|
| FR-020 | Validate webhook signatures with GitHub's documented method before trusting a payload. | MUST |
| FR-021 | Persist delivery identifiers and use them for idempotency where available. | MUST |
| FR-022 | Acknowledge valid webhooks promptly; process heavier work asynchronously. | MUST |
| FR-023 | Store the source event or a sufficient immutable raw representation for debugging/replay, subject to retention and privacy rules. | MUST |
| FR-024 | Normalize source events into an internal event model that does not depend on one webhook payload shape. | MUST |
| FR-025 | Store both source occurrence time and DevFlow receipt time. | MUST |
| FR-026 | Handle duplicate deliveries safely. | MUST |
| FR-027 | Handle late and out-of-order events; never assume arrival order equals occurrence order. | MUST |
| FR-028 | Retry transient failures with bounded exponential backoff or equivalent. | MUST |
| FR-029 | Record processing failures and make them observable. | MUST |
| FR-030 | Recompute affected metrics when late events or corrections change event history. | MUST |

Must-not (REQUIREMENT): do not assume exactly-once delivery; do not treat an HTTP success on the webhook endpoint as proof downstream processing completed; do not delete previously valid data because a later API request fails.

### 5.4 Task-to-code linking (`mvp.md` §9.4)

| ID | Requirement | Pri |
|---|---|---|
| FR-040 | Explicit user-created task↔PR link. | MUST |
| FR-041 | Exact task-key match in configured PR title/body/branch text, when a task-key convention is configured. | MUST |
| FR-042 | `unmatched` state when no reliable link exists. | MUST |
| FR-043 | Store a matching-method field with every relationship. | MUST |
| FR-044 | A way to inspect and correct links. | MUST (see C-01) |
| FR-045 | Audit history for manual link changes (who, when, optional reason). | MUST |
| FR-046 | Match states: `confirmed_manual`, `matched_exact_key`, `suggested`, `unmatched`, `rejected`. Fuzzy similarity is never auto-confirmed. | MUST (state set is "recommended" in source; PROPOSED as final) |

### 5.5 Metric engine (`mvp.md` §9.5, §10)

| ID | Requirement | Pri |
|---|---|---|
| FR-050 | Store metric definitions with versions. | MUST |
| FR-051 | Calculate from normalized source events, not UI state. | MUST |
| FR-052 | Each metric defines start/end events and inclusion/exclusion rules. | MUST |
| FR-053 | Each result stores calculation time, definition version, window/cohort, sample count, data-coverage status, and event references. | MUST |
| FR-054 | Represent missing/invalid data explicitly (unavailable / incomplete / invalid / pending reconciliation); never substitute a guessed timestamp or zero. | MUST |
| FR-055 | Deterministic recomputation. | MUST |
| FR-056 | Include sample count and coverage wherever practical. | MUST |
| FR-057 | Store timestamps in UTC; render with explicit timezone label. | MUST |
| FR-058 | State for each metric whether it uses elapsed time or configured working time; never mix them in one chart without labelling. | MUST |

### 5.6 Dashboard, explorer, bottlenecks, data quality, insights, settings

| ID | Requirement | Pri | Source |
|---|---|---|---|
| FR-060 | Overview shows connected repositories and status, selected date window, metric cards with enough valid data, sample count per metric, incomplete-data indicator, short list of current alerts. | MUST | §9.6 |
| FR-061 | From any metric, user can open the contributing records and timestamps; unavailable metrics state which event or integration is missing. | MUST | §8.2, §9.6, §7.1 #14 |
| FR-062 | Show median and p75 for duration metrics when sample size is sufficient. "Sufficient" is undefined (OQ-17). | SHOULD | §9.6 |
| FR-063 | Show trend vs previous comparable window only when both windows have enough comparable data. | SHOULD | §9.6 |
| FR-064 | Data-health summary on Overview: last sync, unmatched activity, missing data. | SHOULD | §9.6 |
| FR-070 | Lifecycle explorer: ordered event timeline for a task/PR with event names and timestamps; distinguishes occurrence vs receipt time; links to source GitHub item when user is authorized; shows which events a metric used. | MUST | §9.7 |
| FR-071 | Explorer shows gaps where an expected event is missing; it never invents a step. | MUST | §9.7 |
| FR-072 | Explorer filters (repository, date window, PR state, task-link status); review rounds and state changes. | SHOULD | §9.7 |
| FR-080 | Deterministic, rule-based detection; each rule configurable or has a documented fixed default. At least one rule must ship; the first-review-wait rule is the likely first. | MUST | §9.8, §21 |
| FR-081 | Each alert shows threshold, observed value, time window, number of affected items, evidence and links to affected records. | MUST | §9.8 |
| FR-082 | Acknowledge and resolve an insight, with optional note. | MUST | §8.3, §9.8 |
| FR-083 | Record insight creation and update times. | MUST | §9.8 |
| FR-084 | Alert wording makes no unsupported causal or personal claims; distinguishes active (incomplete) intervals from completed durations. | MUST | §9.8 |
| FR-090 | Integrations & Data Quality shows: integration status; last successful sync and reconciliation; current errors in plain language; count of unmatched relevant PRs/tasks; missing required events blocking a metric; partial vs full data window; delayed/failed/awaiting-reconciliation events where known. | MUST | §9.9 |
| FR-091 | Also show match coverage over window; processing failures and retry status for authorized admins; a reconciliation action if safe. | SHOULD | §9.9 |
| FR-100 | Each insight has stable ID, rule ID/version, creation time, affected records, evidence, status ∈ {`open`,`acknowledged`,`resolved`}. | MUST | §9.10 |
| FR-101 | Status changes record actor and timestamp. | MUST | §9.10 |
| FR-102 | A resolved insight can reopen when its condition recurs; the exact behaviour must be defined (OQ-09). | MUST | §9.10 |
| FR-103 | Distinguish "condition no longer detected" from "user marked resolved". | MUST | §9.10 |
| FR-110 | Configure repository scope. | MUST | §9.11 |
| FR-111 | Configure date window and basic bottleneck thresholds. | MUST | §9.11, §7.1 #15 |
| FR-112 | Display metric definitions. | MUST | §9.11 |
| FR-113 | Integration disconnect controls. | MUST | §9.11 |
| FR-114 | A documented data retention/deletion path. | MUST | §9.11, §7.1 #17 |
| FR-115 | Configure task-key format/prefix rules. | SHOULD | §9.11 |
| FR-116 | Configure working hours (only if working-time metrics are included; OQ-06). | SHOULD | §9.11 |
| FR-117 | Configure whether bots are excluded from qualifying review events. | SHOULD | §9.11 |
| FR-120 | Audit trail for: integration connect/disconnect; repository scope changes; manual link changes; metric-definition or threshold changes; insight acknowledgement/resolution; administrative replay/reconcile. Retention and user-visible scope are OPEN (OQ-10). | MUST | §15.5 |

### 5.7 Should-have extras (`mvp.md` §7.2)

| ID | Requirement | Pri |
|---|---|---|
| FR-130 | Team-level filters: repository, date range, task status. | SHOULD |
| FR-131 | CSV export of a summary (OQ-14). | SHOULD |
| FR-132 | In-product notification on sync failure or threshold crossing (OQ-13). | SHOULD |
| FR-133 | Lightweight event replay / admin reconciliation action. | SHOULD |

## 6. Metric and rule requirements

### 6.1 MVP metrics (`mvp.md` §10)

| ID | Metric | Status in MVP | Definition status |
|---|---|---|---|
| MET-01 | PR first-review wait | Core (§22 recommended default) | Proposed definition in §10.2A; unresolved items OQ-03, OQ-04 |
| MET-02 | PR time to merge | Core (§22 recommended default) | Proposed definition in §10.2B; cohort default OQ-05 |
| MET-03 | Task lead time | Only if the task source supplies reliable state transitions; otherwise **deferred** | Depends on OQ-01 |
| MET-04 | Coding-to-PR time | Optional; only with a defensible start event | Not committed |
| MET-05 | WIP (tasks in configured in-progress states **or** open PRs, labelled separately) | Possible | Population choice open |
| — | PR merge-to-production time; deployment frequency | **Not in MVP** (no deployment source). Must display as unavailable, never estimated from merges. | — |

Definitions summarised (elapsed time by default):

- **MET-01** start = PR ready-for-review timestamp (PR creation time if never draft); end = first qualifying human review submission after start. Status: completed / active (still waiting) / unavailable (required timestamps missing).
- **MET-02** start = same as MET-01; end = merge timestamp. Population = merged PRs in a visible cohort. Open PRs are `active`, not completed.

Edge-case rules from the source: draft time does not accrue wait; a review before ready does not end the interval; multiple reviewers do not create multiple completions; dismissed review does not automatically count; bots excluded by default where identity is reliable; comment-only issue/PR comments do not qualify (recommended default: a submitted formal PR review qualifies).

### 6.2 Metrics to avoid (REQUIREMENT, `mvp.md` §10.3)

Developer productivity score; lines of code as contribution proxy; commits-per-developer ranking; AI-estimated effort as ground truth; DORA metrics from proxy events; a single universal "cycle time" mixing tasks, PRs and deployments.

### 6.3 Bottleneck rules (`mvp.md` §9.8, §11)

| ID | Rule | MVP status |
|---|---|---|
| RULE-01 | Long review wait: open PR awaiting first qualifying human review beyond configured threshold | Likely first rule; minimum to ship |
| RULE-02 | Large review queue: count of PRs awaiting first review exceeds threshold | Candidate |
| RULE-03 | Long open-PR age beyond threshold, state labelled | Candidate |
| RULE-04 | High stage delay above team-defined threshold for sufficient sample | Candidate |
| RULE-05 | Data-link coverage warning: meaningful share of PRs unlinked | Candidate |

Prioritization of RULE-02–05 awaits pilot results (§9.8). Default thresholds are product assumptions (OQ-09), not industry standards.

## 7. Business rules and constraints

| ID | Rule | Source |
|---|---|---|
| BR-01 | Missing data is never displayed or stored as a zero duration. | §5.2, §9.5 |
| BR-02 | An open PR's current age is never reported as a completed lead time. | §9.5 |
| BR-03 | A merge is not proof of production deployment; deployment metrics are unavailable without a deployment source. | §10.2E–F, §13 |
| BR-04 | Fuzzy text similarity is never a confirmed link; ambiguous keys give `suggested` or `unmatched`. | §9.4, §14 |
| BR-05 | Metrics describe team workflow, not individual contribution; no rankings or scorecards. | §5.9, §15.4 |
| BR-06 | Elapsed and working time are never mixed unlabelled. | §9.5, §10.1 |
| BR-07 | Stage percentages are shown only for non-overlapping partitions; otherwise show durations separately. | §11.4 |
| BR-08 | Alerts make no causal claims unless data proves them. | §9.8, §11.3 |
| BR-09 | A partial sync is always labelled partial; a failed sync never silently erases last known valid data. | §8.1, §14 |
| BR-10 | Durations are non-negative; an invalid sequence is marked invalid/pending reconciliation, not clipped. | §10.1 |
| BR-11 | When sources disagree, preserve evidence and mark for reconciliation. | §13 |
| BR-12 | Every chart states its cohort. Comparisons only across comparable cohorts and coverage. | §10.5, §11.3 |
| BR-13 | Stable source identifiers, not mutable names, identify repositories, users, PRs and tasks. | §9.3, §14 |
| BR-14 | The product does not claim novelty or a specific cycle-time improvement without measured pilot evidence. | §2.2, §6.3 |

## 8. Scope

**Included (MUST)** — `mvp.md` §7.1: workspace + authentication; secure GitHub authorization; repository selection and status; resumable historical sync; webhook processing; raw + normalized event storage; idempotency/retries; explicit linking; conservative task-key matching; core metrics with published definitions; lifecycle explorer; basic dashboard and alerts; data-quality page; metric evidence; time-window and threshold configuration; automated tests (definitions, duplicates, ordering, missing data); disconnect/revoke and a basic data deletion policy.

**Should have** — FR-130 to FR-133 and the SHOULD items above.

**Explicitly excluded / later** — GitLab, Bitbucket, Jira, Linear and other integrations; deployment metrics without a provider; full DORA; AI summaries or recommendations; forecasting; automated issue creation; complex org roles and enterprise SSO; custom formula builder; individual scorecards/leaderboards; automatic complexity estimation; deep code-quality/security/coverage analytics; native mobile app. DevFlow also does not replace project management, GitHub, CI/CD or deployment tooling, and does not use generative AI as the source of truth for metrics.

## 9. Acceptance criteria

Requirement-level criteria are in §5 and §6. Cross-cutting criteria:

| ID | Criterion |
|---|---|
| AC-01 | Re-delivering the same webhook (same delivery ID) leaves event, metric and insight counts unchanged. |
| AC-02 | Delivering events out of order produces the same timeline and metric as delivering them in occurrence order. |
| AC-03 | A late event changes affected metric results and records the recomputation. |
| AC-04 | Every fixture in `mvp.md` §18.1 has documented input events, expected state/value and rationale, and passes. |
| AC-05 | No displayed duration is negative; no missing timestamp renders as `0`. |
| AC-06 | Partial or failed sync is labelled on Overview and Integrations & Data Quality; "fully synced" appears only after the sync job completes. |
| AC-07 | After disconnect, no further GitHub API calls are made for that installation. |
| AC-08 | The edge-case table in `mvp.md` §14 is converted into a test matrix (one test per row) before Phase 5 exit. |
| AC-09 | The 10 invariants in `mvp.md` §18.5 each map to at least one automated test. |

## 10. Non-functional requirements

| ID | Requirement | Source / label |
|---|---|---|
| NFR-001 | Server-side workspace isolation on every query and mutation; tested against ID substitution. | §15.2 REQUIREMENT |
| NFR-002 | Idempotent ingestion keyed on delivery ID / deduplication key. | §5.5, §9.3 REQUIREMENT |
| NFR-003 | Deterministic metric computation for fixed event set + definition version. | §5.6 REQUIREMENT |
| NFR-004 | All timestamps stored in UTC; displayed with explicit timezone. | §10.1 REQUIREMENT |
| NFR-005 | Credentials never in browser, API responses or logs. | §9.2, §15.1 REQUIREMENT |
| NFR-006 | HTTPS in deployed environments. | §15.1 REQUIREMENT |
| NFR-007 | Least-privilege GitHub permissions. | §15.1 REQUIREMENT |
| NFR-008 | Data minimization: store only needed GitHub metadata; do not download repository source code; no private message content. | §15.3 REQUIREMENT |
| NFR-009 | Structured logs, processing errors, sync status and basic operational alerts. | §16 item 7 REQUIREMENT (direction) |
| NFR-010 | Every metric and alert traceable to source events and a definition/rule version. | §5.1 REQUIREMENT |
| NFR-011 | Webhook endpoint acknowledges promptly; no numeric latency target is defined. | §9.3; number is OPEN |
| NFR-012 | Retention and deletion behaviour defined before onboarding real pilot users. | §15.3 REQUIREMENT |
| NFR-013 | Accessibility: WCAG 2.2 AA target for the web UI, keyboard operability, visible focus, reduced-motion support. | **PROPOSED.** `mvp.md` does not state accessibility requirements; this comes from the documentation task brief. Needs approval. |

## 11. Conflicts and gaps found in `mvp.md`

| ID | Finding | Handling |
|---|---|---|
| C-01 | Manual link **correction** is SHOULD in §7.2 but §9.4 says MUST "a way to inspect and correct links" and §8.4/§21 require auditable corrections. | This PRD keeps FR-044/FR-045 as MUST (stricter reading; the narrower reading would break the §21 DoD item). OPEN: confirm. |
| C-02 | §9.8 lists five candidate rules; §22 and §21 recommend/require one. | At least RULE-01 is MUST; others candidate. |
| C-03 | §7.1 #15 requires configuring "time window and alert thresholds"; §9.8 allows "configurable **or** documented fixed default". | FR-111 follows §7.1 (configurable thresholds); a fixed default alone does not satisfy it. |
| C-04 | §9.1 role model is SHOULD, yet §8.4 and §9.2 restrict actions to "authorized users/roles". | Roles term kept abstract until OQ-12. |
| C-05 | §17 describes an "Engineering Analytics" page (§17.3) that is not in the §7.1 Must list. | Treated as candidate/proposed screen S-05 in design.md; OQ-18. |
| C-06 | Task lead time (MET-03) and the Tasks screen depend on the unresolved task source (OQ-01); GitHub Issues open/close does not supply a true start event. | MET-03 deferred unless task source provides transitions. |
| C-07 | Reopen behaviour for resolved insights and reopened PRs is required but undefined. | OQ-09, OQ-19. |
| C-08 | §9.6 requires "enough valid data" and "sufficient sample size" but never defines them. | OQ-17. |
| C-09 | No sign-in UI or workspace-creation screen is described in §17, though §8.1 step 1 requires them. | Minimal screens proposed in design.md. |

## 12. Dependencies, assumptions, open questions, risks

**Dependencies**: GitHub App/API and webhook availability and current documentation (links in `mvp.md` §23, not re-verified here); a hosting environment that supports secure environment variables, webhook endpoints, background jobs and durable storage.

**Assumptions** (ASSUMPTION): A-01 first pilot is one or a few consenting small teams; A-02 GitHub is the only source for MVP; A-03 all MVP numeric thresholds are placeholders until pilot review.

**Open questions** (numbering follows `mvp.md` §22 where possible):

| ID | Question | Blocks |
|---|---|---|
| OQ-01 | Task source: GitHub Issues (Path A), native lightweight tasks (Path B), or one external tracker (Path C)? `mvp.md` recommends A or B. | Tasks screen, MET-03, data model, Phase 3 |
| OQ-02 | Is first-review wait the primary metric, or task lifecycle duration? (Recommended default: first-review wait.) | Overview layout |
| OQ-03 | Exact "ready-for-review" start event for draft and non-draft PRs | MET-01, MET-02 |
| OQ-04 | Which review states qualify as first review; treatment of dismissed reviews and bots | MET-01 |
| OQ-05 | Default cohort per metric (created / merged / active in window) | MET-01, MET-02 UI labels |
| OQ-06 | Elapsed-time only, or working-hours calculation in MVP? | FR-058, FR-116 |
| OQ-07 | History depth for first sync; user-configurable? | Phase 1 |
| OQ-08 | Who may create/correct links; is a reason mandatory? | FR-044, FR-045 |
| OQ-09 | Initial thresholds, severities, dedup windows, reopening rules | RULE-01, FR-102 |
| OQ-10 | Retention of raw payloads, normalized events, audit records, derived results | FR-114, NFR-012 |
| OQ-11 | Framework, database, job system, hosting, authentication mechanism | architecture.md, Phase 1 |
| OQ-12 | Owner-only or owner/member roles | FR-003, FR-004 |
| OQ-13 | In-product alerts only, or email/Slack notifications | FR-132 |
| OQ-14 | CSV export required for first pilot? | FR-131 |
| OQ-15 | Pilot definition: who, which data, acceptance criteria | SC-10, Phase 5 |
| OQ-16 | Unresolved: confirm C-01 (manual link correction MUST vs SHOULD) | FR-044 |
| OQ-17 | Define "sufficient sample size" and "enough valid data" | FR-062, FR-063, FR-060 |
| OQ-18 | Is the Engineering Analytics page (§17.3) in the MVP? | design S-05 |
| OQ-19 | Is a reopened PR a new interval for MET-01/MET-02? | MET-01, MET-02 |
| OQ-20 | Do pilot data subjects trigger any privacy-law obligations? No compliance obligation is asserted anywhere in these docs. | security.md |

**Risks** (from `mvp.md` §20; mitigations summarised, details in [security.md](security.md) and [phases.md](phases.md)):

| ID | Risk | Mitigation |
|---|---|---|
| RISK-01 | Metric ambiguity | Versioned published definitions; fixtures first |
| RISK-02 | Missing/delayed events | Reconciliation, coverage status, explicit incomplete states |
| RISK-03 | Duplicate webhooks | Delivery-level idempotency, unique constraints |
| RISK-04 | Incorrect task matching | Exact-key/manual only; suggestions unconfirmed |
| RISK-05 | GitHub API limits | Pagination, backoff, checkpoints, bounded concurrency |
| RISK-06 | Draft/review complexity | Formal event semantics + fixtures |
| RISK-07 | Overclaiming deployment metrics | Defer until deployment source exists |
| RISK-08 | Alert fatigue | Configurable thresholds, dedup, pilot feedback |
| RISK-09 | Misuse as employee surveillance | Team-level framing; BR-05; wording rules (FR-084) |
| RISK-10 | Scope expansion | One provider, one task-source path, small metric set |
| RISK-11 | Data/security breach | Least privilege, isolation, secret protection, retention controls |
| RISK-12 | Misleading comparisons | Visible cohorts and sample sizes (BR-12) |
| RISK-13 | Recalculation changes old values | Definition versions, evidence, recalculation history |
