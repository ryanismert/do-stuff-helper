# Software Engineering Reference Guide

Reference material for waypoint-design when the waypoint involves building software. Not every section applies to every waypoint — use the complexity assessment to judge relevance.

---

## 1. Architecture Design Approach

### Explore Before Designing

- Read existing code before proposing changes. Understand current patterns, conventions, and constraints.
- Trace data flow end-to-end through the system. Identify where the waypoint's changes touch existing paths.
- Map the dependency graph: what does this component depend on, and what depends on it?

### Clarify Constraints

Before designing, identify:
- **Hard constraints** — must use existing database, must work on specific platforms, must maintain backward compatibility
- **Soft constraints** — team preferences, performance targets, style conventions
- **Non-constraints** — assumptions that seem like constraints but aren't (challenge these)

### Make Decisive Choices

- The design document should make decisions, not present options. "We will use X because Y" — not "we could use X or Y."
- When multiple approaches are viable, pick the one that's simplest to implement correctly and document why alternatives were rejected.
- If a decision genuinely requires user input, call it out as a specific question — don't hide it in a list of options.

### Component Design

- Each component should have a single, clear responsibility expressible in one sentence.
- Define the boundary of each component: what it owns, what it delegates, and what it refuses.
- Identify shared state and eliminate it where possible. Where sharing is necessary, make the ownership model explicit.

### Build Sequence

- Order implementation by risk, not by ease. Build the hardest, most uncertain piece first — if it fails, you learn early.
- Identify the minimum subset that proves the architecture works (the "walking skeleton"). Build that first.
- Defer optimization, polish, and edge cases to later tasks unless they affect architectural decisions.

---

## 2. Technology Decision Framework

### Default to the Existing Stack

- Use what's already in the project unless there's a compelling reason not to.
- "Compelling" means: the existing tool can't do the job, or using it would require workarounds that create more complexity than a new dependency.
- Adding a dependency has ongoing costs: updates, security patches, API changes, cognitive load. Justify each one.

### Decision Criteria Checklist

When evaluating a technology choice, consider:
- **Fitness** — does it solve the actual problem, not a superset of the problem?
- **Maturity** — is it stable enough for production use? Active maintenance?
- **Simplicity** — how much does the team need to learn? What's the debugging story?
- **Integration** — how well does it fit with existing tools, patterns, and deployment?
- **Escape hatch** — if this choice is wrong, how hard is it to switch?

### Document Rejected Alternatives

For significant technology decisions, briefly note:
- What alternatives were considered
- Why each was rejected (one sentence each)
- What would change the decision (conditions under which to revisit)

---

## 3. Data Model & API Design

### Data Model First

- Design the data model before the API or UI. The data model is the foundation — everything else adapts to it.
- Ask: what are the entities? What are the relationships? What are the constraints?
- Draw it out: even a text-based entity list with relationships is better than jumping to code.

### Make Illegal States Unrepresentable

- Use types and structure to prevent invalid data, rather than runtime validation.
- If a field can only have three values, use an enum — not a string with validation.
- If two fields are always present together or always absent together, group them into a single optional object.
- If a record has a status field that determines which other fields are valid, consider using discriminated unions or separate types per status.

### Explicit Error Cases

- Every operation that can fail should have its failure modes enumerated in the design.
- Distinguish between errors the caller can handle (validation failures, not-found) and errors that indicate bugs (assertion failures, impossible states).
- Design error responses that carry enough context for the caller to act. "Not found" is insufficient; "User 123 not found" is actionable.

### Narrow Interfaces

- Expose the minimum surface area needed. Functions should accept the narrowest types that work and return the most specific types possible.
- Prefer small, composable functions over large, configurable ones.
- Design APIs around use cases, not around data structures. The question is "what does the caller need to do?" — not "what does the data look like?"

### Schema Specification

When the waypoint defines data structures:
- Specify required vs. optional fields
- Define valid ranges, formats, and constraints
- Include example instances that demonstrate typical and edge-case data
- Note migration strategy if modifying existing structures

---

## 4. Testing Strategy

### Behavioral Coverage Over Line Coverage

- Tests should verify behavior ("when X happens, Y should result"), not implementation details.
- If a refactor that preserves behavior breaks tests, those tests were testing the wrong thing.
- A feature is tested when its acceptance criteria have corresponding test cases.

### Test Pyramid

- **Unit tests** — fast, isolated, cover logic and edge cases. The bulk of the test suite.
- **Integration tests** — verify components work together. Cover the critical paths.
- **End-to-end tests** — verify the system works from the user's perspective. Expensive; use sparingly for the most important flows.

### Critical Gap Identification

When designing tests, specifically look for:
- **Untested error paths** — what happens when the database is down, the API returns 500, the file doesn't exist?
- **Missing edge cases** — empty inputs, maximum values, concurrent access, malformed data
- **Absent negative tests** — tests that verify the system correctly rejects invalid inputs
- **Unverified state transitions** — do tests cover the full lifecycle, not just the happy path?

### Test Design Principles

