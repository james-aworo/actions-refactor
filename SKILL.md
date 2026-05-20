---
name: actions-refactor
version: 0.1.3
description: |
  Refactors Actions code one controller flow at a time into the dedicated `domains/actions` package.

  Use when relocating Actions controllers, services, repositories, data classes, payloads,
  request/response models, mappers, validators, configs, and tests while preserving existing behavior.

  This skill is intentionally narrow. It performs a controller-first structural relocation and adds thin
  boundary wrappers for cross-domain calls. It does not perform full hexagonal architecture, business cleanup,
  authorization redesign, service splitting, repository redesign, DTO redesign, or naming cleanup.

triggers:
  - actions refactor
  - refactor actions
  - move actions controller
  - relocate actions
  - actions package move
  - actions wrapper extraction
  - actions adapter extraction
  - actions use case extraction
  - controller-first actions refactor
  - domains actions refactor

allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - MultiEdit
  - Glob
  - Grep
  - AskQuestion
  - Jetbrains
---

# actions-refactor

## 0. Role

You are a guarded Actions refactoring workflow controller.

Your job is to move, or verify as already moved, one Actions controller flow at a time into `backend/network/src/main/java/ai/prewave/dashboard/domains/actions` while preserving behavior and making dependency boundaries explicit.


This is not a general cleanup skill.
This is not a full hexagonal architecture skill.
This is not a naming-improvement skill.
This is not a “Cursor found a nicer pattern, so let it rewrite half the backend” skill.

The skill exists to make the Actions refactor safe, boring, reviewable, and reversible.

## 0.1 Existing-location guard

Before moving any controller, service, repository, or related Actions class, first check its current package and file path.

If the file is already under:

`backend/network/src/main/java/ai/prewave/dashboard/domains/actions`

or already declares the correct `ai.prewave.dashboard.domains.actions...` package, do not move it, rename it.

Treat already-correct placement as completed migration state and continue with the remaining workflow steps from that location.

This guard applies for the entire workflow lifecycle. Do not reconsider moving an already-correctly-placed file in later steps unless the user explicitly instructs otherwise.

Core rule:

```text
One selected controller flow at a time.
Preserve behavior.
Move known Actions-owned files.
Keep foreign files where they are.
Wrap cross-domain calls with dumb forwarding wrappers only.
Record anything uncertain or deferred in actions-refactor-review.jsonl.
```

Definitions:

```text
Actions-owned:
  A controller, service, repository, data object, mapper, validator, config, or test that belongs to the Actions feature based on the ownership map in this skill.

Foreign-domain dependency:
  A service, repository, evaluator, utility, or data type owned by another module/domain but used by Actions.

Outgoing dependency:
  Actions code calls a foreign-domain dependency.

Incoming dependency:
  Foreign-domain code calls an Actions-owned service, repository, or internal class.

Adapter:
  An Actions-owned wrapper for outgoing dependencies.
  It forwards exactly one method invocation per method.
  It contains no logic.

Use case:
  An Actions-owned wrapper for incoming dependencies.
  It forwards exactly one method invocation per method.
  It contains no logic.
```

---

## 1. Operating Principle

This refactor is scoped to the smallest safe unit: one controller flow.

The human selects one controller. The skill walks the graph from that controller, moves only the Actions-owned files reached from that flow, wraps only the cross-domain dependencies required by that flow, and records follow-up work as structured review tasks.

The workflow must optimize for:

- stable behavior,
- compile safety,
- small reviewable diffs,
- predictable package destinations,
- repeatability across multiple developers,
- useful audit output for later follow-up agents.

Do not try to finish the whole Actions module in one run unless explicitly instructed. Even then, prefer repeated controller-flow runs.
Ask the human for the specific entry point, which should be an Actions*Controller.

---

## 2. Target Package Layout

Use this package structure under the existing application root package.

Canonical logical layout:

```text
backend/network/src/main/java/ai/prewave/dashboard/domains/actions/controller
backend/network/src/main/java/ai/prewave/dashboard/domains/actions/service
backend/network/src/main/java/ai/prewave/dashboard/domains/actions/repository
backend/network/src/main/java/ai/prewave/dashboard/domains/actions/adapter
backend/network/src/main/java/ai/prewave/dashboard/domains/actions/usecase
backend/network/src/main/java/ai/prewave/dashboard/domains/actions/data
backend/network/src/main/java/ai/prewave/dashboard/domains/actions/mapper
backend/network/src/main/java/ai/prewave/dashboard/domains/actions/config
```

Rules:

```text
Controllers               -> domains/actions/controller
Actions services          -> domains/actions/service
Actions repositories      -> domains/actions/repository
Outgoing adapters         -> domains/actions/adapter
Incoming use cases        -> domains/actions/usecase
DTOs/payloads/data models -> domains/actions/data
Mappers                   -> domains/actions/mapper
Validators                -> domains/actions/service or domains/actions/validator if the package already exists
Configs                   -> domains/actions/config
Tests                     -> matching test package path
```

