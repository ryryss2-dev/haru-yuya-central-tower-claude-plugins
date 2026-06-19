---
name: central-tower-common-safety
description: Common execution and safety rules for all Haru-Yuya Central Tower / OSAI / Health / Provider Intelligence / GRAVITY work. Load this skill together with the task-specific Health or OSAI skill. It defines canonical workspace, Board discipline, secret hygiene, approval boundaries, validation, reporting, and scope-control rules.
---

# Haru-Yuya Central Tower Common Safety Skill

## 1. Purpose

This skill contains only the rules shared by all Central Tower workers.

Load this skill together with one task-specific skill:

- `central-tower-health-execution`
- `osai-runtime-learning-execution`

Do not use this common skill as a substitute for the task-specific skill.

## 2. Core behavior

Work in Japanese.

Be concise, but preserve enough detail for another AI to resume safely.

Report in this order:

1. 結論
2. 根拠
3. 次アクション

Do not guess silently.

When something is unclear, first classify the ambiguity.

### Resolve mechanically without asking Yuya

Proceed with low-risk investigation when the ambiguity can be resolved using:

- targeted source inspection
- existing schemas
- fixtures
- tracked tests
- sanitized metadata
- existing Board records
- read-only local commands
- known provider documentation already present in the repository

### Stop and ask Yuya only when required

Ask Yuya when the decision affects:

- paid or unknown-cost execution
- credit card, trial enrollment, purchase, or billing
- secret or credential creation, replacement, deletion, or remapping
- ambiguous provider/account/key identity
- production hot-path activation
- proxy start or restart
- scheduler or launchctl activation
- broad actual crawl or cap expansion
- raw storage policy
- destructive file or repository operations
- high-risk GRAVITY / BlueStacks state-changing actions

Do not return routine mechanical decisions to Yuya when they can be resolved by code, tests, guards, or existing canonical records.

## 3. Workspace

Use this workspace root:

`/Users/yuya/Scripts/BlueStacksGravityLauncher`

Canonical proxy root:

`tools/continue_foundry_proxy`

Do not treat the following as authoritative:

- `/Users/yuya/Scripts/continue-foundry-proxy`
- old compatibility roots
- copied or archived proxy trees
- temporary extraction directories

Before work:

- confirm the workspace root
- identify the exact ticket
- classify the task as Health, OSAI, or GRAVITY
- load this common skill
- load the matching task-specific skill
- read only the relevant Board section

## 4. Source of truth

`CURRENT_BOARD.md` is the single operational source of truth.

Do not treat these as canonical unless the Board explicitly points to them:

- `AI_COMMAND_BOARD.md`
- `COMMAND_BOARD.md`
- old handoff files
- generated previews
- stale dashboards
- chat summaries
- archived artifacts

Use targeted `rg`, `sed`, or equivalent inspection.

Do not read the entire Board unless Yuya explicitly approves or the task genuinely requires a whole-board consistency audit.

Prefer:

- ticket header
- status
- exact_resume_state
- blockers
- acceptance_criteria
- files_changed
- tests_result
- safe_next_action

Do not overwrite the whole Board.

Do not paste huge logs into the Board.

Do not create duplicate `exact_resume_state`, `safe_next_action`, or status blocks.

## 5. Scope discipline

Use small, bounded goals.

A valid task should define:

- workspace
- task classification
- ticket
- read-first files
- exact task
- allowed changes
- forbidden operations
- validation
- completion report

If the requested scope is too broad, propose a split before implementation.

Do not expand from:

- investigation into implementation
- implementation into runtime activation
- local tests into provider send
- source changes into proxy restart
- preview work into production integration
- one credential slot into broad fleet execution

without an explicit scope decision.

## 6. Existing implementation first

Before creating new code:

1. search for existing implementation
2. inspect the current canonical path
3. reuse existing schemas, guards, ledgers, and helpers
4. document why reuse was not possible if creating a new path

Do not create parallel runners, duplicate ledgers, duplicate bucket definitions, or duplicate policy modules without a concrete reason.

Prefer pure shared helpers for cross-lane constants and schemas.

Avoid importing a large runtime module solely to reuse one constant if that creates:

- circular imports
- import-time file I/O
- network access
- process startup
- expensive initialization
- hot-path latency

## 7. Secret and privacy hygiene

Never display, save, log, or commit:

- API key values
- Authorization headers
- Bearer tokens
- raw request headers
- raw response headers
- raw provider responses
- raw prompt bodies
- `.env` contents
- Keychain values
- `os.environ` contents
- access tokens
- refresh tokens
- passwords
- private account identifiers that are not needed for operation
- third-party private conversations
- personal or family data unless explicitly approved and sanitized

Allowed identifiers include:

- provider ID
- model name
- key slot alias
- credential lane
- region
- account alias
- source reference
- sanitized error class
- status code
- latency bucket
- payload size bucket
- request shape
- capability labels

Use sanitized summaries and irreversible counts.

Do not save message fingerprints when raw text may contain secrets unless the design explicitly guarantees:

- request-local use only
- no persistent hash storage
- no cross-request tracking
- no reversible encoding

## 8. Canonical payload-size contract

Use one shared six-stage payload-size definition:

- `tiny`: 0–2,000 chars
- `small`: 2,001–8,000 chars
- `medium`: 8,001–16,000 chars
- `large`: 16,001–32,000 chars
- `xlarge`: 32,001–64,000 chars
- `huge`: 64,001+ chars

Do not create a competing four-stage or provider-specific bucket definition unless the canonical schema is intentionally revised.

Use the existing shared helper when safe.

If the existing helper creates unsafe imports or side effects, report the issue and propose moving only the pure constant/helper into a lightweight shared module.

Do not make the move without scope approval when it affects multiple subsystems.

## 9. Cost and actual-execution boundary

Do not treat every low-risk actual request as requiring a fresh Yuya GO.

A bounded actual request may proceed without another per-request confirmation only when all of the following are already approved by the relevant ticket or manifest:

- provider is known
- account and credential slot mapping are known
- cost class is `free`, `trial`, or `complimentary`
- request count is bounded
- retry policy is bounded
- model and capability match are known
- secret handling is already established
- sanitized evidence only
- no scheduler activation
- no production hot-path activation
- no scope expansion

Explicit Yuya GO is required for:

- paid execution
- unknown-cost execution
- first use of an unknown provider/account/key mapping
- broad batch actual crawl
- retry or request-cap expansion
- scheduler registration
- proxy start or restart
- hot-path connection
- secret creation, replacement, deletion, or remapping
- raw storage policy change
- destructive operation

A previous approval applies only to its stated scope.

Do not infer permission for a broader provider, model, key slot, region, request shape, or payload class.

## 10. Runtime and hot-path safety

Treat these as production or hot-path changes:

- route selection defaults
- failover or fallback defaults
- provider-send behavior
- max request counts
- retry behavior
- credential resolution
- proxy request transformation
- automatic dispatch
- scheduler behavior
- dashboard actions that trigger execution

A source change is not active merely because the file was edited.

Before claiming a runtime change is effective, confirm:

- running process
- process start time
- source modification time
- reload support
- active configuration
- restart requirement

Do not say “takes effect on next request” unless auto-reload is proven.

Do not restart a proxy without explicit Yuya approval.

Dashboard open, refresh, polling, or preview rendering must never trigger:

- provider send
- Eval run
- scheduler activation
- credential mutation
- destructive action

## 11. Failover and fallback baseline

Global failover defaults must remain conservative unless an approved ticket says otherwise.

Baseline:

- global default: OFF
- profile-specific enablement: explicit and bounded
- maximum alternate attempts: bounded
- same-key retry: prohibited unless explicitly approved
- paid or unknown-cost alternate routes: excluded
- capability match: required
- credential enabled: required
- Health availability: required
- active cooldown routes: excluded
- huge payload automatic failover: prohibited by default

A temporary provider error is not permission to spray the same request across providers.

Do not automatically forward `huge` payloads to alternate providers.

Normalize or reduce the payload first, then reassess using a bounded policy.

## 12. Temporary failure handling

Do not convert temporary failures into permanent HOLD without evidence.

Temporary classes include:

- 429
- 503
- timeout
- stale evidence
- cooldown active
- cooldown recently ended
- transient network failure
- request-shape mismatch
- payload-size mismatch
- tool-contract mismatch
- temporary capability mismatch

Classify failure domain separately:

- transport
- credential
- quota
- cooldown
- request_shape
- payload_size
- tool_contract
- capability
- provider policy

Answer quality is not a Health transport failure.

Use reinspection, cooldown, or temporary non-AVAILABLE states where appropriate.

Permanent HOLD requires a durable reason.

## 13. Health and OSAI ownership boundary

### Health owns facts about whether a route can currently be used

Examples:

- credential presence and status
- connectivity
- authentication
- quota and cooldown
- model availability
- region
- request shape
- payload-size bucket
- capability
- sanitized operational evidence
- AVAILABLE / temporary unavailable / reinspection

### OSAI owns how and why a route should be used

Examples:

- task suitability
- routing
- fallback and roaming semantics
- benchmark interpretation
- answer quality
- Teacher Inbox
- rubrics
- Judge results
- learning events
- canonical update candidates
- final policy decisions

Health may supply evidence to OSAI.

Health must not directly rewrite OSAI canonical policy.

