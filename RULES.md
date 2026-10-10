# DevFlow — Project Rules and Engineering Conventions

| Field | Value |
|---|---|
| Status | Draft v0.1. **Every rule is PROPOSED** unless it cites a `mvp.md` requirement, in which case it is marked REQ. |
| Stack evidence | None. No repository, language or framework was available, so rules are stack-neutral. Add stack-specific rules (formatter, linter, folder layout) after the stack decision (OQ-11) and mark them VERIFIED only when a config file proves them. |
| Related | [prd.md](prd.md) · [architecture.md](architecture.md) · [design.md](design.md) · [memory.md](memory.md) · [security.md](security.md) · [phases.md](phases.md) |

Rule IDs: `RL-nn`. Keywords MUST / MUST NOT / SHOULD are used as in `mvp.md` §1.1. Exceptions require a written entry in §12.

## 1. Code organization and responsibilities

| ID | Rule | Basis |
|---|---|---|
| RL-01 | Keep these boundaries as separate modules (directories or packages): web UI, application API, GitHub integration layer, event-processing worker, metric engine, rules engine, persistence. Exact paths follow the chosen stack. | architecture.md §4 |
| RL-02 | The metric engine and rules engine MUST be pure: inputs are normalized events/records, a definition or rule version, and an explicit evaluation time. They MUST NOT read the clock, network, database, or UI state. | NFR-003, AD-09 |
| RL-03 | UI code MUST NOT compute metrics, durations, coverage, or alert conditions. It renders results produced by the engines. | FR-051 |
| RL-04 | Code that depends on GitHub payload shape MUST live only in the GitHub integration layer. Everything downstream consumes the normalized event model. | FR-024 |
| RL-05 | Business wording (metric labels, alert templates, status labels) lives in one place per kind (a single module or resource file), not inline in components, so [design.md](design.md) §2 forbidden-language checks can scan it. | DAC-07 |

## 2. Naming and terminology

| ID | Rule |
|---|---|
| RL-06 | Use the exact `mvp.md` vocabulary in code, API and UI: connection states `Connecting/Syncing/Connected/Needs attention/Disconnected`; insight states `open/acknowledged/resolved`; match states `confirmed_manual/matched_exact_key/suggested/unmatched/rejected`. Represent them as closed enumerations, never free strings. |
| RL-07 | Use stable IDs: `FR-`, `NFR-`, `BR-`, `OQ-`, `AD-`, `RL-`, `SEC-` (controls), `T-` (threats), `SR-` (security risks), `PH-` (phases), `DG-`/`DAC-`/`D-` (design). IDs are never reused or renumbered; retire by marking "Withdrawn". |
| RL-08 | Names state the unit and basis: `waitElapsedSeconds`, not `wait`. Never use the unqualified name `cycleTime` (REQ §10.3). Metric identifiers are `MET-nn` with a definition version. |
| RL-09 | Timestamps are named by meaning (`occurredAt`, `receivedAt`, `calculatedAt`) and stored in UTC. |

## 3. Type safety and error handling

| ID | Rule | Basis |
|---|---|---|
| RL-10 | Use the strictest static typing the chosen language offers (e.g., strict mode if TypeScript is chosen). No untyped escape hatches at module boundaries. | PROPOSED |
| RL-11 | Validate all external input at the boundary (webhook payloads, API requests, GitHub API responses, settings forms) with a runtime schema; reject or quarantine invalid input; never trust a type assertion. | security.md §6 |
| RL-12 | A duration result is a tagged union (`completed(value)`, `active(elapsedSoFar)`, `unavailable(reason)`, `invalid`, `pendingReconciliation`), not a nullable number. `0` MUST NOT be used to represent absence. | BR-01, FR-054 |
| RL-13 | Negative durations MUST be returned as `invalid`/`pendingReconciliation`; they MUST NOT be clamped. | BR-10 |
| RL-14 | Errors surfaced to users are plain-language messages with a stable error code; stack traces, SQL, tokens and payload fragments never reach the client or logs. | NFR-005 |
| RL-15 | Do not swallow errors. Every caught error is either handled with a defined outcome, recorded as a processing failure (FR-029), or rethrown. | FR-029 |
| RL-16 | A failed upstream call MUST NOT delete or overwrite previously valid data. Writes after a fetch are all-or-nothing per record set. | REQ §9.3 |

