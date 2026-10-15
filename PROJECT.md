# Mint — Project Definition

> Status: Planning draft v0.1  
> This document defines Mint itself. Decisions marked **TBD** are intentionally open and must not be treated as implementation requirements.

## 1. Project Overview

Mint is a self-hosted Web platform for defining tools, accepting user requests, creating persistent offline tasks, and presenting task status, history, and results.

Mint consists of a frontend and a backend. It is designed to operate independently of any particular offline task-processing program.

### 1.1 Goals

- Provide a Web interface for users to discover and use available tools.
- Let authorized users define and manage tool metadata, input schemas, and presentation-related configuration.
- Validate requests and persist tasks in MySQL.
- Let users inspect task status, execution results, and task history according to their permissions.
- Define a stable, database-mediated task contract that an independently implemented consumer can follow.
- Keep the initial deployment straightforward to operate while allowing future extension.

### 1.2 Non-goals

Mint does not need to:

- Implement the runtime that executes every kind of offline task.
- Manage the processes, worker pools, or internal queues of an external task consumer.
- Call an external consumer's functions, methods, or runtime API.
- Depend on one particular task consumer, programming language, or internal scheduling implementation.
- Include AI/model integration in the initial scope. A unified AI service may be designed later.
- Accept arbitrary user-uploaded scripts for execution in the initial release.

## 2. Scope and Users

### 2.1 Initial operating context

- Self-hosted deployment, initially intended for a small intranet user group (approximately 5–20 users).
- The architecture should avoid unnecessary operational complexity while not preventing future expansion.
- No public user registration in the initial release.
- Administrators create and manage user accounts.
- Users can access only the tasks, history, and results permitted by the authorization model.

### 2.2 Main user capabilities

- Sign in and sign out.
- Browse available tools and view their descriptions and input requirements.
- Submit a task through a tool.
- Inspect task status, history, and available results.
- Manage tool definitions when authorized.
- Perform account and access management when authorized as an administrator.

The exact role model and permission granularity are **TBD**.

## 3. System Boundaries

Mint is composed of the following parts:

1. **Frontend** — user-facing interaction and presentation.
2. **Backend** — authentication, authorization, business rules, tool definitions, task creation, task queries, and result access.
3. **MySQL** — persistence for Mint's application data and the database-mediated task handoff contract.
4. **External task consumers** — independent programs that may read eligible tasks and write execution states and results according to the agreed contract. They are integration counterparts, not Mint modules.

Mint must remain functional when no external consumer is running. A task can be accepted and persisted while remaining pending until a compatible consumer processes it.

### 3.1 Frontend responsibilities

- Present the available tools and their input forms.
- Validate inputs for usability; the backend remains authoritative for validation.
- Submit requests and display task identifiers and status.
- Present task history and results according to backend responses.
- Provide administrative interfaces for authorized users.

The frontend must not connect directly to MySQL or rely on consumer-specific behavior.

### 3.2 Backend responsibilities

- Authenticate users and authorize every protected operation.
- Maintain user and tool-definition records.
- Validate task submissions against the stored tool definition and input schema.
- Create and persist task records.
- Query task status, history, and results.
- Enforce user-level data isolation and administrative permissions.
- Validate data read from or written to the task contract where applicable.
- Manage references to files and other result artifacts.

The backend does not execute offline tasks merely because a task was submitted.

### 3.3 External task-consumer boundary

Mint defines the data contract; it does not define or implement a particular consumer.

The integration contract is expressed through shared MySQL schema, field semantics, state transitions, and result formats. It must not require shared runtime code or direct function, method, or API calls between Mint and a consumer.

Consumers may be implemented independently, provided they follow the contract. Consumer process management, scheduling, retries, and runtime recovery are outside Mint's internal responsibilities, although their observable task-state behavior must be reflected in the contract.

## 4. Architectural Principles

### 4.1 Independence

Mint must be deployable and usable without an external task consumer being online. Task submission must not require an active consumer connection.

### 4.2 Database-mediated task handoff

MySQL task tables are the sole task communication and queueing channel between Mint and external consumers. Mint persists a task; a consumer discovers and claims eligible work through the agreed database protocol; the consumer records execution state and results through that protocol.

### 4.3 Explicit contracts

Task fields, state semantics, allowed transitions, result formats, error representation, and schema evolution must be documented rather than inferred from implementation details.

### 4.4 Least privilege and data isolation

Every operation must enforce authorization on the backend. Users must not gain access to other users' tasks or results merely by changing an identifier.

### 4.5 Small, testable components

