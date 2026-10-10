# DevFlow — Implementation Roadmap and Delivery Phases

| Field | Value |
|---|---|
| Status | Draft v0.1 |
| Source | `mvp.md` §19 (suggested sequencing, "not calendar commitments"), §21, §22; requirement IDs from [prd.md](prd.md) |
| Estimates | **None.** `mvp.md` provides no team size, capacity, or dates; none are invented. Sequence is by dependency and risk only. |
| Repository state | No repository was provided; "not started" is assumed for all phases (ASSUMPTION; correct this table when code exists). |
| Related | [architecture.md](architecture.md) · [design.md](design.md) · [rules.md](rules.md) · [memory.md](memory.md) · [security.md](security.md) |

Phase IDs `PH-0` … `PH-5` follow `mvp.md` §19. Task IDs `TSK-<phase>.<n>`.

## Dependency overview

```mermaid
flowchart LR
  PH0[PH-0 Decisions + fixtures] --> PH1[PH-1 Foundations + GitHub connection]
  PH1 --> PH2[PH-2 Event model + reliability]
  PH2 --> PH3[PH-3 Linking + metrics]
  PH3 --> PH4[PH-4 Dashboard + bottlenecks]
  PH4 --> PH5[PH-5 Pilot hardening]
```

Why this order: metrics are only trustworthy on a reliable normalized timeline (PH-2 before PH-3); alerts and dashboards must display verified metrics (PH-3 before PH-4); the security, isolation and recovery verification (PH-5) needs the full system. Security controls and tests accompany each phase; PH-5 is the pre-pilot gate, not the first time security is considered.

| Phase | Status |
|---|---|
| PH-0 | Not started (ASSUMPTION) |
| PH-1 – PH-5 | Not started (ASSUMPTION) |

## PH-0 — Resolve product choices and write fixtures

**Objective.** Close decisions that otherwise change the data model or metric semantics, and create the ground truth before any metric code.

| Item | Detail |
|---|---|
| Requirement IDs | Defines FR-050, FR-052, FR-058, FR-046, FR-080 inputs; no runtime features |
| Decisions required (blocking) | OQ-01 task source · OQ-03 ready-for-review start · OQ-04 qualifying review, dismissed reviews, bots · OQ-05 default cohorts · OQ-06 elapsed vs working time · OQ-10 retention/deletion · OQ-11 stack, database, jobs, hosting, authentication · OQ-12 roles · OQ-19 reopened-PR semantics · OQ-16 manual-correction priority |
| Decisions required (non-blocking, resolve before PH-4) | OQ-02 primary metric · OQ-07 history depth · OQ-08 link permissions · OQ-09 thresholds, severities, dedup, reopen · OQ-13 notifications · OQ-14 CSV · OQ-17 sufficient sample size · OQ-18 Analytics screen · OQ-20 privacy applicability · design D-05 default window |
| Prerequisites | `mvp.md` accepted as source; owner for each decision named |

Tasks and outputs:

| ID | Task | Output |
|---|---|---|
| TSK-0.1 | Decide each blocking OQ; record decision, approver, date in [memory.md](memory.md) | Updated memory.md decision log |
| TSK-0.2 | Finalize MET-01 and MET-02 definitions, cohorts, edge-case rules, version 1 | Metric definition documents (feeds FR-112) |
| TSK-0.3 | Write ground-truth fixtures for every MVP metric: simple PR with one qualifying review; draft later marked ready; multiple review rounds then merge; review before ready; bot review then human review; dismissed review; open PR still waiting; missing ready timestamp; duplicate webhook; out-of-order events; late event needing recompute; task link added after merge; partial sync then reconciliation; PR merged with no deployment event. Each has inputs, expected state/value, rationale. | Reviewed fixture set (REQ §18.1) |
| TSK-0.4 | Choose minimal GitHub App permission set and webhook subscriptions against current GitHub docs | Permission list (resolves SQ-01, D-11) |
| TSK-0.5 | Draft retention/deletion policy | Policy document (SEC-70, FR-114) |
| TSK-0.6 | Convert `mvp.md` §14 edge-case table into a test-matrix skeleton | Matrix (AC-08) |
| TSK-0.7 | Re-run repository inspection once a repository exists; update VERIFIED sections in architecture.md, rules.md, design.md | Reconciled docs |

Testing and verification: fixtures reviewed by someone other than the author; every fixture has an unambiguous expected result.

**Definition of done / exit criteria (REQ §19):** product choices and metric definitions are documented and reviewable; fixtures exist; stack and authentication decided.

