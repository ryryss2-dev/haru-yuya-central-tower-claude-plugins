---
name: central-tower-health-execution
description: Use this skill for Haru-Yuya Central Tower Health work: provider availability, credential slots, account and region mapping, model catalogs, bounded probes, quota and cooldown handling, request-shape and payload-size evidence, AVAILABLE lifecycle, reinspection queues, and Health dashboard evidence. Load together with central-tower-common-safety.
---

# Haru-Yuya Central Tower Health Execution Skill

## 1. Required companion skill

Always load:

`central-tower-common-safety`

This Health skill defines Health-specific responsibilities only.  
The common skill remains authoritative for workspace, Board, secrets, cost gates, hot-path gates, validation, and reporting.

## 2. Health mission

Health answers:

> Can this exact provider route be used now, under this credential, model, region, request shape, payload size, and capability?

Health does not answer:

- Is this the best model for the task?
- Was the answer high quality?
- Should this become canonical routing policy?
- Should Teacher Inbox adopt this result?
- Should OSAI learn from it?

Those belong to OSAI.

## 3. Canonical Health cell

The minimum canonical cell is:

```text
provider_id
+ credential_slot
+ model
+ request_shape
+ payload_size_bucket
```

Add dimensions when required:

```text
+ region
+ account_alias
+ capability
+ endpoint_family
```

For AWS Bedrock, use:

```text
provider_id
+ credential_slot
+ region
+ model
+ request_shape
+ payload_size_bucket
```

Do not merge distinct accounts, credential slots, regions, model aliases, or request shapes into one availability record.

## 4. Health-owned state

Health owns evidence and lifecycle for:

- credential presence
- credential enabled state
- account mapping
- region mapping
- authentication
- connectivity
- HTTP or provider status
- quota
- cooldown
- model availability
- model permission
- endpoint compatibility
- request shape
- payload-size bucket
- capability
- latency evidence
- sanitized operational failures
- AVAILABLE inventory
- temporary unavailable state
- reinspection queue
- stale evidence
- proof expiry

Health may publish facts to OSAI.

Health must not directly update OSAI canonical routing policy.

## 5. Health does not own answer quality

Do not classify these as Health success or failure:

- correctness
- usefulness
- writing quality
- task fit
- reasoning quality
- instruction following
- Teacher approval
- benchmark adoption
- canonical learning decision

A successful tiny-text reply proves only that the route handled that tested cell.

It does not prove:

- tool support
- vision support
- long context support
- browser support
- code quality
- routing suitability
- answer quality

## 6. Availability lifecycle

Use explicit lifecycle states.

Recommended states:

- `AVAILABLE`
- `TEMPORARILY_UNAVAILABLE`
- `COOLDOWN`
- `STALE`
- `REINSPECTION_QUEUED`
- `HOLD`
- `UNKNOWN`

### AVAILABLE

Use only when:

- credential is enabled
- capability matches
- proof is fresh enough
- current cooldown does not exclude the route
- tested request shape matches
- tested payload-size bucket matches
- no durable provider restriction is known

### Temporary failures

Do not convert these directly into permanent HOLD:

- 429
- 503
- timeout
- transient network failure
- stale evidence
- cooldown active
- cooldown recently ended
- request-shape mismatch
- payload-size mismatch
- temporary capability mismatch
- temporary model unavailability

Prefer:

- `TEMPORARILY_UNAVAILABLE`
- `COOLDOWN`
- `STALE`
- `REINSPECTION_QUEUED`

### HOLD

Use HOLD only for durable reasons, such as:

- invalid or revoked credential
- known unsupported account type
- model permanently unavailable for the account
- provider policy restriction
- unresolved identity ambiguity that makes sending unsafe
- explicit Yuya hold
- cost class not approved

Record the exact durable reason.

## 7. Failure-domain taxonomy

Classify failures into one primary domain:

- `transport`
- `credential`
- `quota`
- `cooldown`
- `request_shape`
- `payload_size`
- `tool_contract`
- `capability`
- `provider_policy`
- `model_availability`
- `region`
- `account_mapping`
- `unknown`

Do not use answer-quality categories as Health failure domains.

Recommended sanitized fields:

```yaml
provider_id:
credential_slot:
account_alias:
region:
model:
request_shape:
payload_size_bucket:
capability:
status:
sanitized_error_class:
failure_domain:
http_status:
latency_ms:
proof_generated_at:
proof_expires_at:
next_eligible_at:
reinspection_reason:
```

## 8. Payload-size evidence

Use the shared six-stage contract from the common skill:

- tiny
- small
- medium
- large
- xlarge
- huge

Do not infer that success in one bucket applies to another.

Examples:

- tiny text success does not prove huge text support
- non-tool success does not prove tool support
- text success does not prove vision support
- one region success does not prove another region
- one credential slot success does not prove another slot

## 9. Request-shape evidence

Use explicit request-shape labels.

Examples:

- `text_basic`
- `text_history`
- `text_tools`
- `vision`
- `vision_tools`
- `responses_api`
- `chat_completions`
- `streaming`
- `structured_output`
- `embedding`
- `rerank`

Do not collapse incompatible API shapes into one proof.

## 10. Probe policy

Use existing Health runners and transports before creating new ones.

Check relevant existing modules first, including:

- `run_central_api_health_check_one.py`
- `run_central_api_practical_prompt_one.py`
- `central_api_actual_consumer.py`
- `central_api_actual_transport.py`
- `provider_model_catalog.py`
- `provider_send_fn.py`
- `health_three_lane_schema.py`
- `three_lane_status_updater.py`
- `health_osai_ammunition_lifecycle.py`
- existing credential registry and provider command registry