Use `data`, not `dto`.

Do not invent alternative package names.
Do not create `domain`, `application`, `infrastructure`, `port`, or `hexagonal` packages in this refactor.
Do not create temporary production packages like `_review`, `pending`, or `migration_review`.

---

## 3. Hard Non-Goals

Do not perform these changes during this skill:

```text
- no full hexagonal architecture
- no service splitting
- no repository redesign
- no query rewrite
- no DTO redesign
- no endpoint redesign
- no route changes
- no authorization refactor
- no @PreAuthorize expression changes
- no method/class/field/parameter renames
- no business logic cleanup
- no behavior changes
- no formatting churn
- no generated code movement
- no dependency or build-file changes unless required to restore compilation and explicitly approved
```

If any of these look useful, record a structured follow-up item in `actions-refactor-review.jsonl` instead of doing it.

The first pass should leave better boundaries, not a surprise architecture migration dressed as “cleanup”.

---

## 4. Required Human Input

Every run must start with one selected controller.

Required input shape:

```text
Controller: <ControllerName>
```

Useful optional input:

```text
Current package: <current.package>
Target package root: <target.root.package>
Run mode: audit_only | apply
```

If the selected controller is missing or ambiguous, ask one focused question:

```text
Which Actions controller should I refactor in this run? Give the class name, for example `ActionCommentController` or `ActionController`.
```

Do not proceed without a selected controller.

---

## 5. Controller-First Workflow

Follow this order exactly:

```text
Step 0: Bootstrap repository and run context
Step 1: Resolve selected controller
Step 2: Audit selected controller flow
Step 3: Classify direct dependencies
Step 4: Plan file moves and wrapper changes
Step 5: Stop unless apply mode is confirmed
Step 6: Move selected controller to domains/actions/controller
Step 7: Move reached Actions-owned data types to domains/actions/data
Step 8: Move reached Actions-owned services to domains/actions/service
Step 9: Move reached Actions-owned repositories to domains/actions/repository
Step 10: Move reached mappers/configs/validators/tests if clearly Actions-owned
Step 11: Move controller orchestration into Actions service when needed to enforce controller -> Actions service
Step 12: Add outgoing adapters for foreign calls from Actions services
Step 13: Search incoming usages for moved Actions services
Step 14: Add incoming use cases for foreign callers when needed
Step 15: Update imports and Spring injection
Step 16: Run compile/checks
Step 17: Write patch report and review tasks 
Step 18: Report result
```

Important boundary rule:

```text
Controllers must call Actions-owned services.
Controllers must not call adapters directly.
Adapters are used inside Actions services.
```

Correct flow:

```text
ActionController -> ActionService -> TargetServiceActionsAdapter -> TargetService
ActionController -> BulkActionService -> ActionStatusService -> ActionStatusRepository
ForeignService   -> ForeignDomainActionsUseCase -> ActionService
```

Incorrect flow:

```text
ActionController -> TargetServiceActionsAdapter -> TargetService
```

Do not place adapters at the controller boundary.

---

## 6. Bootstrap: Run First

Run this before auditing, editing, or writing reports.

```bash
set -euo pipefail

RUN_ID="$(date -u +%Y%m%dT%H%M%SZ)-$$"
REPO_ROOT="$(git rev-parse --show-toplevel 2>/dev/null || true)"

if [ -z "$REPO_ROOT" ]; then
  echo "BLOCKED: not_a_git_repository"
  exit 0
fi

cd "$REPO_ROOT"
REPO_ROOT="$(pwd -P)"
BRANCH="$(git branch --show-current 2>/dev/null || echo detached)"
HEAD_SHA="$(git rev-parse HEAD 2>/dev/null || echo unknown)"

STATE_DIR=".cursor/plans/actions-refactor/state"
AUDIT_DIR="$STATE_DIR/audits"
PATCH_DIR="$STATE_DIR/patches"
TMP_DIR="$STATE_DIR/tmp"
LOG_FILE="$STATE_DIR/runs.jsonl"
REVIEW_FILE="$STATE_DIR/actions-refactor-review.jsonl"

mkdir -p "$AUDIT_DIR" "$PATCH_DIR" "$TMP_DIR"
touch "$LOG_FILE"
touch "$REVIEW_FILE"

python3 - <<'PY'
import json
import os
from pathlib import Path
from datetime import datetime, timezone

event = {
  "ts": datetime.now(timezone.utc).strftime("%Y-%m-%dT%H:%M:%SZ"),
  "event": "started",
  "skill": "actions-refactor",
  "version": "0.1.0",
  "runId": os.environ.get("RUN_ID", "unknown"),
  "repoRoot": os.environ.get("REPO_ROOT", ""),
  "branch": os.environ.get("BRANCH", ""),
  "head": os.environ.get("HEAD_SHA", ""),
}
log_file = Path(os.environ.get("LOG_FILE", ".cursor/plans/actions-refactor/state/runs.jsonl"))
log_file.parent.mkdir(parents=True, exist_ok=True)
with log_file.open("a", encoding="utf-8") as fh:
    fh.write(json.dumps(event, separators=(",", ":")) + "\n")
PY

echo "RUN_ID=$RUN_ID"
echo "REPO_ROOT=$REPO_ROOT"
echo "BRANCH=$BRANCH"
echo "HEAD=$HEAD_SHA"
echo "STATE_DIR=$STATE_DIR"
echo "AUDIT_DIR=$AUDIT_DIR"
echo "PATCH_DIR=$PATCH_DIR"
echo "REVIEW_FILE=$REVIEW_FILE"
```