- **DAMP over DRY** — tests should be Descriptive And Meaningful Prose. Duplicating setup code is fine if it makes each test self-contained and readable.
- Each test should test one thing. If a test name requires "and", split it.
- Test names should describe the scenario and expected outcome: "returns_empty_list_when_no_items_match_filter"

### Criticality Rating

Rate testing importance for each component on a 1-10 scale:
- **8-10**: Data integrity, financial calculations, authentication, authorization. Comprehensive coverage required.
- **5-7**: Core business logic, API contracts, state management. Solid coverage with edge cases.
- **1-4**: UI layout, logging, admin tooling. Basic smoke tests sufficient.

Use this rating to allocate testing effort in the task plan.

---

## 5. Error Handling

### Zero Tolerance for Silent Failures

- Every catch block must do something meaningful: log, rethrow, return an error, or recover.
- Never catch and ignore. Never catch and log without rethrowing if the caller needs to know.
- If an error is expected and handled, document why the handling is correct.

### Catch Block Specificity

- Catch the most specific error type possible. Catching `Error` or `Exception` at the top level masks bugs.
- Different error types often need different handling. A network timeout and a validation error are not the same thing.
- If you can't handle an error meaningfully at the current level, let it propagate.

### Actionable Error Messages

- Error messages should answer: what happened, what was the system trying to do, and what can be done about it.
- Include context: IDs, file paths, input values (redact sensitive data).
- Write error messages for the person who will debug the problem at 2 AM.

### Justified Fallbacks

- Every fallback or default behavior should have a comment explaining why it's safe.
- Fallbacks that hide failures are bugs waiting to happen. If a config file is missing, falling back to defaults might be fine for development but catastrophic in production.
- Prefer failing loudly over degrading silently, unless the degraded state is explicitly designed and documented.

### Observability

- Design for debuggability. When something goes wrong, can you determine what happened from logs alone?
- Structured logging over string concatenation. Include correlation IDs that trace a request across components.
- Define what metrics matter for this waypoint. If it processes data, track throughput and error rates. If it serves requests, track latency and status codes.

---

## 6. Type Design

### Four-Axis Evaluation

Evaluate types on four axes:
- **Encapsulation** — does the type hide implementation details? Can internals change without affecting consumers?
- **Expression** — does the type make the code's intent clear? Can you understand what a function does from its signature alone?
- **Usefulness** — does the type prevent real bugs? Or is it ceremony that adds complexity without catching mistakes?
- **Enforcement** — does the type system actually enforce the constraint, or can it be easily bypassed?

### Invariant Identification

For each core type, identify:
- What must always be true? (e.g., "an order always has at least one item")
- What must never be true? (e.g., "a completed task cannot have a null completion date")
- Where are these invariants enforced? (constructor, factory function, validation layer)

### Narrow Types

- Prefer specific types over general ones. `UserId` is better than `string`. `PositiveInteger` is better than `number`.
- Use branded or opaque types to prevent mixing values that share a primitive type but have different semantics.
- When a function accepts configuration, type the config object rather than accepting loose parameters.

---

## 7. Code Quality

### Follow Existing Patterns

- Match the codebase's existing style, naming conventions, and architectural patterns.
- If the codebase uses callbacks, don't introduce promises in one module. If it uses classes, don't switch to functional style.
- When introducing a genuinely better pattern, do it as a deliberate migration — not a one-off inconsistency.

### Confidence-Based Decisions

- High confidence changes (well-understood, local impact): implement directly.
- Medium confidence changes (some uncertainty, moderate impact): implement with clear rollback path.
- Low confidence changes (significant uncertainty, wide impact): prototype first, validate assumptions, then commit.

### Comment Quality

- Comments explain **why**, not **what**. The code already shows what it does.
- Good: `// Retry with exponential backoff because the payment API rate-limits burst traffic`
- Bad: `// retry the request` or `// increment counter`
- If code requires a comment to explain what it does, the code should be simplified first.

---

## 8. Applying This Guide

Map these sections to the waypoint design format:

| Guide Section | Design Document Section |
|---|---|
| Architecture Design | **Design** — architecture, key components, build sequence |
| Technology Decisions | **Key Decisions** — technology choices with rejected alternatives |
| Data Model & API | **Schema / Format** — data structures, API contracts |
| Testing Strategy | **Implementation Notes** — testing approach, criticality ratings |
| Error Handling | **Edge Cases** — failure modes, error handling strategy |
| Type Design | **Design** — type hierarchy, invariants |
| Code Quality | **Implementation Notes** — patterns to follow, confidence levels |

Not every software waypoint needs all sections. Use this mapping:

- **Minimal waypoints** (small, well-understood changes): Focus on code quality and testing criticality. A sentence or two is enough.
- **Standard waypoints** (new features, moderate scope): Add architecture, key decisions, and error handling. A few paragraphs per section.
- **Detailed waypoints** (new systems, architectural changes): Full treatment. Data model, type design, testing strategy, observability. This is where rejected alternatives and migration strategies matter.