Risks/blockers: unresolved OQ-01 blocks the data model; unresolved OQ-11 blocks all implementation; RISK-01 (metric ambiguity).

Deferred from PH-0: non-blocking decisions listed above.

## PH-1 — Foundations and GitHub connection

**Objective.** A test repository can be connected and imported without exposing credentials.

| Item | Detail |
|---|---|
| Requirement IDs | FR-001, FR-002, FR-003*, FR-004*, FR-010, FR-011, FR-012, FR-013, FR-014, FR-015, FR-016, FR-019*, FR-110, FR-113; basic FR-020 (endpoint + signature validation); NFR-001, NFR-005, NFR-006, NFR-007, NFR-008 (*SHOULD) |
| Design | S-01, S-02, S-08 (status, scope, disconnect), I-01, I-02, I-07 |
| Security | SEC-01–SEC-05, SEC-07–SEC-09, SEC-10, SEC-13–SEC-15, SEC-18, SEC-20–SEC-25, SEC-75 |
| Prerequisites | PH-0 exit: stack, auth mechanism, permission set, task-source path for later phases |

| ID | Task | Output |
|---|---|---|
| TSK-1.1 | Project skeleton, CI, lint/format, secret scanning, env example (names only) | Repository tooling per [rules.md](rules.md) |
| TSK-1.2 | Authentication, sign-out, workspace creation, central authorization function with workspace scoping | S-01, S-02 step 1; SV-01 tests (cross-workspace) for existing routes |
| TSK-1.3 | GitHub App registration and authorization/installation flow with `state` validation | S-02 steps 2–3 |
| TSK-1.4 | Integration and Repository persistence; status model (`Connecting` … `Disconnected`); repository selection | FR-011, FR-012 |
| TSK-1.5 | GitHub API client with pagination, rate-limit handling, installation-token handling server-side only | FR-014, FR-016 |
| TSK-1.6 | Historical sync job with persisted checkpoints | FR-015 |
| TSK-1.7 | Webhook endpoint: raw-body signature verification, persist, enqueue, respond | FR-020 basics |
| TSK-1.8 | Disconnect and revoked-access handling | FR-013, SEC-15 |
| TSK-1.9 | S-08 basic status page | FR-090 subset |

Testing: signature tests (SV-02), cross-workspace tests (SV-01), callback tests (SV-05), revocation test (SV-08), pagination and rate-limit tests with constructed responses, secret-exposure scan (SV-04).

**Definition of done:** a test repository can be connected and imported; no credentials appear in browser, responses or logs; disconnect stops API use; cross-workspace tests pass. **Exit criteria:** REQ §19 Phase 1 exit; SR-01–SR-04 mitigations verified.

Risks/blockers: GitHub App approval/permissions (RISK-05, RISK-11); authentication choice; hosting for webhook endpoints.

Deferred: roles beyond what OQ-12 decides; reconciliation (PH-2).

## PH-2 — Event model and reliability

**Objective.** Replaying a fixture and delivering duplicates/out-of-order events produces a correct normalized timeline.

| Item | Detail |
|---|---|
| Requirement IDs | FR-017, FR-018, FR-021–FR-030, FR-133*; NFR-002, NFR-004, NFR-009; backend for FR-090, FR-091* |
| Security | SEC-11, SEC-12, SEC-34, SEC-40–SEC-45, SEC-52–SEC-54, SEC-71, SEC-72 |
| Prerequisites | PH-1 complete; fixtures from PH-0; retention decision for raw payloads (OQ-10) |

| ID | Task | Output |
|---|---|---|
| TSK-2.1 | Normalized event schema (fields per `mvp.md` §9.3, refined) and raw-payload storage | Schema + migration |
| TSK-2.2 | Idempotent insert with delivery ID and `deduplication_key` under unique constraint | FR-021, FR-026 |
| TSK-2.3 | Worker with bounded exponential backoff, failure recording, processing status | FR-028, FR-029 |
| TSK-2.4 | Event ordering by occurrence time and semantics; late-event detection | FR-027 |
| TSK-2.5 | Event-type coverage for the supported set (PR created/ready/synchronized/closed/merged, review submitted/dismissed, integration lifecycle) | FR-017 |
| TSK-2.6 | Backfill/webhook overlap handling (AD-06) | Overlap tests |
| TSK-2.7 | Initial reconciliation job: fetch, compare, upsert, record outcome, safe to rerun | FR-018 |
| TSK-2.8 | Structured logging and operational status for sync/failures | NFR-009 |
| TSK-2.9 | Admin replay/reconcile action if built | FR-133 |

