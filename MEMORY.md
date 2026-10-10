# DevFlow — Persistent Project Context (memory.md)

Compact context for future AI coding sessions. Read this first, then open the specific document you need. Do not copy whole sections from other documents into this file.

Last reviewed: 2026-10-10 · Source: `mvp.md` v0.1 · Repository evidence: **none yet**.

## 1. Documents

| File | Owns |
|---|---|
| [prd.md](prd.md) | What the product must do: FR, NFR, BR, MET, RULE, OQ, RISK, SC, AC |
| [architecture.md](architecture.md) | How it is built: modules, data flow, data model, AD decisions |
| [design.md](design.md) | UX/UI: tokens, screens S-01–S-10, interactions I-01–I-10, DG, DAC, D |
| [rules.md](rules.md) | Engineering conventions RL-01–RL-54 |
| [security.md](security.md) | Assets AS, threats T, controls SEC, verification SV, risks SR |
| [phases.md](phases.md) | Order of work PH-0–PH-5, TSK |
| `mvp.md` (repo root or as supplied) | Original product intent; source of truth; do not edit casually |

## 2. Product purpose and terminology

DevFlow: GitHub-first engineering workflow intelligence for small teams. It links work items to pull requests, computes transparently defined metrics, shows the evidence behind every number, and flags simple rule-based bottlenecks. It is team-level workflow diagnostics, never individual performance scoring.

| Term | Meaning |
|---|---|
| Workspace | Tenant boundary; owns integrations, repositories, derived data |
| Integration | A GitHub App installation or connection |
| Normalized event | Internal event derived from a source event; idempotent; has `occurred_at` and `received_at` |
| Metric definition / result | Versioned start/end/inclusion rules / computed value or state with sample count, coverage, evidence |
| MET-01 | PR first-review wait |
| MET-02 | PR time to merge |
| Insight | A rule-triggered alert with evidence and status `open`/`acknowledged`/`resolved` |
| Match status | `confirmed_manual`, `matched_exact_key`, `suggested`, `unmatched`, `rejected` |
| Connection state | `Connecting`, `Syncing`, `Connected`, `Needs attention`, `Disconnected` |
| Cohort | Which PRs a metric includes (created / merged / active in window) |
| Elapsed vs working time | Never mixed unlabelled |

## 3. Confirmed facts

CONFIRMED (stated in `mvp.md`; REQUIREMENT, not VERIFIED code):
- GitHub is the only MVP integration; one task-source path only; two core PR metrics are the recommended default; one explainable bottleneck rule is the minimum.
- Missing data is never zero; open PR age is never completed lead time; merge is not deployment; fuzzy matches are never auto-confirmed; no individual rankings.
- Ingestion is idempotent and tolerant of duplicates, late and out-of-order events; webhook signatures are verified; heavy work is asynchronous.
- Timestamps stored in UTC; metrics deterministic for a fixed event set and definition version.
- Workspace isolation is enforced server-side.

## 4. Technical stack

**UNKNOWN.** No repository was inspected because none was provided. `mvp.md` §16.1 lists candidates only (Next.js/React + TypeScript, Tailwind, Recharts, Node.js with Express/NestJS, PostgreSQL or MongoDB, BullMQ + Redis). None is chosen. When a repository exists: inspect it, then record the VERIFIED stack here with file paths.

## 5. Decisions

Approved decisions (none yet). Add rows only with approver and date.

| ID | Decision | Approved by | Date | Rationale | Recorded in |
|---|---|---|---|---|---|
| — | — | — | — | — | — |

PROPOSED (not approved; do not treat as final): AD-02 through AD-09 in architecture.md §12; token set in design.md §4; navigation structure in design.md §3; screen S-01 minimal form; NFR-013 (WCAG 2.2 AA target); RL-01–RL-54 conventions; security controls marked Proposed.

## 6. Open decisions (highest impact first)

| ID | Decision | Blocks |
|---|---|---|
| OQ-01 | Task source: GitHub Issues / native tasks / external tracker | Data model, Tasks screen, MET-03 |
| OQ-11 | Stack, database, jobs, hosting, authentication | All implementation |
| OQ-03, OQ-04 | Ready-for-review start event; qualifying review; dismissed; bots | MET-01/02 correctness |
| OQ-05, OQ-19 | Default cohorts; reopened PR semantics | MET-01/02 labels |
| OQ-06 | Elapsed only, or working time | FR-058, FR-116 |
| OQ-10 | Retention and deletion | FR-114, security P1 |
| OQ-12, OQ-16 | Roles; manual link correction priority (MUST vs SHOULD conflict) | FR-003/004/044 |
| OQ-09, OQ-17 | Thresholds, severities, reopen rule; sample-size definition | RULE-01, insight lifecycle |
| OQ-02, 07, 08, 13, 14, 15, 18, 20 | See prd.md §12 | Various |

## 7. Conventions in force

None are enforced by tooling yet. Planned conventions are PROPOSED in rules.md. Most important: pure metric/rules engines; UI never computes metrics; `workspace_id` scoping everywhere; idempotent writes; enum vocabulary from `mvp.md`; tagged-union duration results (never `0` for missing); no secrets or payload bodies in logs; stable IDs for all documentation items; do not renumber IDs.

## 8. Current implementation status

| Area | Status |
|---|---|
| Code | None provided / none inspected |
| Documentation | Seven docs drafted from `mvp.md`; unvalidated against any code |
| Tests, fixtures | None |
| Phase | PH-0 not started (ASSUMPTION) |

## 9. Constraints and known limitations

- Documents contain no verified implementation facts; every technical claim is PROPOSED or REQUIREMENT.
- GitHub event names, payload fields, permission names and signature details were **not** re-verified against current GitHub docs (network access was unavailable); verify at implementation (`mvp.md` §23 links).
- Colour contrast ratios in design.md §4.2 were computed; no screen has been built or tested.
- No numeric thresholds, sample sizes, retention periods, latency or scale targets exist; do not invent them.
- No compliance obligations are asserted (OQ-20).

## 10. Conflicts found in `mvp.md` (prd.md §11)

C-01 manual correction MUST (§9.4) vs SHOULD (§7.2) · C-02 five candidate rules vs one required · C-03 thresholds configurable vs fixed default allowed · C-04 roles SHOULD vs "authorized roles" required · C-05 Analytics page in §17.3 but not in Must list · C-06 task lead time depends on unresolved task source · C-07 reopen behaviour undefined · C-08 "sufficient sample" undefined · C-09 no sign-in/workspace screens described.

## 11. How to update this project context

1. A change in behaviour, scope, metric definition, decision, or status updates its owning document in the same change (rules.md RL-50).
2. To approve a PROPOSED item: change its label in the owning document, add a row in §5 with approver and date, and update any dependent document by ID reference.
3. To resolve an OQ: record the answer in prd.md §12, remove it from §6 here, and update affected documents (architecture AD, design D, phases prerequisites, security SQ).
4. When a repository first appears or changes: inspect it, change claims to VERIFIED with paths, fix any contradictions, update §4 and §8.
5. IDs are stable. Withdraw rather than delete or renumber.
6. After any update, run the cross-reference check: every ID referenced in one document exists in its owner document; terminology matches `mvp.md` vocabulary.
7. Keep this file short. Link; do not duplicate.
