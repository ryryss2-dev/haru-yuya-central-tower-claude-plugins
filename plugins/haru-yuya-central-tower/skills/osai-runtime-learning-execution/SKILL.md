---
name: osai-runtime-learning-execution
description: Use this skill for Haru-Yuya Central Tower OSAI work: task packets, routing, failover, fallback, roaming, testcodex parity, agent behavior, Judge and benchmark interpretation, Teacher Inbox, learning events, canonical update candidates, and OSAI Runtime Intelligence dashboard. Load together with central-tower-common-safety.
---

# Haru-Yuya Central Tower OSAI Runtime and Learning Execution Skill

## 1. Required companion skill

Always load:

`central-tower-common-safety`

This OSAI skill defines OSAI-specific responsibilities only.  
The common skill remains authoritative for workspace, Board, secrets, cost gates, hot-path gates, validation, and reporting.

## 2. OSAI mission

OSAI answers:

> Given the current Health evidence and the task, which route should be used, how should failures be handled, how good was the result, and what should be learned?

OSAI owns:

- task packet semantics
- planner decisions
- route selection
- fallback
- failover
- roaming
- task suitability
- agent behavior
- testcodex parity
- Judge and benchmark interpretation
- Teacher Inbox
- rubrics
- learning events
- feedback
- escalation candidates
- canonical update candidates
- OSAI Runtime Intelligence dashboard
- final answer ownership

OSAI does not own credential truth or current provider availability evidence.  
Those come from Health.

## 3. OSAI mainline

The mainline is:

```text
task input
→ normalized task_packet
→ capability and policy requirements
→ consume Health AVAILABLE inventory
→ route selection
→ provider execution
→ bounded fallback/failover/roaming
→ result normalization
→ answer ownership
→ telemetry
→ Judge / Teacher / Learning
```

Do not let dashboard polish block this path.

Do not stop at static previews when the approved task requires real runtime integration.

## 4. Health evidence consumption

OSAI may consume only clearly identified Health evidence.

Recommended route cell:

```text
provider_id
+ credential_slot
+ model
+ request_shape
+ payload_size_bucket
+ account_scope_id          # independent capacity unit (separate-by-default)
+ rate_limit_scope_id       # 429 cooldown unit
```

Add when required:

```text
+ region
+ account_alias
+ capability
```

Health AVAILABLE means:

> This exact cell is currently operationally eligible.

It does not mean:

- best quality
- best task fit
- cheapest
- fastest
- approved for all tasks
- approved for huge payloads
- approved for tools or vision unless proven

OSAI must preserve those distinctions.

## 5. Task packet

Use or extend the existing task packet instead of creating a parallel contract.

A task packet should carry only what downstream routing needs.

Recommended fields:

```yaml
task_id:
task_family:
user_intent:
input_modality:
required_capabilities:
request_shape:
estimated_payload_chars:
payload_size_bucket:
latency_priority:
quality_priority:
cost_policy:
privacy_policy:
tool_requirements:
browser_requirement:
vision_requirement:
streaming_requirement:
structured_output_requirement:
fallback_policy:
human_gate:
```

Do not inject the entire Board, repository, or unrelated project context into every request.

Pass targeted context only.

## 6. Payload discipline

Use the shared six-stage payload contract.

OSAI must inspect:

- total payload size
- largest message
- role-specific counts
- tool schema size
- history size
- duplicate context
- image count
- request shape

Do not store raw prompts to obtain these metrics.

### Huge payload policy

For `huge` payloads:

- do not automatically fail over across providers
- do not duplicate the same huge request
- prefer normalization, summarization, context pruning, or staged execution
- re-evaluate capability and cost after normalization
- require an explicit approved policy for any alternate send

A provider 503 is not permission to spray a huge request across routes.

## 7. Routing ownership

OSAI owns route choice using:

- Health current availability
- task capability requirements
- payload-size evidence
- request shape
- cost policy
- latency policy
- quality observations
- cooldown state
- account and credential restrictions
- user preference
- ticket-specific rules

Health facts are inputs, not final routing decisions.

OSAI must not bypass Health by selecting:

- disabled credentials
- stale-only cells when fresh proof is required
- active cooldown cells
- capability mismatches
- paid or unknown-cost routes without approval
- unproven huge-payload cells

## 8. Failover, fallback, and roaming

These are runtime decisions, but they must be bounded.

### Global default

Keep global same-request failover OFF unless an approved ticket explicitly changes it.

### Profile-specific policy

A profile may enable bounded behavior when:

