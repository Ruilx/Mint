# Mint — Prototype and Implementation Plan

> Status: Planning draft v0.1  
> This is a staged plan, not authorization to begin implementation. Stage details and acceptance criteria may be revised during planning.

## 1. Objective

Plan a first useful release of Mint as an independent self-hosted Web platform with a frontend, backend, MySQL persistence, tool definitions, task submission, and task/result viewing.

Mint must be able to accept and persist a task while no external consumer is online. A compatible consumer can later discover the task and update its state and result through the agreed database contract.

## 2. Planning Rules

- Confirm architecture and data contracts before implementing dependent features.
- Keep unresolved technology choices marked **TBD**.
- Do not make Mint dependent on any particular external task consumer.
- Build and validate the database task contract before relying on it for end-to-end operation.
- Avoid adding AI integration to the initial scope.
- Each stage should have explicit acceptance criteria and a review before the next stage begins.

## 3. Proposed Stages

### Stage 0 — Confirm the Project Baseline

**Purpose:** Resolve the decisions that affect the structure of the application before substantial implementation.

Work items:
- Confirm frontend framework and project structure.
- Evaluate the backend framework and whether Rhea is a suitable foundation.
- Evaluate the data-access approach and whether Pydoo is suitable.
- Confirm MySQL version within the required 8.0/8.1 range.
- Define authentication/session behavior and the initial permission model.
- Draft the first domain model and database migration approach.
- Define the task contract and state machine at a conceptual and schema level.
- Define the local development and test workflow.

Acceptance criteria:
- Major technology choices are recorded with reasons.
- Confirmed constraints are separated from candidates and TBD items.
- Initial entity relationships and task-state semantics are documented.
- No implementation depends on an undocumented external-consumer assumption.

### Stage 1 — Application Foundation

**Purpose:** Establish the runnable frontend/backend application and reliable persistence.

Work items:
- Create the project structure and configuration approach.
- Establish database connectivity and schema migrations.
- Add basic logging and consistent error handling.
- Establish automated tests and a repeatable local development process.
- Implement the minimum health/readiness behavior appropriate for the service.

Acceptance criteria:
- The application starts using documented steps.
- Database schema can be created or migrated reproducibly.
- Configuration does not require secrets to be committed.
- A basic automated test suite runs in the documented environment.

### Stage 2 — Accounts and Authorization

**Purpose:** Provide controlled access to Mint.

Work items:
- Implement sign-in/sign-out and session or token handling according to the chosen design.
- Implement administrator-managed user accounts; no public registration in the initial release.
- Define and enforce the initial role/permission model.
- Add backend authorization checks to protected operations.
- Ensure user-specific task and result access is isolated.

Acceptance criteria:
- An administrator can create and manage user accounts.
- Unauthenticated requests cannot access protected functionality.
- Unauthorized users cannot manage users or tools.
- A user cannot retrieve another user's task or result by changing an identifier.
- Authentication and authorization behavior is covered by tests.

### Stage 3 — Tool Definitions

**Purpose:** Allow authorized users to define the tools that Mint can present and use to create task requests.

Work items:
- Define tool metadata and input-schema representation.
- Implement create, read, update, and disable/archive behavior as selected during design.
- Validate submitted input against the stored schema on the backend.
- Define how a task refers to the applicable tool definition and how later edits affect previously created tasks.
- Implement tool browsing and input-form presentation in the frontend.

Acceptance criteria:
- Authorized users can manage tool definitions.
- Other permitted users can browse and submit valid requests.
- Invalid input is rejected by the backend with actionable error information.
- Previously submitted tasks remain interpretable if a tool definition changes, according to the chosen versioning/snapshot policy.
- Arbitrary uploaded executable scripts are not enabled by default.

### Stage 4 — Task Submission and Database Contract

**Purpose:** Implement Mint's producer-side task behavior and establish the generic contract for independent consumers.

Work items:
- Define the task schema, state machine, field semantics, and result/error formats.
- Implement task creation and persistence.
- Persist the initial pending state and validated input.
- Define atomic claim semantics for concurrent consumers, even though the claim operation may be implemented by consumers rather than Mint.
- Define legal state transitions and conditional updates.
- Define retry metadata, timeout representation, cancellation behavior, and stale-execution protections as required by the chosen contract.
- Define schema migration and compatibility expectations for independently upgraded consumers.
- Document the contract without tying it to any particular consumer implementation.