OSAI must not treat a Health smoke success as proof of answer quality.

## 14. Dashboard boundary

Dashboard work must not block the main runtime path.

A dashboard may:

- render sanitized evidence
- expose status
- link to the owning subsystem
- show approval gates
- show queue and lifecycle state

A dashboard is not the source of truth unless the Board explicitly says so.

UI completeness is not an acceptance gate for:

- provider communication
- routing
- fallback
- failover
- roaming
- learning
- ingestion
- telemetry

Do not spend a critical implementation lane on cosmetic dashboard work while the runtime path is incomplete.

## 15. Validation

Prefer:

- `py_compile`
- targeted tracked tests
- schema validation
- `json.tool`
- `git diff --check`
- secret leak checks
- deterministic fixtures

Do not create untracked throwaway test files when an existing tracked suite is appropriate.

Do not weaken assertions merely to make tests pass.

If a broader suite has a failure:

- record the exact command
- record the failing test name
- record the assertion or exception
- classify relation to the current change as `true`, `false`, or `unknown`
- provide evidence before calling it pre-existing

If full tests contain live network calls, hanging tests, destructive operations, or uncontrolled provider sends, do not run them.

Explain why and run the relevant focused tests instead.

## 16. Secret leak checks

After API, probe, benchmark, routing, adapter, or provider-send code changes, search for markers such as:

- `sk-`
- `AIza`
- `Bearer`
- `Authorization`
- `x-goog-api-key`
- `api_key_value`
- `access_token`
- `refresh_token`
- `raw_response`
- `raw_headers`
- `raw_prompt`
- `full_prompt`

A text match is not automatically a leak.

Classify each match as:

- actual secret
- sanitized fixture
- test marker
- forbidden-word explanation
- false positive

Do not expose a real secret while reporting the check.

## 17. Forbidden without explicit Yuya GO

- paid or unknown-cost API execution
- unapproved actual provider send
- broad all-key actual crawl
- scheduler or launchctl registration
- proxy start or restart
- route-selector or provider-send hot-path activation
- retry-cap expansion
- request-cap expansion
- raw storage policy change
- secret creation, replacement, deletion, or remapping
- git add
- git commit
- git push
- `git add .`
- git reset
- git stash
- file deletion
- symlink creation
- QNAP synchronization
- purchase, payment, or charge operations
- GRAVITY / BlueStacks / ADB / tap / input / sendevent state-changing operation
- automatic Eval execution before its explicit gate
- automatic canonical-policy overwrite

## 18. Board update

After accepted work, update `CURRENT_BOARD.md` briefly and accurately.

Include:

- status
- files_read
- files_changed
- tests_result
- judgment
- incomplete
- remaining_risks
- exact_resume_state
- safe_next_action

When a prior claim was wrong, correct it explicitly.

Examples:

- `prior_claim_incorrect`
- `source_state_corrected`
- `hot_path_change_attempted`
- `hot_path_change_reverted`
- `runtime_not_restarted`
- `live_unverified`

Do not leave contradictory old blocks in the same ticket.

Do not claim a patch is applied merely because it was generated.

## 19. Report format

Include:

- task classification
- ticket
- files_read
- files_changed
- tests_result
- actual provider send count
- proxy restart count
- forbidden operations check
- judgment
- incomplete
- remaining risks
- exact resume state
- safe next action

When available, include:

- model name
- reasoning level
- token usage

If unavailable, write `不明`.

## 20. Completion truthfulness

Use exact states.

Good:

- `source_ready`
- `runtime_not_restarted`
- `live_unverified`
- `design_complete`
- `tests_passed`
- `shadow_only`
- `queued`
- `blocked_on_explicit_approval`

Avoid vague claims such as:

- complete
- production ready
- fixed
- active
- verified

unless the required runtime and evidence conditions are actually satisfied.

## 21. Goal discipline

A good goal contains:

- Workspace
- Task classification
- Ticket
- Read first
- Task
- Allowed changes
- Forbidden operations
- Validation
- Return format

Do not create broad goals such as:

- finish Central Tower
- complete OSAI
- fix all Health issues
- make everything production ready

Split broad work into bounded tickets.

## 22. Final common rule

This common skill defines shared safety and execution discipline only.

When a rule conflicts with a more specific approved ticket:

1. follow the approved ticket within its exact scope
2. preserve secret, cost, destructive-operation, and hot-path gates
3. record the exception in `CURRENT_BOARD.md`
4. do not generalize the exception to other tickets

When Health and OSAI disagree:

- Health is authoritative for current operational availability evidence
- OSAI is authoritative for task suitability, quality, and learning decisions
- `CURRENT_BOARD.md` is authoritative for approved scope and current execution state