- profile scope is explicit
- runtime activation is approved
- maximum attempts are bounded
- alternate route is Health AVAILABLE
- capability matches
- cost class is allowed
- payload is not huge
- same-key retry is prohibited
- active cooldown is excluded

Recommended baseline:

```yaml
max_requests: 2
max_alternate_attempts: 1
same_key_retry: false
same_provider_same_model_retry: false
paid_or_unknown_cost_excluded: true
capability_match_required: true
health_available_required: true
active_cooldown_excluded: true
huge_payload_automatic_failover: false
provider_only_grouping_prohibited: true        # never bulk-act on provider_id alone
key_slot_account_scope: separate_by_default     # share only via recorded exception
cooldown_unit: rate_limit_scope_id              # 429 cools the scope, not the provider
failure_unit: credential_identity_ref           # auth failure isolates to the credential
```

### Error handling

- 429: respect key/provider cooldown; alternate only when policy permits
- 503: at most one eligible alternate for non-huge payloads
- timeout: at most one eligible alternate for non-huge payloads
- auth failure: do not retry same credential
- capability mismatch: do not send
- request-shape mismatch: adapt only with a proven adapter
- payload too large: normalize before alternate routing

## 9. No-surface-error goal

The user-facing goal is to avoid unnecessary provider errors reaching the user.

However, do not hide serious failures by:

- uncontrolled retries
- repeated expensive sends
- capability downgrades without disclosure
- silent model substitution
- fabricated success
- empty placeholder answers

Use bounded alternate routing and normalized error handling.

If no safe route remains, return a clear sanitized failure.

## 10. Testcodex and GUI parity

For testcodex / Codex Desktop stability, distinguish:

- backend source readiness
- running proxy code
- runtime activation
- GUI behavior
- provider result
- final answer quality

Do not claim a source change is live until process state proves it.

Manual GUI Golden Tasks should cover:

- basic conversation
- multi-turn history
- file reading
- file editing
- tools
- image input
- browser or external web search
- source attribution
- structured output
- failure recovery
- fallback/failover behavior
- large payload behavior

A backend test pass is not GUI parity proof.

A provider request success is not final answer-quality proof.

## 11. Existing implementation first

Check existing OSAI modules before creating new paths.

Relevant areas may include:

- task packet schemas
- planner and route selector
- dispatcher
- one-shot actual path
- foundry proxy
- Responses / Chat Completions adapters
- loop guard
- command result
- event streaming
- latency tracking
- state checkpoint
- feedback and learning DB
- provider observation DB
- routing policy candidates
- Teacher Inbox renderer and data model
- OSAI Runtime Intelligence renderer

Reuse existing canonical ledgers and events.

Do not create a second routing history or learning ledger without explicit need.

## 12. Canonical telemetry

The canonical usage ledger is:

`logs/api_usage/events.jsonl`

Use canonical event kinds where applicable:

- `probe`
- `dispatch`
- `outcome`
- `feedback`
- `human_burden`
- `escalation_candidate`
- `canonical_update_candidate`
- `nightly_summary`

Do not treat route history or SQLite feedback as a replacement for the canonical event ledger unless the Board explicitly changes this.

Use sanitized event content.

## 13. Quality and suitability

Separate these concepts:

- operational availability
- task suitability
- answer quality
- latency
- cost
- reliability
- human burden
- safety

A model can be AVAILABLE but unsuitable for a task.

A model can be high quality but too expensive for the current cost policy.

A smoke reply is not a benchmark pass.

A benchmark pass is not automatic canonical adoption.

## 14. Benchmark and Judge

Benchmarks should use:

- reference dataset
- explicit task family
- deterministic scoring where possible
- rubric version
- route identity
- request shape
- payload-size bucket
- sanitized outcome
- evaluator identity
- confidence
- evidence reference

Self-scored results remain provisional.

High-risk routing and safety decisions require appropriate Teacher or human review before canonical adoption.

Do not let Health own Judge semantics.

## 15. Teacher Inbox

Teacher Inbox belongs to OSAI.

It may collect:

- failed answers
- repeated user dissatisfaction
- routing mistakes
- tool misunderstandings
- file-handling mistakes
- image or browser mistakes
- safety-boundary mistakes
- candidate canonical updates
- Eval candidates

Teacher Inbox should show:

- packet ID
- task family
- sanitized prompt summary
- candidate answer summary
- failure reason
- rubric
- sensitivity state
- evidence refs
- proposed action
- approval state

Do not store raw secrets, headers, tokens, or private third-party content.

