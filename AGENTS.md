# Mint — AI Agent Collaboration Rules

> Status: Planning draft v0.1  
> These rules apply to AI coding agents working in the Mint repository.

## 1. Primary Rule

Understand the request, repository state, and relevant project documents before making changes. Optimize for correctness, maintainability, and consistency with confirmed Mint constraints—not for the shortest implementation.

## 2. Source of Truth

Use the following precedence when guidance conflicts:

1. The user's latest explicit instruction.
2. Confirmed constraints and decisions in Mint project documents.
3. Existing code and tests, where they do not conflict with a confirmed requirement.
4. Candidate designs, examples, and agent assumptions.

Do not present an assumption or candidate as a confirmed decision.

If an instruction conflicts with a confirmed constraint, stop before making the conflicting change, explain the conflict, and ask for clarification.

## 3. Planning and Clarification

- For architecture-level work, first identify the affected responsibilities, data contracts, and dependencies.
- Separate confirmed requirements, proposed solutions, and unresolved questions.
- Ask before making a decision that materially changes scope, architecture, security, persistence semantics, or compatibility.
- Prefer the smallest design that meets the stated requirements.
- Do not introduce a dependency or infrastructure component without a concrete reason.
- During planning, propose revisions freely, but record the rationale and affected documents.

## 4. Mint Boundary Rules

- Treat Mint as an independent Web platform containing frontend and backend.
- Do not make Mint depend on a specific external task consumer.
- Do not add consumer-specific function, method, runtime, or API calls to Mint.
- The task handoff between Mint and external consumers is mediated by the agreed MySQL schema and protocol.
- Keep the contract explicit: document field meanings, state transitions, result formats, and compatibility expectations.
- Do not implement offline task execution inside Mint unless the user explicitly changes the project scope.
- Do not assume a consumer is online when accepting or persisting a task.
- Do not claim exactly-once execution merely because a task has one final success state.

## 5. Code Change Discipline

Before editing:

1. Inspect the relevant files and current conventions.
2. Read the applicable project and feature documentation.
3. Identify the smallest coherent change.
4. Check whether the change affects data schema, persisted records, API compatibility, security, or user-visible behavior.

While editing:

- Keep responsibilities separated and code understandable.
- Avoid unrelated refactors.
- Do not silently remove behavior, configuration, tests, or documentation.
- Do not add placeholders that appear functional but are not implemented.
- Do not hardcode secrets, credentials, machine-specific paths, or deployment assumptions.
- Validate all untrusted inputs on the backend.
- Enforce authorization on every protected backend operation.
- Treat task IDs and file references as untrusted inputs; verify ownership and permission before returning data.
- Use transactions and concurrency controls where correctness depends on multiple database operations.
- Keep schema changes explicit and provide a migration or documented migration procedure when appropriate.

## 6. Testing and Verification

- Add or update tests for changed behavior.
- Test validation failures and authorization boundaries, not only successful paths.
- For task-related changes, consider duplicate submission, concurrent consumers, stale updates, retries, and interrupted state transitions where relevant.
- Do not claim tests passed unless they were actually run and their outcome is known.
- If tests cannot be run, state what was not run and why.
- Keep test scope proportional to the change, but do not skip tests for persistence or security behavior without explanation.

## 7. Documentation Discipline

- Update documentation when behavior, configuration, schema, or architecture changes.
- Keep `PROJECT.md` focused on Mint's purpose, scope, architecture, and constraints.
- Keep `AGENT.md` focused on collaboration and engineering rules.
- Keep `prototype_plan.md` focused on implementation stages, dependencies, decisions, and acceptance criteria.
- Do not duplicate large sections across documents; link conceptually by naming the relevant document or section.
- Mark decisions as **Confirmed**, **Candidate**, **TBD**, or **Deferred** where useful.
- Do not add documentation about a particular external consumer as though it were part of Mint's own architecture. Describe only the generic contract Mint requires.

## 8. Dependencies and Technology Choices

- MySQL must remain within the confirmed version range: 8.0 or 8.1.
- Rhea and Pydoo are candidates, not mandatory dependencies.
- Evaluate existing code and dependency behavior before recommending adoption.
- Avoid adding packages that duplicate standard-library or existing project capabilities without a clear benefit.
- Do not select a frontend framework, migration system, authentication design, or deployment stack by assumption if it is still marked TBD.

## 9. Security and Privacy

- Do not expose credentials, tokens, passwords, or secrets in source code, logs, tests, or documentation.
- Do not weaken authorization to make a feature easier to implement.
- Ensure users cannot access another user's tasks or results by guessing or changing identifiers.
- Handle uploaded or generated files through controlled storage and explicit access checks.
- Report security-relevant uncertainty rather than concealing it behind a permissive default.

## 10. Completion Report

At the end of a work item, summarize:

- What changed and why.
- Important design decisions and assumptions.
- Tests and checks run, including outcomes.
- Known limitations or unresolved questions.
- Documentation or migration steps needed.

Keep the report factual. Do not claim completion of work that remains partial.

## 11. Scope Changes

The user owns the project's confirmed constraints. Do not modify or delete a confirmed constraint merely to make the current implementation easier. If a constraint appears impractical, explain the evidence, propose alternatives, and wait for the user's decision.

Planning documents may evolve during planning. Implementation should follow the latest confirmed decisions and explicitly record any approved changes.

## 12. Conversation Language

Reply in the language used by the user.