Keep frontend presentation, backend business logic, database access, and task-contract logic sufficiently separated to test independently. Avoid introducing infrastructure solely for hypothetical scale.

### 4.6 Decisions are explicit

Distinguish confirmed constraints from candidates and unresolved questions. An implementation agent must not silently turn a candidate into a binding decision.

## 5. Data and Domain Concepts

The following are conceptual entities; exact tables and relationships are **TBD**.

- **User** — an account and its authorization attributes.
- **Tool Definition** — a tool's identity, description, input schema, and other metadata required to present and validate a request.
- **Task** — a persisted request to perform work, including ownership, input, lifecycle state, timestamps, and relevant execution metadata.
- **Task Result** — structured output, error information, or references to result artifacts.
- **File/Artifact Reference** — metadata pointing to a controlled storage location for larger files.

Tool definitions describe how a request is represented and validated. They do not imply that Mint itself executes the underlying operation.

## 6. Task Lifecycle and Database Contract

The task lifecycle and its exact schema must be designed before implementation.

The design must address at least:

- Task identity and ownership.
- Tool identity and the version or snapshot of the definition used for submission.
- Validated input representation.
- Initial pending state.
- Execution state, completion state, and failure information.
- Creation, start, update, and completion timestamps as appropriate.
- Result representation and artifact references.
- Atomic claiming when multiple consumers may operate concurrently.
- Rules for retries, timeouts, cancellation, and recovery.
- Protection against stale consumers overwriting newer task state.
- Schema compatibility when Mint and consumers are upgraded independently.

State names and transition rules are **TBD**. Do not assume that a basic status column alone provides safe concurrent claiming or exactly-once side effects.

Task execution may be repeated by a consumer. Where duplicate side effects matter, the task contract and the relevant integration must define idempotency or deduplication behavior; Mint must not promise universal exactly-once execution.

## 7. Storage

- MySQL stores application data, task records, task state, and suitable result metadata.
- Large files should not be stored directly in ordinary task rows. Use controlled file or shared storage and persist references and access metadata in MySQL.
- Storage implementation, path conventions, retention, cleanup, and access-control details are **TBD**.
- If external consumers need to access artifacts, the storage and permission contract must make that access explicit.

## 8. Technology Constraints and Candidates

### Confirmed constraints

- Database family/version range: MySQL 8.0 or 8.1.
- Backend is a Python Web API service.
- Frontend and backend are both part of Mint.
- No direct runtime calls between Mint and an external task consumer.

### Candidates, not decisions

- **Rhea** may be evaluated as a starting point for the Web API layer.
- **Pydoo** may be evaluated as a data-access helper if its transaction, locking, and connection behavior fit Mint's requirements.
- Frontend framework, API conventions, migration tooling, packaging, and deployment method remain **TBD**.

Do not add a candidate dependency before evaluating whether it is necessary and suitable.

## 9. Security and Operational Requirements

The design must address:

- Secure authentication and password/session handling.
- Backend authorization for every protected operation.
- Per-user task and result isolation.
- Safe validation of tool inputs and output data.
- Controlled access to files and artifacts.
- Secret and configuration management.
- Logging and error reporting without unnecessarily exposing sensitive data.
- Database backup and recovery expectations.
- A deployment and upgrade procedure appropriate for a self-hosted service.

Detailed policies and implementation choices are **TBD**.

## 10. Deferred Capabilities

The following are intentionally not part of the initial implementation unless separately approved:

- Unified AI model/API integration.
- Public self-service registration.
- Arbitrary user-uploaded executable scripts.
- Consumer-specific runtime integrations.
- Infrastructure such as a separate message broker, unless later requirements justify it.

## 11. Open Decisions

At minimum, resolve these before the relevant implementation stage:

1. Frontend framework and application structure.
2. Backend framework and whether to build on Rhea.
3. Data-access approach and whether Pydoo is appropriate.
4. Authentication/session design and role/permission model.
5. Tool-definition schema and validation rules.
6. Task schema, lifecycle state machine, and schema migration strategy.
7. Atomic claim and concurrent-consumer protocol.
8. Retry, timeout, cancellation, and stale-execution handling.
9. Result and error format.
10. File storage, access, retention, and cleanup.
11. API design, testing strategy, deployment, and backup/restore process.

## 12. Change Policy

This is a planning baseline, not a frozen implementation specification. During planning, decisions may be revised through explicit discussion. Once a constraint is marked confirmed, an implementation agent must not change it silently; it must explain the conflict and request a decision.

Changes to Mint's scope or architecture should update this document and any affected plans or agent instructions.