## 16. OpenAI Evals integration

OpenAI Evals is an OSAI Teacher/Judge concern, not a Health concern.

Health may provide:

- credential availability
- complimentary weekly Eval eligibility signal
- usage evidence
- reset or remaining count when available

OSAI owns:

- packet selection
- sanitization
- rubric
- batching
- Eval creation
- run approval
- result ingestion
- learning event generation

### First-run gate

The first Eval run must require Yuya approval.

Recommended gate:

```yaml
manual_approval_required: true
approved_by: Yuya
max_runs: 1
contains_sensitive_data: false
sanitization_passed: true
complimentary_weekly_evals_confirmed: true
paid_overflow_allowed: false
```

Opening or refreshing a dashboard must not trigger an Eval run.

## 17. Learning boundary

OSAI may create:

- feedback events
- escalation candidates
- canonical update candidates
- nightly summaries
- task suitability updates
- route-quality observations

OSAI must not directly overwrite canonical policy from one model judgment.

Recommended flow:

```text
observation
→ sanitized learning event
→ candidate
→ review or threshold
→ canonical adoption
```

OSAI must never, using provider_id alone: bulk-cooldown, bulk-HOLD, bulk failure-propagate, or bulk-serialize a key_slot group; nor propagate one slot's 429/auth/5xx to a different-account slot; nor apply a permanent provider-wide brake on unknown scope. Separate-account slots are independent capacity. Every temporary brake carries expiry + review_due + graduation_condition.

Teacher Inbox is for bootstrap and exception review.

Normal bounded routing, fallback, roaming, and async learning enqueue should not require a human GO once their runtime policy is approved.

## 18. Feedback detection

OSAI may detect:

- explicit negative feedback
- repeated dissatisfaction
- correction loops
- abandonment
- repeated retry by the user
- “違う”, “わかりにくい”, “直して”, and similar patterns

Detection should create a sanitized feedback proposal.

It must not silently rewrite canonical behavior.

Avoid treating every short correction as a severe failure.

## 19. OSAI Runtime Intelligence dashboard

This dashboard may show:

- current runtime path
- selected route
- alternate candidates
- payload-size state
- capability state
- fallback/failover policy
- recent outcomes
- Teacher Inbox
- Eval approval
- learning events
- candidate policy updates
- human burden
- bottlenecks

The dashboard should own Teacher Inbox UI.

Health may keep only a compact link or eligibility summary.

Dashboard work must not block runtime integration.

## 20. Human gates

Ask Yuya before:

- paid or unknown-cost route activation
- first unknown provider/account/key use
- proxy start or restart
- production hot-path activation
- scheduler activation
- broad request-cap expansion
- secret changes
- raw storage policy changes
- first OpenAI Eval run
- automatic Eval scheduling
- canonical policy adoption when the ticket requires human approval
- destructive GRAVITY / BlueStacks actions

Do not ask Yuya for routine bounded routing decisions after the relevant runtime policy is approved.

## 21. Validation

Prefer:

- py_compile
- tracked focused tests
- adapter contract tests
- route-selection tests
- failover policy tests
- payload bucket boundary tests
- sanitized telemetry tests
- fixture rendering tests
- git diff --check
- secret leak checks

Tests should separately verify:

- global default OFF
- profile-specific policy disabled before approval
- bounded enable after approval
- max request count
- same-key retry prohibition
- paid route exclusion
- capability matching
- cooldown exclusion
- huge-payload suppression
- no raw content in metrics
- source-ready versus runtime-active reporting

Do not claim GUI readiness from unit tests alone.

## 22. OSAI completion report

Report:

- ticket
- task family
- files_read
- files_changed
- tests_result
- provider_send_count
- proxy_restart_count
- runtime_activation_state
- selected route behavior
- failover/fallback state
- payload instrumentation state
- GUI parity state
- Teacher Inbox impact
- learning events created
- incomplete
- remaining risks
- exact resume state
- safe next action

Use exact labels:

- `design_only`
- `source_ready`
- `runtime_disabled`
- `runtime_not_restarted`
- `live_unverified`
- `GUI_retest_required`
- `shadow_only`
- `candidate_only`
- `canonical_not_updated`

## 23. Final OSAI rule

OSAI owns task suitability, routing semantics, answer quality, Teacher/Judge interpretation, and learning.

Health owns operational availability evidence.

When in doubt:

- credential, quota, cooldown, model availability → Health
- task routing, answer quality, Teacher Inbox, learning → OSAI
- approved scope and current state → CURRENT_BOARD.md