Testing: duplicate delivery, out-of-order, late event, retry, backfill-overlap, reconciliation idempotency, failed-upstream-does-not-delete tests (RL-16); fixtures run end to end through ingestion.

**Definition of done:** AC-01, AC-02 pass for all supported event types; failures observable; reruns safe. **Exit criteria:** REQ §19 Phase 2 exit; SR-03 and SR-15 verified at this level.

Risks/blockers: GitHub payload field availability for "ready for review" (OQ-03); RISK-02, RISK-03.

Deferred: metric recomputation hooks beyond the event trigger (PH-3).

## PH-3 — Linking and metrics

**Objective.** Approved fixtures calculate exactly as expected, and every result is inspectable.

| Item | Detail |
|---|---|
| Requirement IDs | FR-040–FR-046, FR-050–FR-058, FR-030 (recompute), FR-115*, FR-116* (only if OQ-06 includes working time), FR-117*, FR-120 (link, definition, threshold audit); MET-01, MET-02 (MET-03 only if task source provides transitions) |
| Design | S-04 (conditional), S-10 data, I-05; design.md §4.5 formatting |
| Prerequisites | PH-2 exit; OQ-01, OQ-03–OQ-06, OQ-08, OQ-19 resolved; fixtures approved |

| ID | Task | Output |
|---|---|---|
| TSK-3.1 | Task entity per chosen path (A: issues distinguished from PRs; B: minimal native tasks) | FR-040 foundations |
| TSK-3.2 | Link model: explicit link, exact-key match (when convention configured), `unmatched`, matching method, match status | FR-040–FR-043, FR-046 |
| TSK-3.3 | Manual confirm/change/remove with audit | FR-044, FR-045 |
| TSK-3.4 | Metric definition storage and versioning | FR-050 |
| TSK-3.5 | Pure metric engine for MET-01 and MET-02 with result tagged union (completed/active/unavailable/invalid/pending) | FR-051–FR-056, RL-12, RL-13 |
| TSK-3.6 | Result persistence with definition version, cohort, sample count, coverage, event references | FR-053 |
| TSK-3.7 | Recompute on late events and on link changes | FR-030 |
| TSK-3.8 | Metric evidence API/data for S-10 | FR-061 backing |
| TSK-3.9 | Full ground-truth and unit-test suites | RL-29–RL-32 |
| TSK-3.10 | Optional: task-key convention setting, bot exclusion setting, working-time calendar | FR-115–FR-117 |

Testing: all fixtures from TSK-0.3; unit tests per RL-30; invariants (determinism, never-negative, missing never zero) from REQ §18.5; link-after-merge recompute test.

**Definition of done:** SC-01, SC-03, SC-05, SC-07 satisfied; DAC-01 data basis exists. **Exit criteria:** REQ §19 Phase 3 exit — approved fixtures calculate exactly as expected and each result lists its evidence.

Risks/blockers: OQ-01/03/04 ambiguity (RISK-01, RISK-04, RISK-06); recomputation cost (architecture §13).

Deferred: MET-03 unless transitions exist; MET-04, MET-05 unless explicitly approved; deployment metrics (not in MVP).

## PH-4 — Dashboard and bottlenecks

**Objective.** A pilot user can identify a delayed PR and understand the evidence behind the alert.

| Item | Detail |
|---|---|
| Requirement IDs | FR-060, FR-061, FR-062*, FR-063*, FR-064*, FR-070, FR-071, FR-072*, FR-080–FR-084, FR-090, FR-091*, FR-100–FR-103, FR-111, FR-112, FR-114, FR-120 (insight and threshold audit); SHOULD: FR-130, FR-131, FR-132 |
| Design | S-03, S-06, S-07, S-08 (complete), S-09, S-10 UI, S-05 (only if OQ-18 = in MVP); I-03, I-04, I-06, I-08, I-09, I-10 |
| Prerequisites | PH-3 exit; OQ-09, OQ-17 resolved; design D-05, D-08, D-09 decided |