If bootstrap prints `BLOCKED`, stop and report the blocker.

Do not continue outside a Git repository.
Do not write state outside `.cursor/plans/actions-refactor/state` unless explicitly instructed.
Do not commit state files.

Expected ignored state path:

```text
.cursor/plans/actions-refactor/state
```

If the state path is not ignored, still continue, but warn the user before final response.

---

## 7. Actions Ownership Map

The ownership map is the source of truth for this skill.

Do not infer ownership from names when the map is explicit.
Do not treat non-Actions-looking names as foreign if this map or the human says they are Actions-owned.
This repository has legacy naming. Naming is evidence, not law.

### 7.1 Actions-Owned Controllers

```text
ActionCommentController
ActionController
ActionFileController
ActionHistoryController
ActionItemController
ActionLinkController
ActionRecommendationController
ActionStatusController
ActionTypeController
```

### 7.2 Actions-Owned Services

```text
ActionCommentService
ActionService
BulkActionService
ActionFileService
ActionHistoryService
ActionItemService
ActionLinkService
ActionRecommendationService
ActionRecommendationServiceNew
ActionStatusService
ActionTypeService
ActionEventTypeService
```

### 7.3 Actions-Owned Security / Permission Classes

```text
ActionPermissionEvaluator
```

Notes:

```text
ActionPermissionEvaluator is deprecated.
Preserve it and move it only if it is required by the selected flow.
Record deprecated usage in actions-refactor-review.jsonl.
Do not replace or remove it during this refactor.
```

### 7.4 Actions-Owned Repositories

```text
ActionCommentRepository
ActionRepository
ActionSkipRepository
ActionStatusRepository
ActionFileRepository
ActionHistoryRepository
ActionItemRepository
ActionLinkRepository
ActionRecommendationRepository
AlertActionRecommendationRepository
ActionRecommendationRepositoryNew
ActionTypeRepository
ActionEventTypeRepository
```

Important local ownership correction:

```text
Questionnaire<Suffix> used by Actions flow is NOT Actions-owned.
Incident<Suffix> used by Actions flow is NOT Actions-owned.
```

### 7.5 Actions-Owned Data / Payload / Response / Filter / Sort Types

Use this category for DTOs, payloads, request models, response models, filters, sorts, planner data, action view data, and similar shapes.

Known examples include:

```text
ActionCommentPayload
MinimalAction
ActionSort
ActionFilter
CreateActionPayload
CreateActionResult
CreateActionPayloadV2
DeleteActionResult
SkipActionPayload
SkipActionResult
SkipExistingActionPayload
UpdateActionPayload
UpdateActionPayloadV2
BulkUpdateActionsPayload
UpdateActionResult
ActionTarget
PlannerFilter
PlannerSort
PlannerFilterRange
ActionDashboardExcelPayload
ActionDashboardParameters
ActionHistory
ActionItemPayload
ExecutionActionItemPayload
ActionLinkPayload
TargetActionRecommendations
AlertRecommendation
ActionStatus
ActionType
ActionTypeGroup
RequestResponder
```

Pattern rules:

```text
Planner<Suffix> used by Actions planner endpoints is Actions-owned.
TargetActionRecommendations is Actions-owned.
ExecutionActionItemPayload is Actions-owned.
RequestResponder is Actions-owned.
<Prefix>Responder used in the Actions flow is Actions-owned unless code evidence proves otherwise.
```

### 7.6 Shared / Framework / Foreign Types That Must Not Be Moved Blindly

```text
UserData
Pageable
Authentication
MultipartFile
PWPeriodDTO
ReportFile
DSLContext
ApplicationEventPublisher
ObservationRegistry
FeatureFlagUtil
```

Rules:

```text
Framework types stay where they are.
Generated types stay where generated.
Shared platform/security/user types stay where they are.
Foreign-domain services and repositories stay where they are.
```

If one of these appears in an Actions controller or service, update imports only.
Do not move it into `domains/actions`.

---

## 8. Dependency-Type Decision Matrix

Use this matrix for every dependency reached from the selected controller flow.

### 8.1 Controllers

```text
If selected controller is Actions-owned:
  move to domains/actions/controller.
  preserve routes, annotations, method names, signatures, return types, and behavior.

If controller is not in the Actions-owned map:
  do not move.
  record a review item if it appears Actions-related.
```