Acceptance criteria:
- Mint can persist a valid task while no consumer is running.
- The task remains visible as pending until its persisted state changes.
- Invalid requests do not create misleading executable tasks.
- The contract explains how consumers identify eligible work and report execution outcomes.
- Concurrent claim and stale-update risks are addressed in the design and tested where the Mint implementation participates.
- No direct function, method, or runtime API call to an external consumer is introduced.

### Stage 5 — Task History, Status, and Results

**Purpose:** Let users understand what happened to submitted tasks.

Work items:
- Implement task list and detail views.
- Display persisted status, timestamps, errors, and results.
- Define pagination, filtering, and sorting needs.
- Define how structured results and large artifact references are represented.
- Implement access-controlled retrieval of result artifacts where applicable.
- Decide retention and cleanup policies.

Acceptance criteria:
- Users can view only tasks and results they are authorized to access.
- The interface accurately reflects persisted state and does not invent execution progress.
- Failure information is understandable without exposing secrets.
- Large files are handled through controlled storage references rather than being placed indiscriminately in task rows.

### Stage 6 — Integration Validation and Operational Readiness

**Purpose:** Validate the full producer/consumer contract and prepare the initial deployment.

Work items:
- Use a minimal independent test consumer or contract-level tests to validate database interoperability; do not make that consumer a Mint runtime dependency.
- Test tasks remaining pending while no consumer is available.
- Test state changes, failures, retries, stale updates, and duplicate/concurrent claims according to the contract.
- Document deployment, configuration, database backup, restoration, and upgrade procedures.
- Review security, logs, error handling, and data retention.

Acceptance criteria:
- Mint can run independently of a consumer.
- A separate compatible consumer can process tasks using only the documented database contract.
- Contract incompatibilities are detectable through tests.
- Installation, upgrade, and recovery procedures are documented.
- Known limitations are recorded before declaring the initial release ready.

## 4. Deferred Work

Unless separately approved, defer:
- Unified AI/model/API integration.
- Public user registration.
- Arbitrary user-provided executable uploads.
- Consumer-specific integration code.
- Separate message-broker infrastructure.
- Advanced multi-tenant administration beyond the initial authorization needs.

## 5. Cross-cutting Risks

| Risk | Planning response |
|---|---|
| Mint accidentally depends on a specific consumer | Enforce the database-contract boundary in architecture review and tests |
| Concurrent consumers claim the same task | Design and validate atomic claiming and conditional state transitions |
| An old execution overwrites a newer attempt | Define attempt identity/versioning or fencing semantics in the task contract |
| Task state and side effects diverge | Define retry/idempotency expectations; do not promise universal exactly-once execution |
| Tool definitions change after task submission | Define snapshot/versioning semantics before implementation |
| Users access other users' data | Enforce backend authorization and test identifier tampering |
| Large files become difficult to share or secure | Define controlled storage, references, and permissions |
| Candidate dependencies constrain the architecture prematurely | Evaluate Rhea and Pydoo before adoption; keep alternatives explicit |
| Independent upgrades break interoperability | Document schema compatibility and migration rules |

## 6. Decisions to Resolve Before Implementation

1. Frontend framework and build tooling.
2. Backend framework and whether Rhea will be used.
3. Data-access layer and whether Pydoo will be used.
4. MySQL version, migration tool, and schema conventions.
5. Authentication/session model and roles.
6. Tool schema format and versioning policy.
7. Task schema and complete state machine.
8. Atomic claim, lease/timeout, retry, cancellation, and stale-update protocol.
9. Result/error format and artifact storage.
10. API conventions and error response format.
11. Test tooling and required CI checks.
12. Deployment, backup, restore, and upgrade strategy.

## 7. Stage Gate

Do not treat this plan as a mandate to implement every stage immediately. First review and revise this plan with the user. Implementation should begin only after the relevant design decisions are confirmed and the first implementation stage is explicitly agreed.