| ID | Task | Output |
|---|---|---|
| TSK-4.1 | Application shell, navigation, scope selectors persisted in URL, tokens and shared components from design.md §4–§6 | DAC-14, DAC-12 base |
| TSK-4.2 | S-03 Overview with MetricCards and completeness notices | DAC-02, DAC-03, DAC-04 |
| TSK-4.3 | S-10 evidence table with exclusions section | FR-061 |
| TSK-4.4 | S-07 lifecycle explorer with gap and late-event display | FR-070, FR-071 |
| TSK-4.5 | Rules engine and RULE-01 (minimum); other rules only if approved after review of OQ-09 | FR-080, FR-081 |
| TSK-4.6 | Insight lifecycle with statuses, history, auto-clear vs user-resolved, reopen policy | FR-100–FR-103 |
| TSK-4.7 | S-06 Bottlenecks list and detail | DAC-06, DAC-08 |
| TSK-4.8 | S-08 complete: errors, unmatched counts, missing-event reasons, retry/reconcile | FR-090 |
| TSK-4.9 | S-09 Settings: scope, definitions display, thresholds, window, retention/deletion action | FR-110–FR-114 |
| TSK-4.10 | Optional: CSV export, in-product notifications, filters | FR-130–FR-132 |
| TSK-4.11 | Accessibility checks and forbidden-language scan in CI | RL-41, DAC-07, DAC-10–DAC-13 |

Testing: design acceptance criteria DAC-01–DAC-16; insight dedup/reopen tests; threshold-evaluation tests; end-to-end journey (REQ §18.4) through evidence view and link correction.

**Definition of done:** UC-01–UC-06 completable; alert evidence traceable; unavailable/partial/active states displayed per design. **Exit criteria:** REQ §19 Phase 4 exit.

Risks/blockers: alert fatigue (RISK-08), metric misunderstanding (RISK-12), OQ-17 undefined thresholds.

Deferred: RULE-02–RULE-05 unless approved; Analytics screen if OQ-18 = no; dark theme (D-06).

## PH-5 — Pilot hardening

**Objective.** No known critical correctness or security defects remain in the MVP test scope; pilot limitations are documented.

| Item | Detail |
|---|---|
| Requirement IDs | Verifies all MUST items; AC-01–AC-09; NFR-001–NFR-012; SC-01–SC-10 |
| Prerequisites | PH-4 exit; pilot definition (OQ-15); privacy applicability decision (OQ-20); retention policy approved |

| ID | Task | Output |
|---|---|---|
| TSK-5.1 | Security review; run SV-01–SV-14; close P0/P1 items in security.md §13 | Review record |
| TSK-5.2 | Edge-case matrix complete (every `mvp.md` §14 row has a passing test) | AC-08 |
| TSK-5.3 | Invariant-to-test mapping complete (REQ §18.5) | AC-09 |
| TSK-5.4 | Rate-limit, sync-failure and reconciliation failure testing under realistic data volume | SV-09 |
| TSK-5.5 | UX validation and terminology review with pilot users | Findings |
| TSK-5.6 | Manual verification of metrics against source timelines for pilot repositories | Comparison record (REQ §18.6) |
| TSK-5.7 | Fix incorrect metrics and noisy alerts; tune thresholds for usefulness, not for more alerts | Defect log closed |
| TSK-5.8 | Document known limitations visible to pilot users; define feedback collection and defect triage | REQ §21 pilot readiness |

**Definition of done:** every checkbox in `mvp.md` §21 satisfied or explicitly waived by a recorded decision. **Exit criteria:** REQ §19 Phase 5 exit.

Risks/blockers: pilot team availability; discrepancies between GitHub data and expectations (RISK-02); unresolved P0/P1 security items.

## Requirement coverage by phase

| Phase | Requirement IDs |
|---|---|
| PH-0 | decisions behind FR-046, FR-050, FR-052, FR-058, FR-080; OQ-01–OQ-20 |
| PH-1 | FR-001–FR-004, FR-010–FR-016, FR-019, FR-020 (basic), FR-110, FR-113 |
| PH-2 | FR-017, FR-018, FR-021–FR-030, FR-133 |
| PH-3 | FR-040–FR-046, FR-050–FR-058, FR-115–FR-117, FR-120 (links, definitions) |
| PH-4 | FR-060–FR-064, FR-070–FR-072, FR-080–FR-084, FR-090, FR-091, FR-100–FR-103, FR-111, FR-112, FR-114, FR-120 (insights, thresholds), FR-130–FR-132 |
| PH-5 | Verification of all of the above |

## Explicitly deferred (from `mvp.md` §7.3; not scheduled)

GitLab, Bitbucket, Jira, Linear and other integrations · deployment metrics and merge-to-production time · advanced DORA · AI summaries/recommendations · forecasting · automated issue creation in external tools · complex role hierarchies and enterprise SSO · custom formula builder · individual scorecards or leaderboards · automatic task-complexity estimation · deep code-quality/security/coverage analytics · native mobile application · dark theme (design D-06).