### 8.2 Actions-Owned Services

```text
If service is Actions-owned and reached from selected controller:
  move to domains/actions/service.
  preserve constructor dependencies, methods, annotations, and behavior.
  then classify its dependencies.

If service is not Actions-owned:
  leave it in place.
  call it through an adapter when used from an Actions service.
```

### 8.3 Actions-Owned Repositories

```text
If repository is Actions-owned and reached from selected controller flow:
  move to domains/actions/repository.
  preserve queries and dependencies exactly.

If repository is foreign:
  leave it in place.
  call it through an adapter when used from an Actions service if wrapping is required by the boundary rule.

Never rewrite repository queries during this skill.
```

### 8.4 DTOs / Payloads / Response Models / Filters / Sorts / Data Types

```text
If type is Actions-owned:
  move to domains/actions/data.
  preserve fields, constructors, annotations, enum values, serialization annotations, and behavior exactly.

If type is shared/framework/foreign:
  leave it in place.
  update imports only.

If type has Jackson polymorphic config, custom serializer config, reflection-based mapping, or external contract usage:
  do not move blindly.
  record a review item unless the move is clearly safe and compile/tests cover it.
```

### 8.5 Mappers / Validators

```text
If mapper/validator is Actions-owned and reached from selected controller flow:
  move to domains/actions/mapper or the existing equivalent Actions package.

If mapper/validator is shared or foreign:
  leave it in place.
  wrap only if it is a foreign service-like dependency called from Actions service code and wrapping is required.
```

### 8.6 Configs

```text
If config is Actions-specific:
  move to domains/actions/config.

If config is global/shared/platform config:
  do not move.
```

### 8.7 Tests

```text
If test directly targets moved Actions controller/service/repository/data:
  move or update matching test package path.
  preserve test intent and assertions.

Do not rewrite tests for style.
Do not loosen assertions to make tests pass.
Do not delete tests.
```

### 8.8 Security / Permission Evaluators

```text
If evaluator is Actions-owned:
  move only if reached and required.
  preserve behavior and annotations.

If evaluator is foreign:
  leave in place.
  wrap only if called from Actions service code and required by the outgoing adapter rule.

Do not change @PreAuthorize expressions.
Do not move authorization expressions into services.
Do not introduce new authorization services.
```

### 8.9 Framework Types

```text
Never move framework types.
Examples:
  Pageable
  Authentication
  MultipartFile
  ApplicationEventPublisher
  ObservationRegistry
  DSLContext
```

Use imports only.

### 8.10 Generated jOOQ Types

```text
Never move generated jOOQ tables, records, POJOs, keys, or generated source files.
```

If generated types are used by moved repositories, update imports only.

### 8.11 Events / Publishers

```text
Do not wrap ApplicationEventPublisher by default.
Do not redesign domain events.
Do not move shared event types unless explicitly Actions-owned.
```

If event type ownership is unclear, record a review item.

---

## 9. Controller Boundary Rule

Controllers in `domains/actions/controller` should call Actions-owned services, not foreign-domain services.

When an Actions controller endpoint currently calls both an Actions service and foreign services, move the orchestration into the Actions service if this can be done without changing endpoint behavior.

Example before:

```kotlin
@GetMapping("{id}")
fun getActionById(
    @PathVariable id: Int,
    @ActiveUser user: UserData
): Action = actionService.getActionById(id, user)
    .also { action -> targetMetaService.appendEntityTargetMetadata(action, { listOf(it.target) }, user.org()) }
    .also { action -> targetService.fetchParents(listOf(action.target), user.customerId, user.organizationId) }
```

Example after:

```kotlin
@GetMapping("{id}")
fun getActionById(
    @PathVariable id: Int,
    @ActiveUser user: UserData
): Action = actionService.getActionById(id, user)
```

Then inside `ActionService`, preserve the same behavior using adapters:

```kotlin
fun getActionById(id: Int, user: UserData): Action =
    actionRepository.getActionById(id, user)
        .also { action ->
            targetMetaServiceActionsAdapter.appendEntityTargetMetadata(
                action,
                { listOf(it.target) },
                user.org()
            )
        }
        .also { action ->
            targetServiceActionsAdapter.fetchParents(
                listOf(action.target),
                user.customerId,
                user.organizationId
            )
        }
```

Rules:

```text
- Preserve controller route and method signature.
- Preserve returned response shape.
- Preserve ordering of side effects.
- Preserve @PreAuthorize and custom annotations.
- Do not rename `getActionById`, even if it now enriches data.
- If the method name becomes misleading, record a follow-up item.
```

If moving controller orchestration into the service requires uncertain behavior changes, stop that specific transformation and record a review item.

---

## 10. Outgoing Adapter Rule

When Actions service code calls a foreign-domain service, repository, evaluator, or domain utility, create or reuse an adapter inside `domains/actions/adapter`.

Adapter naming convention:

```text
<ForeignDependencyName>ActionsAdapter
```

Examples:

```text
TargetServiceActionsAdapter
TargetMetaServiceActionsAdapter
ReportFileServiceActionsAdapter
UserJpaRepositoryActionsAdapter
IncidentServiceActionsAdapter
PermissionServiceActionsAdapter
```

Use one adapter per foreign dependency class.
Do not create one adapter per call site.
Do not create one adapter per method.
Do not create vague adapters like `TargetActionsAdapter` if the team wants one wrapper per dependency class.

Adapter method rule:

```text
Each adapter method must contain exactly one forwarded method invocation.
No logic.
No branching.
No validation.
No transformation.
No fallback.
No filtering.
No sorting.
No mapping.
No logging unless already present in the original call and explicitly required.
```

Allowed adapter shape:

```kotlin
@Component
class TargetServiceActionsAdapter(
    private val targetService: TargetService
) {
    fun fetchParents(targets: List<Target>, customerId: Int, organizationId: Int) =
        targetService.fetchParents(targets, customerId, organizationId)
}
```

Forbidden adapter shape:

```kotlin
@Component
class TargetServiceActionsAdapter(
    private val targetService: TargetService
) {
    fun fetchParents(targets: List<Target>, customerId: Int, organizationId: Int) {
        if (targets.isNotEmpty()) {
            targetService.fetchParents(targets, customerId, organizationId)
        }
    }
}
```

That is logic. Do not do it.

Before creating an adapter:

```text
1. Search domains/actions/adapter for an existing adapter for that exact foreign dependency.
2. If it exists, reuse it and add only the missing forwarding method.
3. If it does not exist, create it using the naming convention.
4. Replace foreign dependency injection in Actions service with adapter injection.
5. Replace direct foreign calls with adapter calls.
```

---

## 11. Incoming Use Case Rule

After moving an Actions-owned service reached from the selected controller flow, search for usages outside `domains/actions`.

Scope rule:

```text
Only scan incoming usages for Actions-owned services reached from the selected controller flow.
Do not globally rewrite unrelated Actions services.
```

When foreign-domain code calls an Actions-owned service directly:

```text
foreign code -> Actions use case -> Actions service
```

Create the use case inside:

```text
domains/actions/usecase
```

Use case naming convention:

```text
<ForeignDomainName>ActionsUseCase
```

If the foreign domain name is unclear, use the owning package/folder name as evidence. If still unclear, ask the human or record a review item.

Use case method rule:

```text
Each use case method must contain exactly one forwarded method invocation.
No logic.
No branching.
No validation.
No transformation.
No fallback.
No filtering.
No sorting.
No mapping.
```

Allowed use case shape:

```kotlin
@Component
class AlertActionsUseCase(
    private val actionService: ActionService
) {
    fun updateActionStatus(actionId: Int, statusId: Int, user: UserData) =
        actionService.updateActionStatus(actionId, statusId, user)
}
```

When foreign code is changed to call an Actions use case:

```text
- preserve behavior
- keep the call signature equivalent
- avoid unrelated edits in the foreign package
- record the touched foreign class/package in actions-refactor-review.jsonl
- mark owner/team review as required
```

Do not create incoming use cases for calls already inside `domains/actions`.

---

## 12. Shared Dependency and Merge Guardrails

Before moving any data type, service, repository, mapper, validator, adapter, or use case, search usages.

If the symbol is shared by multiple Actions controllers or services:

```text
- move it only to the canonical package
- preserve name, fields, methods, annotations, and behavior exactly
- do not create an alternative copy
- do not rename it
- do not split it
- do not inline it
- update imports only
```

If another developer already moved the same symbol to the canonical package:

```text
- reuse the existing moved file
- update imports to the canonical location
- do not create a duplicate
```

For wrappers:

```text
- search before creating
- reuse existing adapter/use case for the same dependency/domain
- add only missing forwarding methods
- do not create duplicate wrappers with similar names
```

Deterministic names matter. Git conflicts are annoying enough without two agents inventing `TargetAdapter`, `TargetActionsAdapter`, and `TargetServiceProxy` in three branches like a naming committee from hell.

---

## 13. Audit JSON

Before editing application files, write audit JSON to:

```text
.cursor/plans/actions-refactor/state/audits/<RUN_ID>.audit.json
```

Shape:

```json
{
  "runId": "string",
  "timestamp": "ISO-8601",
  "request": {
    "raw": "string",
    "selectedController": "string",
    "requestedAction": "audit_only | apply"
  },
  "repository": {
    "root": "string",
    "branch": "string",
    "head": "string",
    "dirty": false
  },
  "skill": {
    "name": "actions-refactor",
    "version": "0.1.0"
  },
  "scope": {
    "controller": "string",
    "currentPackage": "string",
    "targetPackage": "string",
    "targetRoot": "domains/actions",
    "confidence": "high | medium | low"
  },
  "dependencies": {
    "actionsOwned": ["string"],
    "foreign": ["string"],
    "ambiguous": ["string"],
    "incomingUsagesToCheck": ["string"]
  },
  "plannedChanges": {
    "moves": [
      {
        "symbol": "string",
        "from": "string",
        "to": "string",
        "type": "controller | service | repository | data | mapper | validator | config | test"
      }
    ],
    "outgoingAdapters": [
      {
        "foreignDependency": "string",
        "adapter": "string",
        "reason": "string"
      }
    ],
    "incomingUseCases": [
      {
        "foreignCaller": "string",
        "useCase": "string",
        "actionsService": "string",
        "reason": "string"
      }
    ],
    "controllerOrchestrationMoves": [
      {
        "controllerMethod": "string",
        "targetServiceMethod": "string",
        "reason": "controller currently calls foreign dependencies directly"
      }
    ]
  },
  "classification": {
    "risk": "LOW_RISK | MEDIUM_RISK | HIGH_RISK | BLOCKED",
    "allowedToApply": false,
    "requiresBehaviorChange": false,
    "requiresRename": false,
    "requiresAuthorizationChange": false,
    "requiresRepositoryRewrite": false,
    "requiresBuildChange": false,
    "requiresForeignOwnerReview": false
  },
  "decision": {
    "summary": "string",
    "reasons": ["string"],
    "stopReason": "string | null",
    "nextSafeAction": "string"
  }
}
```

### 13.1 allowedToApply Rule

Set `allowedToApply=true` only when:

```text
risk != BLOCKED
AND scope.confidence == high
AND selected controller is Actions-owned
AND planned moves have canonical package destinations
AND no planned rename exists
AND no planned behavior change exists
AND no authorization change exists
AND no repository query rewrite exists
AND no build/dependency change exists
AND all wrappers are dumb forwarding wrappers
```

Every other case is `allowedToApply=false`.

---

## 14. Stop Gate

After writing audit JSON:

```text
IF allowedToApply != true:
  report audit result
  do not edit application files
  stop
```

The stop gate is not advice.
It is the guardrail.

Allowed stop reasons:

```text
not_a_git_repository
selected_controller_missing
selected_controller_not_found
selected_controller_not_actions_owned
ownership_ambiguous
requires_behavior_change
requires_authorization_change
requires_repository_rewrite
requires_build_change
wrapper_would_need_logic
target_package_not_scanned
existing_uncommitted_changes_conflict
```

---

## 15. Apply Rules

When applying, use the smallest behavior-preserving edit.

Rules:

```text
- preserve package-private behavior where possible
- preserve annotations exactly
- preserve method names exactly
- preserve method signatures exactly
- preserve constructor parameter names where possible
- preserve return types exactly
- preserve endpoint paths exactly
- preserve request/response contracts exactly
- preserve side-effect ordering
- preserve @Lazy, @Value, @Transactional, @PreAuthorize, @Cacheable, @Async, and custom annotations
- preserve comments unless they become invalid because of moved package references
```

Do not reformat unrelated code.
Do not optimize imports outside changed files.
Do not reorder constructor dependencies unless required by tooling and harmless.
Do not replace Kotlin expression bodies with block bodies unless required to move existing orchestration safely.

---

## 16. Controller Orchestration Move Rule

If an Actions controller method calls foreign dependencies directly, move that orchestration into an Actions-owned service only when all are true:

```text
- the controller already calls an Actions-owned service in the same endpoint, or an obvious Actions-owned service owns that endpoint behavior
- the moved logic can be preserved exactly
- no route/signature/return-type change is required
- no authorization annotation change is required
- foreign calls can be replaced by adapters inside the service
```

Do not rename the service method.

If the method name becomes less precise after moving orchestration, record a follow-up item in `actions-refactor-review.jsonl`.

Review item example:

```json
{
  "id": "ARF-0001",
  "type": "method_naming",
  "status": "open",
  "severity": "low",
  "affectedClass": "domains/actions/service/ActionService.kt",
  "relatedController": "domains/actions/controller/ActionController.kt",
  "summary": "Review ActionService.getActionById naming after target enrichment was moved from controller",
  "currentBehavior": "ActionService.getActionById retrieves the action and enriches it with target metadata and parent target information.",
  "reasonDeferred": "The first refactor pass preserves method names, signatures, and behavior.",
  "recommendedFollowUp": "Evaluate whether the method should be renamed or split in a later cleanup pass without changing the API response contract.",
  "acceptanceCriteria": [
    "Method name or structure reflects behavior",
    "Tests updated if method structure changes",
    "No endpoint path or response contract change without approval"
  ],
  "reviewOwners": [
    "actions"
  ],
  "createdBySkill": "actions-refactor",
  "createdAt": "ISO-8601"
}
```

---

## 17. Review Task Output

Use machine-readable JSONL for follow-up tasks.

File:

```text
actions-refactor-review.jsonl
```

Each line must be one valid JSON object.
Do not write Markdown headings into this file.
Do not write comments into this file.
Do not write trailing commas.
Do not overwrite previous lines. Append only.

Review task shape:

```json
{
  "id": "ARF-0001",
  "type": "ambiguous_ownership | deferred_cleanup | method_naming | deprecated_dependency | foreign_owner_review | remaining_coupling | wrapper_followup | test_gap | scan_gap",
  "status": "open",
  "severity": "low | medium | high",
  "affectedClass": "string",
  "affectedPackage": "string",
  "relatedController": "string",
  "relatedService": "string | null",
  "summary": "string",
  "currentBehavior": "string",
  "reasonDeferred": "string",
  "recommendedFollowUp": "string",
  "acceptanceCriteria": ["string"],
  "reviewOwners": ["string"],
  "createdBySkill": "actions-refactor",
  "createdAt": "ISO-8601"
}
```

A good review task should read like a mini Jira ticket.
A future human or AI agent should be able to execute it without reconstructing the whole refactor from archaeology and regret.

Write review tasks for:

```text
- ambiguous ownership
- foreign package/class touched for incoming use case
- deprecated ActionPermissionEvaluator usage
- misleading method names preserved intentionally
- remaining direct foreign dependencies not wrapped because unclear
- repository ownership concerns deferred
- tests that could not be moved or run
- component scanning uncertainty
- package-private/protected visibility issues
- any behavior-risk item intentionally not changed
```

---

## 18. Incoming Foreign Owner Review Task

When this skill changes code outside `domains/actions`, append a review task.

Required fields:

```json
{
  "type": "foreign_owner_review",
  "severity": "medium",
  "affectedClass": "foreign class path",
  "affectedPackage": "foreign package",
  "relatedController": "selected controller",
  "relatedService": "Actions service now behind use case",
  "summary": "Foreign caller was changed to use an Actions use case wrapper.",
  "currentBehavior": "The foreign class previously called an Actions-owned service directly. It now calls an Actions-owned use case that forwards to the same service method.",
  "reasonDeferred": "Owner/team review is required because the foreign package was touched even though behavior should be unchanged.",
  "recommendedFollowUp": "Ask the owning team to review the call-site change and confirm the boundary wrapper is acceptable.",
  "acceptanceCriteria": [
    "Foreign owner confirms the call-site change is acceptable",
    "Behavior remains unchanged",
    "No additional foreign-domain refactor is bundled into this PR"
  ],
  "reviewOwners": ["foreign-domain-owner", "actions"]
}
```

---

## 19. Verification

After applying changes, run the strongest available lightweight checks without changing project configuration.

Recommended check order:

```text
1. git diff --stat
2. git diff --check
3. compile command if known
4. targeted tests for moved controller/service/repository if known
5. broader backend test command only if practical and already available
```

Useful commands:

```bash
git diff --stat
git diff --check
```

For Gradle projects, inspect available tasks before running broad commands:

```bash
./gradlew tasks --all | grep -E "compile|test|check" | head -100
```

Do not install dependencies.
Do not add scripts.
Do not modify Gradle files unless explicitly required and approved.
Do not hide failing checks.

If checks fail:

```text
- determine whether failure is caused by this refactor
- if caused by this refactor, fix if mechanical and within scope
- if not mechanical, stop and report
- if unrelated existing failure, report evidence and continue only if compile/import safety is otherwise clear
```

---

## 20. Diff Safety Validation

After editing, validate:

```text
- selected controller moved to canonical package
- Actions-owned reached services moved to canonical package
- Actions-owned reached repositories moved to canonical package
- Actions-owned reached data types moved to canonical package
- old imports updated
- no duplicate active controllers with same route remain
- no duplicate wrappers for same dependency/domain created
- adapters contain no logic
- use cases contain no logic
- routes unchanged
- annotations unchanged
- method signatures unchanged
- DTO fields unchanged
- repository queries unchanged
- no generated files moved
- no unrelated formatting churn
```

Suspicious diff terms that require extra inspection:

```text
rename
TODO
FIXME
hasAuthority
hasPermission
@PreAuthorize
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@Transactional
@Async
@Cacheable
DSLContext
package.json
build.gradle
gradle.lockfile
```

Suspicious does not mean forbidden. It means inspect and confirm the change is still within scope.

---

## 21. Patch Report

Write patch report to:

```text
.cursor/plans/actions-refactor/state/patches/<RUN_ID>.patch.json
```

Shape:

```json
{
  "runId": "string",
  "timestamp": "ISO-8601",
  "auditPath": "string",
  "reviewFile": "actions-refactor-review.jsonl",
  "patch": {
    "branch": "string",
    "selectedController": "string",
    "filesChanged": ["string"],
    "moves": ["string"],
    "adaptersCreatedOrUpdated": ["string"],
    "useCasesCreatedOrUpdated": ["string"],
    "summary": "string",
    "diffStat": "string"
  },
  "checks": {
    "commandsRun": ["string"],
    "passed": true,
    "skipped": false,
    "notes": ["string"]
  },
  "reviewTasks": {
    "created": 0,
    "ids": ["string"]
  }
}
```

Append completion event to `runs.jsonl`.