## 4. API and data-access conventions

| ID | Rule | Basis |
|---|---|---|
| RL-17 | Every data-access function for a workspace-owned entity takes the workspace identity from the authenticated context and applies it in the query. Client-supplied workspace or record IDs alone never authorize access. Provide no unscoped repository function for workspace-owned tables. | NFR-001 |
| RL-18 | Every API handler calls the central authorization function (user, workspace, action) before reading or writing. | architecture.md §7 |
| RL-19 | Inserts of source-derived records and events are idempotent: use a unique constraint plus upsert or conflict-ignore; never "check then insert". | NFR-002 |
| RL-20 | Webhook handler order is fixed: read raw body → verify signature → persist raw delivery → enqueue → respond. No parsing before verification. | FR-020, FR-022 |
| RL-21 | Audited actions (link changes, threshold/definition changes, insight status changes, integration changes, admin replay/reconcile) write their audit record in the same transaction as the change. | FR-120 |
| RL-22 | Identity of repositories, PRs, users and tasks is the stable source ID. Display names and titles are never keys or join conditions. | BR-13 |
| RL-23 | Schema changes are made by versioned migrations committed with the code that needs them. | PROPOSED |
| RL-24 | Metric definition and rule changes create a new version; existing results keep their version. Never edit a published version in place. | FR-050, RISK-13 |

## 5. Dependencies

| ID | Rule |
|---|---|
| RL-25 | Add a dependency only with a written reason in the pull request (what it replaces, why not standard library, maintenance status). `mvp.md` §16.1 names candidate libraries; none is approved until OQ-11 closes. |
| RL-26 | Lockfile committed; installs use the lockfile; dependency versions are exact or lockfile-pinned. |
| RL-27 | Dependencies run automated vulnerability checks in CI and on a schedule once CI exists; critical findings block release. (security.md §10) |
| RL-28 | Third-party scripts or fonts are not loaded from external CDNs in the application UI (system font stack is specified in [design.md](design.md) §4.3). |

## 6. Testing

| ID | Rule | Basis |
|---|---|---|
| RL-29 | Ground-truth fixtures (REQ §18.1) are written and reviewed **before** the metric engine implementation. Each fixture records input events, expected state/value and rationale. | phases.md PH-0 |
| RL-30 | Every metric definition has unit tests for start/end selection, draft behaviour, qualifying-review logic, bot exclusion, dismissal, invalid duration, cohort membership, missing-data states and versioning (REQ §18.2). | AC-04 |
| RL-31 | Every invariant in REQ §18.5 maps to at least one automated test; the mapping lives in the test plan. | AC-09 |
| RL-32 | Each row of the `mvp.md` §14 edge-case table becomes one named test. | AC-08 |
| RL-33 | Integration tests cover signature validation (valid, invalid, missing), duplicate delivery, pagination, rate-limit backoff, retry, reconciliation, workspace authorization, link correction with audit, recomputation. | REQ §18.3 |
| RL-34 | An end-to-end test exercises the ten-step journey in `mvp.md` §18.4 against a test GitHub installation or recorded fixtures. | REQ §18.4 |
| RL-35 | A bug fix includes a regression test that fails before the fix. | PROPOSED |
| RL-36 | Tests do not call live GitHub in CI by default; they use recorded or constructed payloads. A separate, explicitly invoked suite may use a test installation. | PROPOSED |
| RL-37 | Time-dependent tests inject the clock. | RL-02 |