### Bounded low-risk probe

A bounded actual probe may proceed without a fresh per-call Yuya confirmation when the ticket or manifest already approves:

- known provider
- known account
- known credential slot
- free, trial, or complimentary cost class
- bounded request count
- retry=0 unless specifically approved
- known model
- matching capability
- sanitized result only
- no proxy restart
- no scheduler
- no hot-path activation

### Default probe constraints

Unless the ticket says otherwise:

- one provider/model/credential cell at a time
- max one request per cell
- retry=0
- no same-key retry
- no fallback
- no failover
- no parallel requests on the same credential
- stop or cooldown on 429
- record 401/403 by credential slot
- use the smallest sufficient payload
- save sanitized evidence only

## 11. Cost and credential boundary

Ask Yuya before:

- paid execution
- unknown-cost execution
- new provider billing activation
- new trial enrollment
- credit card action
- secret creation
- secret replacement
- secret deletion
- credential remapping
- unknown account identity
- broad all-key crawl
- cap expansion
- scheduler activation

Do not reveal secret values.

Use aliases such as:

- `central:provider:api_key:1`
- `aws_bedrock_account_1`
- `nvidia_nim_key_2`

## 12. Cooldown and anti-ban behavior

429 and provider throttling must create bounded cooldown evidence.

Record:

- provider
- credential slot
- model
- status
- cooldown source
- cooldown start
- next eligible time when known
- retry count
- whether the cooldown is provider-wide, account-wide, key-wide, or model-specific

Do not repeatedly probe a route during active cooldown.

Do not run parallel requests through the same credential unless explicitly approved.

Temporary 429 or 503 must not become permanent exclusion automatically.

## 13. Reinspection queue

Reinspection entries should include:

```yaml
cell_id:
reason:
failure_domain:
queued_at:
next_eligible_at:
attempt_count:
max_attempts:
in_flight:
dedupe_key:
last_evidence_ref:
```

Requirements:

- dedupe equivalent cells
- prevent duplicate in-flight probes
- honor `next_eligible_at`
- do not create infinite retry loops
- preserve historical proof separately from current availability
- expired proof returns to reinspection; it is not silently treated as current

## 14. Historical proof versus current inventory

Keep these separate:

### Historical proof

Shows that a cell succeeded at a specific time.

### Current AVAILABLE inventory

Shows that the cell is currently eligible for OSAI consumption.

Historical success must not automatically remain AVAILABLE forever.

Proof TTL should be configurable.

When proof expires:

- mark stale
- enqueue reinspection when appropriate
- retain historical evidence
- do not erase the old proof

## 15. Provider-specific identity

Never merge provider identity based only on display name.

Track exact:

- provider ID
- endpoint
- account
- credential slot
- region
- requested model
- returned model
- substitution or fallback evidence

If requested and returned model differ, classify explicitly.

Examples:

- `exact_model_match`
- `fallback_or_substituted`
- `unknown_returned_model`

Do not certify a requested model when the provider substituted another model.

## 16. Bedrock handling

Bedrock must be tracked by:

- account
- credential slot
- region
- inference profile or model identifier
- request shape
- payload-size bucket

Do not assume browser Playground success proves CLI/API success.

Do not assume one region proves another.

Do not assume one account proves another.

Preferred current regions may be represented by Board policy, but Health evidence must remain per-region.

## 17. Health dashboard role

The Health dashboard may show:

- runner summary
- provider/account/key/model cells
- current AVAILABLE inventory
- cooldowns
- stale proofs
- reinspection queue
- replenishment state
- sanitized failures
- last proof
- next eligible time
- links to OSAI-owned Teacher Inbox or quality views

The Health dashboard must not become the primary owner of:

- Teacher Inbox
- rubric editing
- answer-quality review
- OSAI learning decisions
- canonical routing adoption

Existing Teacher Inbox UI may remain temporarily until migration is complete.

Do not delete it before the OSAI replacement is live and reviewed.

## 18. Dashboard non-blocking rule

Dashboard work must not block:

- credential onboarding
- provider communication
- model validation
- availability lifecycle
- reinspection
- OSAI integration

Prefer runtime and evidence correctness over dashboard polish.

## 19. Validation

Prefer:

- py_compile
- tracked focused tests
- schema validation
- fixture rendering tests
- JSON validation
- secret leak checks
- git diff --check

When actual probing is approved, record:

- provider_send_count
- exact cells attempted
- retry count
- result class
- cost class
- sanitized evidence path

Do not claim current availability from a fixture-only test.

## 20. Health completion report

Report:

- ticket
- provider/account/credential cells touched
- files_read
- files_changed
- tests_result
- actual provider send count
- proxy restart count
- probe cells
- current availability changes
- temporary failures
- permanent HOLD reasons
- reinspection entries
- sanitized evidence
- incomplete
- remaining risks
- exact resume state
- safe next action

Use exact labels such as:

- `fixture_only`
- `source_ready`
- `actual_probe_completed`
- `current_available`
- `historical_success_only`
- `temporary_unavailable`
- `stale`
- `reinspection_queued`
- `live_unverified`

## 21. Final Health rule

Health publishes trustworthy current availability evidence.

Health does not decide final task routing, answer quality, Teacher adoption, or learning policy.

When in doubt:

- operational availability fact → Health
- task suitability or quality judgment → OSAI
- approved scope and current state → CURRENT_BOARD.md
