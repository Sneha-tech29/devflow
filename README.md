# devflow
DevFlow --- MVP Specification
Document type: MVP definition / product discovery baseline  

Product: DevFlow --- Developer Collaboration & Engineering
Analytics  
Primary integration for MVP: GitHub  
Intended first users: Small software engineering teams  
Last updated: 2026-10-10
---
1. Purpose of This Document
This document defines the smallest credible, testable version of
DevFlow: what it is, whom it serves, the problems it solves, how the
main workflows should work, what data it needs, how metrics should be
defined, and what must be tested before the product can be trusted.
It is deliberately more detailed than a feature brainstorm, but it is
not yet a final PRD or implementation design. Use it as the source
document to refine product requirements, user stories, acceptance
criteria, technical architecture, and delivery milestones in `PRD.md`.
1.1 How to interpret this document
MUST means required for the MVP to be considered complete.
SHOULD means strongly recommended unless a documented reason
prevents it.
MAY means optional or a candidate for a later iteration.
A metric must never be presented as authoritative if its required
source events are missing or its definition has not been configured.
Sample values and thresholds in this document are illustrative until
validated with pilot teams.
---
2. Product Summary
DevFlow is a GitHub-first engineering workflow intelligence platform for
small software teams. It connects work items to repository activity and
turns the resulting event history into transparent delivery metrics and
evidence-backed bottleneck insights.
Teams often track tasks in one place, code in GitHub, reviews in pull
requests, and releases in another system. This fragmentation makes it
difficult to answer practical questions such as:
Which stage of delivery is taking the longest?
Are pull requests waiting for review longer than the team expects?
How long does work take from an agreed start point to completion?
Are delays caused by review queues, large changes, incomplete task
links, or another measurable stage?
Can the team trust the dashboard, and can each number be traced back
to source events?
DevFlow's MVP focuses on traceability, metric transparency, and basic
bottleneck detection, not employee surveillance or automated
performance ranking.
2.1 One-sentence value proposition
DevFlow helps small engineering teams understand how work moves from
task to merged code, identify where delivery is waiting, and verify
every metric against its underlying events.
2.2 Product positioning
Recommended positioning:
> A GitHub-first engineering workflow intelligence platform for small
> teams, with traceable task-to-code relationships, clearly defined
> lifecycle metrics, and evidence-backed bottleneck detection.
Do not claim that DevFlow is the first platform to combine task
management and engineering analytics. Existing tools cover parts of this
space. DevFlow's differentiation should be demonstrated through its
focus on explainable task-to-code traceability, data-quality visibility,
and actionable stage-level diagnosis for small teams.
---
3. Problem Statement
3.1 Current problem
Small engineering teams use multiple tools to manage tasks, commits,
pull requests, reviews, and releases. These systems often contain useful
data but do not automatically form one reliable view of the work
lifecycle.
Consequences include:
Fragmented context: A task, branch, pull request, and deployment
may not be visibly connected.
Unclear waiting time: Teams can see that work took a long time
without knowing which stage contributed most.
Manual reporting: Leads may collect status information from
multiple dashboards or ask team members for updates.
Weak metric definitions: Different people may calculate "cycle
time" or "review time" differently.
Incomplete data: Missing task links, late webhook events,
disconnected repositories, or incomplete historical imports can
distort analytics.
Misleading comparisons: Averages or individual-level rankings
can hide skewed distributions, task-size differences, and
overlapping work.
Low trust: If a dashboard cannot explain where a number came
from, users may not rely on it.
3.2 MVP problem to solve
For a team using GitHub, DevFlow must reliably collect supported
pull-request and review events, connect them to work items where
possible, calculate a small set of precisely defined metrics, show the
underlying evidence, and flag simple, explainable bottlenecks.
3.3 Problems explicitly outside the MVP
The MVP will not attempt to:
Replace a full project-management system.
Replace GitHub, a CI/CD platform, or a deployment platform.
Measure individual developer productivity or produce employee
rankings.
Infer developer intent, code quality, or business impact from
activity volume alone.
Claim production deployment frequency when no deployment source is
connected.
Use generative AI as the source of truth for metric calculations.
Support every Git provider and task tracker on day one.
Promise perfect automatic matching of every task and code change.
---
4. Target Users and Jobs to Be Done
4.1 Primary users
A. Engineering lead / team lead
Needs a reliable view of workflow health and where work is waiting.
Typical questions: - Which stage has the largest delay? - Is the PR
review queue getting worse? - Which projects have too much work in
progress? - Can I inspect the events behind this insight before acting?
B. Developer
Needs to understand the status of work and reduce avoidable waiting or
handoff friction.
Typical questions: - Which of my linked PRs are waiting for review? -
What is blocking a task from moving forward? - Is the dashboard's
interpretation of this PR correct?
C. Engineering manager / project owner
Needs a trend-level view of delivery and data quality, without treating
metrics as individual performance scores.
Typical questions: - Is delivery becoming more predictable? - Are the
metrics complete enough to use? - Did a workflow change improve or
worsen the observed delays?
4.2 Initial user profile
The MVP should optimize for a small team with: - One or more GitHub
repositories. - A basic task or issue workflow. - A team lead who wants
visibility without maintaining manual reports. - Limited time to
configure analytics. - A need to inspect and correct mismatched or
missing task links.
Team size is not a hard technical limit for the MVP. The first pilot
should be small enough to validate assumptions and investigate data
discrepancies directly.
4.3 Jobs to be done
When I connect a repository, I want DevFlow to import supported
activity so that I can see recent workflow history.
When a PR is associated with a task, I want to see their
relationship in one place.
When I review a metric, I want to know its exact start and end
events.
When DevFlow flags a bottleneck, I want evidence showing why it was
flagged.
When source data is missing or uncertain, I want that uncertainty to
be visible rather than hidden.
When a task is linked incorrectly, I want a way to correct the
relationship.
When a repository disconnects or data synchronization fails, I want
to know that the dashboard may be incomplete.
---
5. Product Principles
Evidence before inference: Every displayed metric and alert must
be traceable to stored source events and a metric-definition
version.
Missing is not zero: Missing data must be represented as
unavailable, incomplete, or unknown---not as a zero-duration metric.
Definitions before dashboards: Metric semantics must be
documented before values are displayed.
Source-of-truth clarity: Each data type must have an explicit
authoritative source.
Idempotent ingestion: Receiving the same source event more than
once must not duplicate its effect.
Reproducible calculations: The same normalized event history and
metric-definition version should produce the same result.
Human-confirmable linking: Automatic task matching must show its
confidence or method and permit correction.
Privacy by design: Collect only data needed for the selected
features and avoid unnecessary personal profiling.
Team improvement, not surveillance: Metrics describe workflow
conditions; they must not be presented as a definitive measure of
individual contribution.
Graceful uncertainty: If a source is disconnected or data is
incomplete, show the limitation prominently.
---
6. MVP Goals and Success Criteria
6.1 Product goals
Connect a GitHub repository through a least-privilege integration.
Import and normalize supported pull-request, review, and merge
events.
Associate tasks with PRs using explicit links and a transparent
matching hierarchy.
Calculate a small, clearly documented set of lifecycle metrics.
Show task/PR timelines and the events behind metric values.
Detect simple workflow bottlenecks using configurable, explainable
rules.
Expose synchronization health, unmatched activity, and data
completeness.
Validate the calculations against a controlled ground-truth dataset.
6.2 MVP success criteria
The following are proposed validation targets, not claims of existing
performance. Confirm or revise them after pilot testing.
---
Area                                Proposed acceptance target
---
Metric correctness                  100% of approved deterministic
ground-truth fixtures produce the
expected result
Duplicate handling                  Reprocessing a duplicate delivery
has no duplicate effect
Recalculation                       Replaying the same normalized event
set under the same definition
version gives the same result
Traceability                        Every displayed metric has a
visible definition and inspectable
source-event evidence
Data uncertainty                    Missing required events are shown
as unavailable/incomplete, not
silently converted to zero
Alert explainability                Every bottleneck alert includes the
rule, threshold, observed value,
time window, and evidence
Task matching                       Matching method is stored;
ambiguous matches are not silently
treated as confirmed
Sync visibility                     Users can see last successful sync
and any current synchronization
error
Security                            OAuth/App credentials and
repository data are protected
according to the security
requirements in this document
Usability                           A pilot user can connect a
repository and reach a first useful
dashboard without developer
assistance, subject to GitHub
authorization
Do not define success solely by the number of charts, features, or
collected events. Trustworthiness and interpretability are core product
outcomes.
6.3 Out-of-scope success claims
Do not claim a specific percentage reduction in cycle time, improved
productivity, or a universal industry benchmark without a measured pilot
and a defensible comparison method.
---
7. Scope: Must Have, Should Have, Later
7.1 Must have for MVP
User authentication and workspace boundary.
GitHub App or similarly secure GitHub authorization flow.
Repository selection and connection status.
Initial historical synchronization with pagination and resumable
checkpoints.
Webhook processing for supported PR/review lifecycle changes.
Raw event storage and normalized event representation.
Idempotency and safe retry handling.
Explicit task-to-PR linking.
Conservative task-key matching where supported.
Core lifecycle metrics with published definitions.
Task/PR lifecycle explorer.
Basic dashboard and bottleneck alerts.
Data-quality and integration-health page.
Metric evidence / audit trail.
Configuration for time window and alert thresholds.
Automated tests for metric definitions, duplicate events, ordering,
and missing data.
Disconnect/revoke flow and basic data deletion policy.
7.2 Should have if the MVP schedule permits
A small set of team-level filters (repository, date range, task
status).
Manual correction of task-to-PR relationships.
Export of a summary as CSV.
Notification inside the product when a sync has failed or a
threshold is crossed.
Simple configurable working hours, if working-time calculations are
included.
A lightweight event replay or admin reconciliation action.
7.3 Explicitly later
GitLab, Bitbucket, Jira, Linear, and other integrations.
Production deployment metrics unless a deployment provider is
integrated.
Advanced DORA reporting across all four metrics.
AI-generated summaries or recommendations.
Predictive forecasting and delivery-date predictions.
Automated issue creation in external tools.
Complex organization-wide role hierarchies and enterprise SSO.
Custom formula builder for arbitrary metrics.
Individual developer scorecards or leaderboards.
Automatic estimation of task complexity from code changes.
Deep code-quality, security, or test coverage analytics.
Native mobile application.
---
8. Core User Journeys
8.1 First-time setup
User creates an account and workspace.
User selects Connect GitHub.
GitHub authorization clearly explains requested permissions.
User selects the repositories DevFlow is allowed to analyze.
DevFlow stores the integration metadata and begins a historical
sync.
UI shows connection state: `Connecting`, `Syncing`, `Connected`,
`Needs attention`, or `Disconnected`.
The user sees the sync start time, last successful sync, progress if
measurable, and any limitations.
DevFlow displays a dashboard only for data that has been imported
and validated enough to support it.
Acceptance notes - Do not show "fully synced" until the configured
sync job completes. - A partial import must be labeled partial. - A
failed sync must not erase the last known valid data silently. - The
user must be able to disconnect the integration.
8.2 View project health
User opens the Overview page.
User chooses a repository or project and a date window.
DevFlow displays available metrics, sample size, and coverage
information.
User selects a metric card.
DevFlow opens the underlying task/PR records and relevant
timestamps.
If the metric is unavailable, DevFlow explains which event or
integration is missing.
8.3 Investigate a bottleneck
User opens Bottlenecks.
DevFlow lists current rule-based alerts.
Each alert explains the measured condition and threshold.
User opens an alert to inspect affected PRs/tasks and event
timestamps.
User can mark the insight as acknowledged or resolved and optionally
add a note.
The system records the status change and preserves the original
evidence.
8.4 Link a task to a PR
User opens a task or PR.
DevFlow shows existing links and the matching method.
User can confirm, change, or remove a link if permissions allow.
The system records who changed the link, when, and why if a reason
is supplied.
Derived metrics are recalculated where affected.
The dashboard reflects the new calculation and maintains an audit
trail.
8.5 Recover from a sync failure
DevFlow detects a webhook or reconciliation failure.
Integration status changes to `Needs attention`.
User sees a human-readable summary and last successful sync.
A retry or reconciliation job resumes from a safe checkpoint.
Newly recovered events are deduplicated and metrics are
recalculated.
The system records the recovery result.
---
9. Functional Requirements by Module
9.1 Authentication and workspace
MUST - Support a secure sign-in mechanism appropriate to the
selected implementation. - Associate connected repositories and derived
data with a workspace. - Enforce workspace-level data isolation on the
server, not only in the UI. - Prevent one workspace from reading another
workspace's repository data. - Provide a way to sign out and manage the
user's account.
SHOULD - Support workspace membership and a basic owner/member role
model. - Restrict integration management and destructive actions to
authorized roles.
Acceptance criteria - Unauthorized requests cannot retrieve
workspace data by changing an identifier. - All API reads and writes
enforce workspace access checks. - Automated tests cover cross-workspace
access attempts.
9.2 GitHub integration
MUST - Use a GitHub App or another appropriately scoped, secure
authorization model. - Request only permissions required by the selected
MVP features. - Let users select the repositories they want to
connect. - Record installation/account identifiers, repository
identifiers, connection status, and last successful sync. - Support
disconnecting the integration and handling revoked or expired access. -
Handle GitHub API pagination and rate limits. - Persist sync checkpoints
so long-running syncs can resume. - Never expose installation tokens,
access tokens, webhook secrets, or other credentials to the browser.
MUST support at least - Pull request opened/created. - Pull request
marked ready for review, if the source event and API support the
state. - Review submitted. - Review dismissed or otherwise invalidated
where relevant to the metric definition. - Pull request
synchronized/updated when new commits are pushed, if needed for the
timeline. - Pull request closed. - Pull request merged, including merge
timestamp where available. - Repository/integration lifecycle status
needed to detect disconnection.
Exact event names and payload fields must be checked against the current
GitHub API/webhook documentation during implementation. The normalized
domain model should not depend directly on one webhook payload shape.
SHOULD - Reconcile stored state with the GitHub REST API after
missed deliveries or detected inconsistencies. - Show the latest
successful reconciliation time. - Provide a clear explanation when a
permission is missing.
9.3 Event ingestion and processing
MUST - Validate webhook signatures using GitHub's documented
signature-verification method before trusting payloads. - Persist
delivery identifiers and use them for idempotency where available. -
Acknowledge valid webhooks promptly and process heavier work
asynchronously. - Store the source event or a sufficient immutable raw
representation for debugging and replay, subject to privacy and
retention rules. - Normalize source events into a consistent internal
event model. - Store both the source event's occurrence time and the
time DevFlow received it. - Handle duplicate deliveries safely. - Handle
late and out-of-order events. - Retry transient failures with bounded
exponential backoff or an equivalent retry strategy. - Record processing
failures and make them observable. - Recompute affected metrics when
late events or corrections change the event history.
MUST NOT - Assume webhook arrival order equals event occurrence
order. - Assume webhook delivery is exactly once. - Treat a successful
HTTP response from the webhook endpoint as proof that downstream
processing completed. - Delete previously valid data just because one
later API request fails.
Suggested normalized event fields - `id`: DevFlow internal event
ID. - `workspace_id`. - `integration_id`. - `repository_id` and source
repository identifiers. - `source`: for MVP, `github`. -
`source_event_type`. - `source_delivery_id`, if available. -
`subject_type`: for example, `pull_request`, `review`, `issue`, or
`task`. - `subject_id`: stable source identifier. - `action`: normalized
action. - `occurred_at`: source occurrence time, when known. -
`received_at`: time DevFlow received the event. - `source_updated_at`,
if available. - `payload_version` or schema version. -
`raw_payload_reference` or protected raw payload. -
`processing_status`. - `processing_error`, when applicable. -
`deduplication_key`.
The precise schema should be finalized in the technical design. Do not
use mutable display names as the sole identity for repositories, users,
PRs, or tasks.
9.4 Task-to-code linking
The MVP needs a transparent method to connect a work item with its PRs.
MUST support 1. Explicit user-created link. 2. Exact task-key match
in configured PR title/body/branch text, if the task-key convention is
configured. 3. Unmatched state when no reliable link exists. 4. A
matching-method field stored with every relationship. 5. A way to
inspect and correct links. 6. Audit history for manual link changes.
Recommended matching states - `confirmed_manual`: user explicitly
confirmed the relationship. - `matched_exact_key`: exact configured task
key found. - `suggested`: possible match that needs confirmation. -
`unmatched`: no reliable match. - `rejected`: user rejected a suggested
relationship.
Do not silently treat fuzzy text similarity as a confirmed link. If
fuzzy matching is later introduced, show it as a suggestion and require
confirmation unless pilot evidence supports a safer rule.
Relationship fields - `workspace_id`. - `task_source` and `task_id`,
if a task tracker is connected. - `task_key`, when applicable. -
`repository_id`. - `pull_request_id`. - `matching_method`. -
`match_status`. - `created_at`, `updated_at`. - `created_by`, when user
initiated. - `confirmed_by`, `confirmed_at`, where applicable. -
`audit_reason`, optional.
Task source boundary
For the MVP, choose one of these explicit paths before implementation:
Path A --- GitHub Issues as the task source: simplest first
integration; tasks are GitHub issues and PRs can be linked to them.
Path B --- lightweight DevFlow tasks: DevFlow provides minimal
tasks and links them to GitHub PRs.
Path C --- external task tracker: connect one external tracker
in addition to GitHub.
Recommendation: Start with Path A or Path B, not multiple task
trackers. If choosing GitHub Issues, clearly distinguish an issue from a
PR in the data model. If choosing native DevFlow tasks, keep task
management deliberately minimal so it does not become a
project-management product. This choice is a key unresolved product
decision for the PRD.
9.5 Metric engine
MUST - Store metric definitions and their version. - Calculate
metrics from normalized source events, not from UI state. - Define
required start/end events and inclusion/exclusion rules for each
metric. - Store the result's calculation time and relevant event
references. - Represent missing or invalid data explicitly. - Support
deterministic recomputation. - Include sample count and data coverage
wherever practical. - Use consistent timezone handling and UTC storage
for timestamps. - Document whether each metric uses elapsed time or
configured working time.
MUST NOT - Use a vague metric label without a definition. -
Substitute a guessed timestamp for a missing required event. - Treat an
open PR's current age as completed lead time. - Mix elapsed-time and
working-time values in one chart without a clear label. - Aggregate
metrics across incompatible workflows without warning.
9.6 Dashboard / Overview
MUST display - Connected repositories and their status. - Selected
date window. - Core metric cards that have enough valid data. - Sample
count or record count for each metric. - A visible indication when data
is incomplete. - A way to open the records contributing to a metric. - A
short list of current bottleneck alerts, if any.
SHOULD display - Median and 75th percentile for duration metrics
when sample size is sufficient. - Trend compared with a previous
comparable window, only when both windows have enough comparable data. -
A small "data health" summary: last sync, unmatched activity, and
missing data.
Avoid decorative precision. Do not show several decimal places for time
metrics unless there is a clear use case.
9.7 Lifecycle explorer
MUST - Display the timeline for a task/PR using available events. -
Show event names and timestamps. - Distinguish event occurrence time
from receipt/processing time where this matters. - Show links to the
source GitHub item when the user is authorized. - Show which events were
used for a metric. - Show gaps where an expected event is missing rather
than inventing a timeline step.
SHOULD - Filter by repository, date window, PR state, or task-link
status. - Show review rounds and relevant state changes.
9.8 Bottleneck detection
MUST - Start with deterministic, rule-based detection. - Make every
rule configurable or document its fixed default. - Show threshold,
observed value, time window, number of affected items, and evidence. -
Avoid making unsupported causal claims. - Support acknowledging and
resolving an insight. - Keep a record of when an insight was created and
updated.
Initial candidate rules 1. Long review wait: an open PR has
spent longer awaiting its first qualifying human review than the
configured threshold. 2. Large review queue: the count of PRs
waiting for their first qualifying review exceeds a configured
threshold. 3. Long open-PR age: an open PR exceeds a configured age
threshold, with its current state clearly labeled. 4. High stage
delay: a stage's duration is above a team-defined threshold for a
sufficient sample. 5. Data-link coverage warning: a meaningful share
of relevant PRs cannot be linked to a task.
These rules should be prioritized after pilot testing. The first rule is
likely the most useful initial bottleneck signal.
Alert wording requirements - Say "PRs have been waiting for review
longer than the configured threshold," not "reviewers are
unproductive." - Say "The observed review-wait duration is high in this
window," not "the review process caused the release delay," unless the
data proves that relationship. - Clearly distinguish an active,
incomplete interval from a completed duration. - Include links to
affected records and the exact evidence.
9.9 Data quality and integration health
MUST show - Integration status. - Last successful sync and
reconciliation time, if applicable. - Current sync errors in
understandable language. - Number of unmatched relevant PRs/tasks. -
Missing required events that prevent a metric from being calculated. -
Whether the data window is fully synced or partial. - Whether an event
is delayed, failed, or awaiting reconciliation, where known.
SHOULD show - Match coverage over the selected window. - Event
processing failures and retry status for authorized administrators. - A
reconciliation action if safe and feasible.
The data-quality page is a core feature, not an optional debugging
screen. It helps users understand whether analytics can be trusted.
9.10 Insight lifecycle
MUST - Each insight has a stable ID, rule ID/version, creation time,
affected records, evidence, and status. - Statuses should include at
least `open`, `acknowledged`, and `resolved`. - Status changes record
actor and timestamp. - A resolved insight can reopen if the rule
condition becomes true again, but the behavior must be explicitly
defined. - The product must distinguish "condition no longer detected"
from "user marked resolved."
9.11 Settings
MUST - Configure repository scope. - Configure date window and basic
bottleneck thresholds. - Display metric definitions. - Provide
integration disconnect controls. - Provide a documented data
retention/deletion path.
SHOULD - Configure task-key format or prefix rules. - Configure
working hours if working-time metrics are supported. - Configure whether
bots are excluded from qualifying review events.
---
10. Metric Dictionary
Metric definitions must be reviewed before implementation and treated as
versioned product behavior. Names can be familiar, but the exact
start/end events and exclusions must be explicit.
10.1 General rules
Store timestamps in UTC; render them in the user's selected or
browser timezone with a clear label.
For elapsed-time metrics, calculate the difference between
timestamps in UTC.
For working-time metrics, use an explicit calendar and timezone. Do
not silently assume office hours.
Keep elapsed time and working time separate.
Duration metrics should be non-negative. If source corrections
produce an invalid sequence, mark the result invalid or pending
reconciliation rather than clipping it silently to zero.
Completed and currently active intervals must be represented
separately.
Report median and percentiles when sample sizes permit; averages
alone can be distorted by long-running outliers.
Every metric needs a sample count, time window, definition version,
and data-coverage status.
10.2 MVP metric candidates
A. PR first-review wait
Purpose: Measure how long a PR waits from becoming ready for review
until the first qualifying human review event.
Proposed definition: - Start: timestamp when the PR becomes ready
for review. If draft PRs are excluded and the PR was never a draft, use
the PR creation timestamp. - End: timestamp of the first qualifying
human review submission after the start. - Unit: elapsed time by
default. - Status: completed if a qualifying review exists; active if
the PR is still waiting; unavailable if required timestamps are missing.
Important edge cases - Draft PRs should not accrue "ready-for-review
wait" before they are ready. - Define whether a comment-only review
qualifies. Recommended default: a submitted formal PR review qualifies;
ordinary issue/PR comments do not. - Bots should be excluded by default
where identity classification is reliable. - A dismissed review should
not automatically count as the first valid review unless the product
definition explicitly says otherwise. - A review submitted before the PR
became ready should not end the interval. - Multiple reviewers do not
create multiple first-review completion events.
B. PR time to merge
Purpose: Measure elapsed time from the PR's qualifying start to
merge.
Proposed definition: - Start: PR becomes ready for review; if never
draft, PR creation time. - End: merge timestamp. - Population: merged
PRs in the selected cohort, with clear cohort rules. - Status: completed
for merged PRs; active for open PRs; unavailable if timestamps are
missing.
Do not compare this metric across teams without checking differences in
PR policy, draft usage, repository type, and workflow.
C. Task lead time (only if task lifecycle data is available)
Purpose: Measure elapsed time from the task entering the agreed
"started" state to completion.
Proposed definition: - Start: first transition into a configured
started/in-progress state. - End: transition into the configured
completed state. - Source: the selected task system. - If using GitHub
Issues only, issue open/close timestamps alone may not represent
work-start time. Do not label open-to-close duration as engineering lead
time without clearly naming that narrower definition.
If a task source with reliable state transitions is not included in the
first MVP, defer this metric rather than inventing start timestamps.
D. Coding-to-PR time (optional, only with a defensible start event)
Potential definition: first commit associated with the task/PR to the
time the PR becomes ready for review. This is only meaningful if commit
association and the intended start event are reliable. Do not equate the
first commit timestamp with the true start of coding in all cases.
E. PR merge-to-production time (not in the core MVP without deployment data)
Requires a trusted deployment source and an explicit mapping from
merge/commit to production deployment. A merge is not proof of
production deployment. Until a deployment integration exists, this
metric must be unavailable and must not be estimated from merge
timestamps.
F. Deployment frequency (not in core MVP without deployment data)
Requires a reliable definition of a production deployment and a
deployment event source. GitHub merges alone are not an adequate
substitute.
G. Work in progress (WIP)
Possible MVP definition: count of tasks in configured in-progress
states, or open PRs, at a given time. These are different populations
and must have separate labels. Do not combine active tasks and open PRs
into one unlabeled number.
10.3 Metrics to avoid in MVP
"Developer productivity score."
Lines of code as a proxy for contribution.
Commits per developer as a productivity ranking.
AI-generated estimates of task effort as if they were ground truth.
DORA metrics calculated from incomplete proxy events.
A single universal "cycle time" that mixes tasks, PRs, and
deployments.
10.4 Review-cycle semantics
A PR may have multiple review rounds, changes requested, new commits,
dismissed reviews, and later approvals. For MVP: - Define "first review
wait" separately from "time to approval" and "time to merge." - Store
every supported review event needed to reconstruct the timeline. - Do
not assume the first review is an approval. - Do not assume an approval
means the PR is mergeable or ready to ship. - If "review delay after new
commits" is introduced later, define exactly which commit or ready state
starts each interval and how repeated rounds are aggregated. - Avoid
double-counting overlapping wait intervals when calculating a stage's
share of total time.
10.5 Cohort rules
Every chart must explain which records are included. Possible cohort
choices: - PRs merged within the selected period. - PRs created within
the selected period. - PRs whose lifecycle ended within the selected
period. - PRs active at any time during the selected period.
These cohorts answer different questions. The MVP should pick one
default per metric and make it visible in the UI. Avoid comparing a
"created this week" cohort to a "merged this week" cohort without
labeling the difference.
---
11. Bottleneck Intelligence: Rules and Evidence
11.1 Detection model
The MVP should use a transparent rules engine. Each rule includes: -
Stable rule ID and version. - Description and purpose. - Required
event/data fields. - Evaluation window. - Threshold and severity. -
Cooldown/deduplication behavior. - Evidence records. - Explanation
template. - Known limitations.
11.2 Example rule: PR waiting for first review
Condition: A PR is ready for review, has no qualifying human review
after the ready timestamp, and its active wait duration exceeds the
configured threshold.
Required evidence: - PR ID and source link. - Ready-for-review
timestamp. - Current evaluation timestamp. - Whether qualifying review
events were checked. - Repository and configured threshold. - Data
sync/coverage status.
Possible explanation: > This PR has been ready for review for 19
hours and has no qualifying human review recorded. Your configured
threshold is 12 hours. The repository's latest sync was successful at
[timestamp].
The values above are illustrative. The actual message must use real
calculated values and must warn if event coverage is incomplete.
11.3 Thresholds and statistics
Default thresholds are product assumptions and should be
configurable.
Do not describe a threshold as an industry standard unless a source
and context support that claim.
Prefer median and 75th percentile for duration distributions when
there are enough observations.
Use p90 only when the sample is large enough to make it informative;
otherwise show sample size and avoid overstating precision.
Compare a current window with a previous window only when cohort
definitions and coverage are comparable.
Distinguish a single slow item from a systemic queue pattern.
Do not imply causation from correlation alone.
11.4 Avoiding misleading "stage share" calculations
If calculating the percentage of total time spent in review wait or
another stage: - Define the total interval first. - Define whether
stages can overlap. - If stages overlap, do not add their durations as
if they were disjoint. - Use a non-overlapping partition of the timeline
if a percentage of total elapsed time is required. - If this cannot be
calculated reliably, show stage durations separately rather than
inventing a percentage.
---
12. Data Model: Conceptual Entities
This section describes concepts, not a finalized database schema.
12.1 Workspace
Represents a team or tenant.
Likely fields: - ID, name, created timestamp, owner/member
relationships, settings.
12.2 User
Represents an authenticated DevFlow user.
Likely fields: - ID, authentication identity, display name, account
status, created timestamp.
12.3 Integration
Represents a GitHub App installation or connection.
Likely fields: - ID, workspace ID, provider, external
installation/account ID, permissions metadata, status, connected
timestamp, last sync, last reconciliation, error summary.
Secrets and tokens must be stored securely and never included in normal
API responses or logs.
12.4 Repository
Represents a selected GitHub repository.
Likely fields: - Internal ID, integration ID, stable external repository
ID, owner/name for display, default branch if needed, status, last sync.
12.5 Task
Represents a work item from the chosen MVP task source.
Likely fields: - Internal ID, workspace ID, source type, source ID, key,
title, state, created/updated/started/completed timestamps when
genuinely available, source URL.
Do not fabricate a started timestamp if the source does not provide a
reliable event.
12.6 Pull request
Represents a GitHub PR.
Likely fields: - Internal ID, repository ID, stable external PR
ID/number, title, URL, author identifier, draft status,
created/ready/closed/merged timestamps, current state, latest source
update time.
12.7 Review
Represents a review event or review record.
Likely fields: - Stable source ID, PR ID, reviewer identity/reference,
review state, submitted timestamp, dismissed timestamp if applicable,
bot classification status if known.
12.8 Normalized event
Stores the timeline event and its processing metadata.
Use the fields suggested in Section 9.3, refined during technical
design.
12.9 Task--PR relationship
Stores the link, matching method, confirmation status, and audit
history.
12.10 Metric definition
Stores metric ID/name, definition version, start/end event rules,
inclusion/exclusion rules, unit, elapsed/working-time mode, cohort
definition, and status behavior.
12.11 Metric result
Stores the calculated value or state, metric-definition version,
cohort/window, sample count, data-coverage status, calculation
timestamp, and references to supporting events/records.
12.12 Insight
Stores rule ID/version, scope, threshold, observed value, affected
record references, evidence, status, created/updated timestamps, and
status-change history.
12.13 Audit record
Stores relevant user actions and system actions, such as manual link
changes, integration changes, metric-definition changes, and insight
status updates.
---
13. Data Source and Authority Rules
Each fact needs an authoritative source.
---
Information             Preferred source        Important limitation
---
PR creation and state   GitHub                  Must reconcile if
webhooks are missed
PR ready-for-review     GitHub event/state      Draft transitions must
state                   history where available be modeled correctly
Review submission/state GitHub                  Define qualifying
reviews and bot
exclusions
PR merge time           GitHub                  Merge does not prove
production deployment
Task state transitions  Chosen task source      GitHub Issue open/close
alone may not supply a
true start event
Production deployment   Deployment provider /   Not available from
environment event       merge data alone
Working hours/calendar  Workspace configuration Timezone and holidays
must be explicit if
supported
Manual task--PR         DevFlow user action     Must be auditable
correction
Derived metric          DevFlow metric engine   Must reference
definition version and
source events
If sources disagree, preserve the evidence and mark the record for
reconciliation rather than silently selecting an arbitrary timestamp.
---
14. Edge Cases and Required Behavior
---
Case                                Required behavior
---
Duplicate webhook delivery          Deduplicate; no duplicate metric or
event effect
Out-of-order events                 Order by occurrence time and event
semantics, not receipt time alone
Late event                          Recompute affected derived metrics
and record the update
Draft PR                            Do not count draft time as
ready-for-review wait under the
proposed definition
PR reopened                         Preserve previous lifecycle events;
define whether the current cycle is
a new interval
Review dismissed                    Apply the documented
qualifying-review rule; do not
silently count a dismissed event
incorrectly
Bot review                          Exclude by default where bot
identity is reliable; expose
uncertainty if not
Missing timestamp                   Mark metric unavailable/incomplete;
do not invent a value
Task linked after merge             Recompute task-linked metrics;
preserve link-change audit history
Ambiguous task key                  Show suggestion or unmatched state;
do not auto-confirm a fuzzy match
Repository disconnected             Mark integration state and data
window; keep existing data
according to retention policy
GitHub API rate limit               Back off, resume from checkpoint,
and expose sync health
Partial historical sync             Label affected data partial; do not
show it as fully complete
PR merged but not deployed          Do not infer deployment from merge
Force-push / new commits            Preserve relevant source
state/event history where
available; do not overwrite
timeline facts without audit
Changed task title/key              Use stable source identifiers;
avoid relying only on mutable title
text
Timezone changes                    Store UTC; display with explicit
timezone; do not alter the
underlying event instant
Workspace member leaves             Apply documented access revocation
and data ownership policy
Installation permissions revoked    Stop unauthorized calls, mark
connection needs attention, protect
stored secrets
Reconciliation changes a previous   Record source update and
fact                                recomputation; do not silently hide
the change
This table must become a concrete test matrix in the PRD and engineering
test plan.
---
15. Security, Privacy, and Trust
15.1 Integration security
Use a least-privilege GitHub App configuration.
Request only permissions required for the selected features.
Validate webhook signatures using the documented GitHub method.
Store secrets in a secure server-side secret store or encrypted
configuration appropriate to the deployment.
Never log access tokens, installation tokens, webhook secrets, or
sensitive payload fragments.
Support disconnection and token/installation revocation handling.
Restrict administrative operations to authorized users.
Use HTTPS in deployed environments.
15.2 Workspace isolation
Every workspace-owned record must be scoped to its workspace.
Enforce access checks in server-side queries and mutations.
Test direct-object-reference attacks and cross-workspace ID
substitution.
Do not rely only on client-side route guards.
15.3 Data minimization
Store only GitHub metadata needed for the product.
Avoid downloading repository source code unless a later feature
specifically requires it.
Do not collect private message content or unrelated user data.
Be clear about what metadata is collected and why.
Define retention and deletion behavior before onboarding real pilot
users.
15.4 Human impact
Present analytics as workflow diagnostics, not definitive judgments
about people.
Avoid individual rankings and simplistic "productivity scores."
Provide enough context for users to interpret unusual cases.
Make bot handling and incomplete-data limitations visible.
Avoid claiming that a person caused a delay solely because an event
is associated with their account.
15.5 Auditability
Maintain an audit trail for: - Integration connection/disconnection. -
Repository scope changes. - Manual task--PR link creation or
correction. - Metric-definition or threshold changes. - Insight
acknowledgement/resolution. - Administrative replay/reconciliation
actions.
The exact retention period and user-visible audit scope remain decisions
for the PRD.
---
16. Architecture Direction (Not a Locked Technology Decision)
A reasonable initial architecture consists of:
Web application: dashboard, settings, task/PR explorer,
data-quality view.
Application API: authentication, workspace access, queries,
settings, link management.
GitHub integration layer: authorization, API client, repository
sync, webhook endpoint.
Event-processing worker: normalization, idempotency, retries,
reconciliation, metric recomputation.
Database: workspaces, integrations, repositories, source
records, normalized events, links, metrics, insights, audit records.
Job queue: asynchronous sync/reconciliation and metric
recomputation if required by the chosen scale.
Observability: structured logs, processing errors, sync status,
and basic operational alerts.
16.1 Possible stack
A suitable option---not a requirement---is: - Frontend: Next.js or React
with TypeScript. - Styling: Tailwind CSS or a consistent component
system. - Charts: Recharts or a similar library. - Backend: Node.js with
a structured API framework such as Express or NestJS. - Database:
PostgreSQL is worth considering for relational links, constraints, and
auditability; MongoDB is also possible if the data model and integrity
rules are deliberately designed. Choose one after validating query
patterns rather than by habit. - Background jobs: a queue such as BullMQ
with Redis, or a managed job system, if needed. - Hosting: a platform
that supports secure environment variables, webhook endpoints,
background jobs, and durable database storage.
Do not lock the stack in `MVP.md` if the implementation environment has
not been chosen. The PRD should document the final decision and
trade-offs.
16.2 Historical sync and ongoing updates
Recommended flow: 1. Authorize and select repositories. 2. Create a
resumable sync job. 3. Fetch historical PR/review data using paginated
API requests. 4. Store source records and normalized events
idempotently. 5. Register/verify webhooks. 6. Process incoming webhooks
asynchronously. 7. Periodically reconcile important records with the
GitHub API. 8. Recompute affected metrics after material updates. 9.
Expose sync state and data coverage to users.
The exact order of webhook registration and historical import should be
designed to avoid a gap in event coverage. A robust approach may
register webhooks early and reconcile events that overlap the historical
import.
16.3 Reconciliation
Reconciliation is required because webhook delivery can fail, events can
arrive late, and source records can change.
A reconciliation job should: - Fetch source state for records within a
defined window or scope. - Compare source state with stored state. -
Detect missing or changed facts. - Upsert corrected source records and
normalized events. - Recompute affected metric results. - Record the
reconciliation time and outcome. - Avoid duplicating effects when rerun.
---
17. UI / Information Architecture
17.1 Overview
Purpose: a concise view of delivery workflow health.
Candidate content: - Repository selector. - Date-window selector. - PR
first-review wait. - PR time to merge. - Open PRs currently waiting for
review. - Recent bottleneck alerts. - Data-quality summary. - Last
successful sync.
Only show cards for metrics supported by the connected data. Label
unavailable metrics and explain why.
17.2 Tasks / Work items
Purpose: inspect tasks and their code links.
Candidate content: - Task list and status. - Linked PRs. - Matching
method/status. - Unmatched or suggested links. - Manual link
correction. - Source links.
17.3 Engineering Analytics
Purpose: inspect metric distributions and trends.
Candidate content: - Metric selector. - Cohort and date window. -
Median/p75 and sample count. - Trend chart. - Drill-down to contributing
PRs. - Metric definition and coverage explanation.
17.4 Bottlenecks
Purpose: turn metrics into actionable workflow observations.
Candidate content: - Open/acknowledged/resolved filters. - Rule name and
severity. - Observed value vs threshold. - Affected items. - Evidence
and timeline. - Acknowledge/resolve action.
17.5 PR / task lifecycle explorer
Purpose: answer "what happened, and when?"
Candidate content: - Ordered event timeline. - Current source state. -
Metric interval boundaries. - Review rounds and merge event. - Link
confidence/method. - Sync completeness warning.
17.6 Integrations & Data Quality
Purpose: establish whether the analytics are trustworthy.
Candidate content: - GitHub installation and repository scope. -
Connection status. - Last successful sync/reconciliation. - Missing or
delayed event warning. - Unmatched task/PR counts. - Sync error
details. - Retry/reconcile action if available. - Disconnect
integration.
17.7 Settings
Purpose: configure behavior.
Candidate content: - Metric definitions. - Task-key conventions. - Bot
exclusion behavior. - Alert thresholds. - Timezone/working calendar, if
working-time metrics exist. - Retention and integration settings.
---
18. Testing and Validation Plan
Testing is a core product requirement, not a final cleanup step.
18.1 Ground-truth fixtures
Create a small, manually verified dataset with expected outcomes for
every MVP metric. Include: - Simple PR with one qualifying review. -
Draft PR later marked ready. - PR merged after multiple review rounds. -
Review before ready-for-review. - Bot review followed by human review. -
Dismissed review. - Open PR still waiting. - Missing ready timestamp. -
Duplicate webhook. - Out-of-order events. - Late event requiring
recomputation. - Task link added after the PR has merged. - Partial sync
and later reconciliation. - PR merged but no deployment event.
For each fixture, document input events, expected metric state/value,
and rationale.
18.2 Unit tests
Test: - Each metric's start/end selection. - Qualifying-review logic. -
Draft behavior. - Bot exclusion. - Dismissal behavior. -
Negative/invalid duration handling. - Cohort membership. -
Percentile/summary calculations. - Metric versioning. - Threshold
evaluation. - Missing-data states. - Insight deduplication and reopening
rules.
18.3 Integration tests
Test: - Webhook signature validation. - Duplicate delivery handling. -
Persistence and normalized event creation. - GitHub pagination. -
Rate-limit backoff. - Retry behavior. - Reconciliation and upsert
logic. - Workspace authorization. - Manual link correction and audit
record creation. - Metric recomputation after changed source data.
18.4 End-to-end tests
Test a user journey: 1. Create workspace. 2. Connect a test GitHub
installation/repository. 3. Import fixture data. 4. View metric. 5. Open
evidence. 6. Correct a task--PR link. 7. Observe recalculation. 8.
Trigger or simulate a sync failure. 9. Confirm data-health warning. 10.
Reconcile and verify recovery.
18.5 Required invariants
Duplicate source deliveries do not create duplicate effects.
Metric results are deterministic for a fixed event set and
definition version.
A duration is never negative.
Missing required timestamps never become zero-duration values.
Every displayed metric has a definition version and evidence path.
Every bottleneck alert identifies the rule and threshold that
triggered it.
Replaying events does not create duplicate insights or audit records
without reason.
A partial sync is never presented as a complete dataset.
Cross-workspace requests cannot access another workspace's data.
Disconnecting an integration prevents further unauthorized API use.
18.6 Pilot validation
Run a pilot with one or a few consenting teams. Compare DevFlow's
calculated results against manually verified source timelines. Record: -
Incorrect calculations. - Missing events. - Incorrect or ambiguous task
links. - Confusing metric labels. - Alerts users consider useful or
noisy. - Cases where a metric creates an unfair or misleading
impression. - Time needed to connect a repository and understand the
first dashboard.
Do not tune thresholds solely to produce more alerts. Optimize for
useful, defensible signals.
---
19. Delivery Plan
The following phases are suggested sequencing, not calendar commitments.
Phase 0 --- Resolve product choices
Choose the MVP task source: GitHub Issues or lightweight native
tasks.
Finalize the first metric definitions and cohorts.
Define qualifying review behavior.
Choose the initial stack and deployment environment.
Decide data retention and deletion policy.
Create ground-truth test fixtures before implementing the metric
engine.
Exit criteria: Product choices and metric definitions are documented
and reviewable.
Phase 1 --- Foundations and GitHub connection
Authentication and workspace model.
GitHub authorization and repository selection.
Secure credential handling.
Repository and integration status.
Historical sync with pagination/checkpoints.
Basic webhook endpoint and signature validation.
Exit criteria: A test repository can be connected and imported
without exposing credentials.
Phase 2 --- Event model and reliability
Normalized event schema.
Idempotency and duplicate handling.
Asynchronous processing and retries.
Out-of-order and late-event handling.
Sync status and operational logging.
Initial reconciliation.
Exit criteria: Replaying a fixture and delivering
duplicates/out-of-order events produces a correct normalized timeline.
Phase 3 --- Linking and metrics
Task/PR linking path.
Link method/status and manual correction.
Metric definitions and versioning.
First-review wait and time-to-merge calculations.
Metric evidence view.
Ground-truth test suite.
Exit criteria: Approved fixtures calculate exactly as expected, and
every result can be inspected.
Phase 4 --- Dashboard and bottlenecks
Overview page.
Lifecycle explorer.
Basic bottleneck rule(s).
Insight status lifecycle.
Data-quality page.
Threshold settings.
Exit criteria: A pilot user can identify a delayed PR and understand
the evidence behind the alert.
Phase 5 --- Pilot hardening
Security review.
Cross-workspace access tests.
Rate-limit and sync-failure tests.
Reconciliation testing.
UX validation and terminology review.
Fix incorrect metrics and noisy alerts.
Exit criteria: No known critical correctness/security defects remain
in the MVP test scope; pilot limitations are documented.
---
20. Risks and Mitigations
---
Risk                    Why it matters          Mitigation
---
Metric ambiguity        Different definitions   Version and publish
create conflicting      precise definitions
results                 before implementation
Missing or delayed      Dashboard may be        Reconciliation,
events                  incomplete              coverage status, and
explicit incomplete
states
Duplicate webhooks      Inflated counts or      Delivery-level
duplicate transitions   idempotency and unique
constraints
Incorrect task matching Metrics attributed to   Exact-key/manual
the wrong work item     linking first;
suggestions remain
unconfirmed
GitHub API limits       Historical sync may     Pagination, backoff,
fail or stall           checkpoints, bounded
concurrency
Draft/review complexity Review-wait             Formal event semantics
calculations can be     and ground-truth
wrong                   fixtures
Overclaiming deployment Merge does not equal    Defer deployment
metrics                 production deployment   metrics until a
deployment source is
connected
Alert fatigue           Too many low-value      Configurable
alerts reduce trust     thresholds,
deduplication, pilot
feedback
Misuse as employee      Metrics may be          Team-level workflow
surveillance            interpreted unfairly    framing; no
productivity rankings
Scope expansion         Integrations and custom One provider, one
analytics delay the MVP task-source path, small
metric set
Data/security breach    Private repository      Least privilege,
metadata can be         workspace isolation,
sensitive               secret protection,
retention controls
Misleading comparisons  Different               Visible cohort
cohorts/workflows       definitions and sample
produce incomparable    sizes
values
Recalculation changes   Users may not           Store definition
old values              understand why a metric versions, evidence, and
changed                 recalculation history
---
21. Definition of Done for the MVP
The MVP is ready for pilot use when all applicable criteria below are
satisfied.
Product behavior
[ ] A user can create/access a workspace.
[ ] A user can connect and disconnect a GitHub
installation/repository.
[ ] Historical sync and ongoing webhook processing work for the
supported event set.
[ ] Sync state and errors are visible.
[ ] Tasks and PRs can be linked through the chosen MVP task-source
path.
[ ] Link method and status are visible and corrections are
auditable.
[ ] Core metrics have explicit definitions, cohorts, and data-status
behavior.
[ ] Users can inspect the source events behind a metric.
[ ] At least one useful bottleneck rule is available and
explainable.
[ ] Users can acknowledge/resolve insights.
[ ] Incomplete data is not represented as zero or as fully synced.
Correctness and reliability
[ ] Ground-truth fixtures pass.
[ ] Duplicate events are idempotent.
[ ] Out-of-order and late events are handled.
[ ] Reconciliation repairs supported discrepancies.
[ ] Recalculation is deterministic for a fixed event set and
definition version.
[ ] Every alert has traceable evidence.
[ ] Missing-data and invalid-duration behavior is tested.
Security and operations
[ ] Workspace isolation is tested.
[ ] Secrets are not exposed to clients or logs.
[ ] Webhook signature validation is tested.
[ ] Integration revocation/disconnection is handled.
[ ] Rate limits and retry failures are handled.
[ ] Retention/deletion behavior is documented.
[ ] Known limitations are visible to pilot users.
Pilot readiness
[ ] At least one end-to-end pilot workflow has been tested.
[ ] Metrics have been manually checked against source timelines.
[ ] Alert thresholds and terminology have been reviewed.
[ ] Feedback collection and defect triage are defined.
---
22. Decisions to Resolve Before Converting This into `PRD.md`
These are the highest-impact open questions. Do not leave them implicit.
Task source: Will the first version use GitHub Issues, native
DevFlow tasks, or one external task tracker?
Primary metric: Is first-review wait the primary initial metric,
or is task lifecycle duration more important?
Ready-for-review start: How should DevFlow identify the exact
start event for draft and non-draft PRs?
Qualifying review: Which review states count as the first
review? How are dismissed reviews and bots treated?
Cohort defaults: Are charts based on PR creation date, merge
date, or activity during the selected window?
Working time: Is the MVP elapsed-time-only, or does it require
working-hours calculations?
Historical scope: How much history should the first sync import,
and should users be able to configure it?
Manual links: Who can create/correct task--PR links, and must a
reason be recorded?
Alert defaults: What are the initial thresholds, severity
levels, deduplication windows, and reopening rules?
Data retention: How long are raw payloads, normalized events,
audit records, and derived results retained?
Hosting and stack: Which framework, database, job system, and
hosting platform will be used?
Roles: Does the MVP need only a workspace owner, or owner/member
roles?
Notifications: Are in-product alerts sufficient, or are
email/Slack notifications required?
Data export: Is CSV export required for the first pilot?
Pilot definition: Who will test the MVP, with what
repository/data, and what correctness criteria determine acceptance?
Recommended default to keep the first release focused: one task-source
path, GitHub integration, two core PR metrics (first-review wait and
time-to-merge), one explainable bottleneck rule, a lifecycle explorer,
and a data-quality page.
---
23. Research and Reference Links
These references inform the product direction and implementation risks.
Re-check the current documentation before implementation because APIs,
permission names, and product capabilities can change.
GitHub
Pull requests REST API: https://docs.github.com/en/rest/pulls
Pull request reviews REST API:
https://docs.github.com/en/rest/pulls/reviews
Validate webhook deliveries:
https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries
Webhook events and payloads:
https://docs.github.com/en/webhooks/webhook-events-and-payloads
GitHub Apps: https://docs.github.com/en/apps/creating-github-apps
Comparable workflow and analytics products
GitLab Value Stream Analytics:
https://docs.gitlab.com/user/group/value_stream_analytics/
Linear Insights: https://linear.app/docs/insights
Jira Control Chart:
https://support.atlassian.com/jira-software-cloud/docs/view-and-understand-the-control-chart/
Jellyfish Engineering & Product Operations:
https://jellyfish.co/solutions/engineering-product-operations/
DORA / delivery measurement
DORA metrics guide: https://dora.dev/guides/dora-metrics/
DORA measurement frameworks research:
https://dora.dev/research/2025/measurement-frameworks/
These tools demonstrate that delivery analytics and workflow measurement
already exist in the market. DevFlow should differentiate through clear
evidence, conservative task linking, visible data quality, and an MVP
optimized for small-team workflows---not through an unsupported claim of
novelty.
---
24. Final MVP Definition
The most focused credible MVP is:
> A secure GitHub-connected application that imports pull-request and
> review history, links PRs to work items through a clearly defined
> task-source path, calculates first-review wait and time-to-merge using
> documented rules, displays the lifecycle evidence behind each result,
> flags PRs waiting beyond a configurable threshold, and shows users
> when the underlying data is incomplete.
The MVP should be considered successful when a pilot team can answer
where work is waiting, why DevFlow believes that, and whether the data
is complete enough to trust the answer---without manually
reconstructing the timeline from multiple tools.
Everything else should be prioritized only if it strengthens that
outcome or is necessary for safe, accurate operation.