---

## 22. Final Response Format

Keep final response short.
Do not dump audit JSON to Human.

### 22.1 Audit Only / Not Applied

```text
Audit result: <LOW_RISK | MEDIUM_RISK | HIGH_RISK | BLOCKED>
Applied: no
Selected controller: <ControllerName>
Reason: <specific reason>
Audit saved: <path>
Next safe action: <specific next step>
```

### 22.2 Applied

```text
Audit result: MEDIUM_RISK
Applied: yes
Selected controller: <ControllerName>
Summary: <what changed>
Adapters: <created/updated/skipped>
Use cases: <created/updated/skipped>
Checks: <passed/skipped/failed with command names>
Audit saved: <path>
Patch report: <path>
Review tasks: <count written to actions-refactor-review.jsonl>
```

If foreign owner review is required, include:

```text
Owner review needed: yes, see actions-refactor-review.jsonl
```

---

## 23. Examples

### 23.1 Example: `ActionCommentController`

Selected controller:

```text
ActionCommentController
```

Expected traversal:

```text
ActionCommentController
  -> ActionCommentService
    -> ActionCommentRepository
```

Expected moves:

```text
ActionCommentController -> domains/actions/controller
ActionCommentService    -> domains/actions/service
ActionCommentRepository -> domains/actions/repository
ActionCommentPayload    -> domains/actions/data
```

Expected behavior:

```text
- routes unchanged
- @PreAuthorize unchanged
- method signatures unchanged
- service methods unchanged
- repository queries unchanged
```

### 23.2 Example: `ActionController` with foreign controller orchestration

Before:

```kotlin
fun getActionById(id: Int, user: UserData): Action = actionService.getActionById(id, user)
    .also { action -> targetMetaService.appendEntityTargetMetadata(action, { listOf(it.target) }, user.org()) }
    .also { action -> targetService.fetchParents(listOf(action.target), user.customerId, user.organizationId) }
```

After:

```kotlin
fun getActionById(id: Int, user: UserData): Action =
    actionService.getActionById(id, user)
```

Inside `ActionService`:

```kotlin
fun getActionById(id: Int, user: UserData): Action =
    actionRepository.getActionById(id, user)
        .also { action -> targetMetaServiceActionsAdapter.appendEntityTargetMetadata(action, { listOf(it.target) }, user.org()) }
        .also { action -> targetServiceActionsAdapter.fetchParents(listOf(action.target), user.customerId, user.organizationId) }
```

Adapters:

```kotlin
@Component
class TargetMetaServiceActionsAdapter(
    private val targetMetaService: TargetMetaService
) {
    fun appendEntityTargetMetadata(...) =
        targetMetaService.appendEntityTargetMetadata(...)
}
```

```kotlin
@Component
class TargetServiceActionsAdapter(
    private val targetService: TargetService
) {
    fun fetchParents(...) =
        targetService.fetchParents(...)
}
```

No method rename.
No endpoint change.
No annotation change.
No response change.

### 23.3 Example: Foreign caller into Actions

Before:

```kotlin
class SomeForeignService(
    private val actionService: ActionService
) {
    fun doWork(actionId: Int) =
        actionService.someMethod(actionId)
}
```

After:

```kotlin
class SomeForeignService(
    private val someForeignDomainActionsUseCase: SomeForeignDomainActionsUseCase
) {
    fun doWork(actionId: Int) =
        someForeignDomainActionsUseCase.someMethod(actionId)
}
```

Use case:

```kotlin
@Component
class SomeForeignDomainActionsUseCase(
    private val actionService: ActionService
) {
    fun someMethod(actionId: Int) =
        actionService.someMethod(actionId)
}
```

Also append a `foreign_owner_review` task to `actions-refactor-review.jsonl`.

---

## 24. Stop Conditions

Stop immediately if:

```text
- selected controller cannot be found
- selected controller is not Actions-owned
- target package root cannot be determined
- move would require behavior change
- adapter/use case would require logic
- repository query rewrite is required
- authorization expression change is required
- generated code would need to move
- Spring component scan would not include target package and the fix is unclear
- package-private/protected visibility breaks and the fix is not mechanical
- existing uncommitted changes conflict with planned edits
```

When stopped, write audit JSON and report the blocker.

If useful, append a structured review task.

---

## 25. Maintenance Note

This skill is intentionally explicit and repetitive because the refactor touches legacy service boundaries, security annotations, repositories, and controller behavior.

The first version should stay in one `SKILL.md` file so the team can review and adjust the workflow without chasing behavior across helper scripts. Later, stable scripts may be extracted for:

```text
- ownership-map validation
- audit JSON creation
- review JSONL validation
- adapter/use-case linting
- wrapper logic detection
- repeated usage scans
```

Team: Super Compliance
Maintainer: James

For workflow changes:

```text
Update the relevant section only.
Do not weaken the stop gates to make one awkward case pass.
Do not expand scope without changing the skill version and documenting the new behavior.
```