## 7. Accessibility and UI

| ID | Rule |
|---|---|
| RL-38 | UI follows [design.md](design.md): tokens from §4, vocabulary from §5, formatting from §4.5. Do not introduce colors, font sizes or spacing outside the token set; add the token to design.md first. |
| RL-39 | Interactive elements use native semantics where possible; custom widgets implement the matching ARIA pattern with keyboard support (design.md §12). |
| RL-40 | A UI change that touches a status, metric, or alert includes its states (loading, empty, error, partial, unavailable) in the same change. |
| RL-41 | Automated accessibility checks and the forbidden-language string check (DAC-07) run in CI when CI exists. |

## 8. Code quality and review

| ID | Rule |
|---|---|
| RL-42 | Pull requests are small and single-purpose and state which requirement IDs they implement. |
| RL-43 | At least one reviewer other than the author approves before merge. For changes to the metric engine, rules engine, authorization, signature verification, or secret handling, the reviewer MUST confirm the matching tests and [security.md](security.md) controls are covered. |
| RL-44 | Formatter and linter configuration is committed and enforced in CI; the specific tools are chosen with the stack. |
| RL-45 | No commented-out code, no `TODO` without an issue reference, no unexplained magic numbers; thresholds and windows come from settings or named constants documented in [prd.md](prd.md). |
| RL-46 | Comparisons that would rank or score individuals MUST NOT be implemented (BR-05). Per-person aggregates are not computed or returned by any API in the MVP. |

## 9. Logging and secrets

| ID | Rule | Basis |
|---|---|---|
| RL-47 | Logs are structured and contain identifiers (workspace, integration, delivery ID, event ID), never tokens, private keys, webhook secrets, cookies, or raw payload bodies. | NFR-005 |
| RL-48 | Secrets are read from the server-side secret mechanism at runtime; never committed, never bundled into client code; an example env file lists variable names only. | NFR-005 |
| RL-49 | Pre-commit or CI secret scanning is enabled once repository tooling exists. | security.md §7 |

## 10. Documentation conventions

| ID | Rule |
|---|---|
| RL-50 | The seven documents under `docs/` are the project record. A change to behaviour, scope, a decision or a metric definition updates the owning document in the same change. Ownership: prd (what) · architecture (how) · design (UX) · rules (conventions) · security (controls) · phases (order) · memory (context). |
| RL-51 | Use labels **VERIFIED / REQUIREMENT / PROPOSED / ASSUMPTION / OPEN QUESTION**. A PROPOSED item becomes a decision only by recording who approved it and when in [memory.md](memory.md). Never mark an item VERIFIED without a repository path or test that proves it. |
| RL-52 | Reference other documents by ID instead of copying their text. |
| RL-53 | `mvp.md` stays unmodified except for a recorded, justified change. |
| RL-54 | Metric definitions are documented in the user-visible Settings → Metric definitions content and in prd.md; the two must agree. |

## 11. Prohibited practices

- Showing missing data as zero, or imputing a timestamp (BR-01).
- Presenting an open PR's age as completed lead time (BR-02).
- Estimating deployment metrics from merges (BR-03).
- Auto-confirming a fuzzy task match (BR-04).
- Individual scores, rankings, leaderboards, productivity or contribution metrics (BR-05, REQ §10.3).
- Using generative AI output as the source of truth for any metric (REQ §3.3).
- Assuming webhook order or exactly-once delivery (REQ §9.3).
- Trusting a webhook before signature verification.
- Client-side-only authorization.
- Logging credentials or payload bodies.
- Silent definition edits (RL-24).
- Adding features, integrations or dependencies outside [prd.md](prd.md) scope without an approved scope change.

## 12. Exceptions

Exceptions are recorded in this table with rule ID, justification, approver, date, and expiry or review condition. None recorded.

| Rule | Justification | Approver | Date | Review condition |
|---|---|---|---|---|
| — | — | — | — | — |
