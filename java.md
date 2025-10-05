# The Meta

This is a living guide. It sets direction without being overly prescriptive.

Always leave the code healthier than you found it.

Prefer small, safe refactors alongside changes.

## Task ownership

- Take responsibility for the whole task as a single unit of work.

- Own:
  - data flow design
  - tests
  - code
  - supporting documentation

## Using AI

- Use AI to accelerate tasks you could write yourself, review your code, and brainstorm.
- Be cautious for unfamiliar concepts or domains.
- AI can be wrong and sycophantic. Verify its claims.
- You must understand and be able to explain the code you commit.
- You are responsible for what you and AI produce.

## Branch strategy

- We follow a Gitflow-style workflow.
- Typically `main` will mirror Production.
- `develop` is used for completed feature work.
- we cut `release` branches from `develop` for production releases.
- we use `bugfix` and/or `feature` when patching `release` branches. These are merged back into `develop`.
- we create `hotfix` from `main`, and merge back into `develop` and `main`.

## Branch Naming

Include the Jira ticket in branch names:

```
feature/SS-1234-update-data-flow-diagrams
```

Feature branches should be short-lived and removed after merge.

For large projects spanning sprints, create a long-running feature branch and protect it. The team can then can branch project features off it.

Examples:

```
feature/SS-1234-update-data-flow-diagrams
hotfix/SS-4321-prod-nullpointer
bugfix/SS-5678-fix-payment-service-constructor-injection
release/25.10.0
```

## Commit messages

- Prefix with Jira ticket and write an imperative, concise subject.

```
SS-1234: update saveDetails data flow
SS-1235: add validation for missing email in registration
SS-5678: refactor payment service to constructor injection
SS-9012: fix NPE in user lookup on null address
```

- Add a one-paragraph body only if rollback steps or important notes are needed.

### Merge strategy

- Default: merge commits.
- Very noise feature branch may be squashed.

## Pull Request reviews

- Pull Requests are for review, not just approval. Review for both requirements and this guide.
- Aim for <300 LOC of meaningful logic. If larger, keep commits focused and the Pull Request description clear.
- Review the code, not the person. Optimize for maintainability and clarity.
- See https://conventionalcomments.org/ for guidance.

### Java Pull Request checklist

- Data flow updated if relevant.
- Correctness against acceptance criteria.
- Test coverage for the change.
- Log important code paths and events. Do not log PII.
- No disabled tests without Jira or manual instruction.
- Use var by default for locals; ask for explicit types only when they improve clarity or prevent realistic bugs.
- Prefer descriptive names so type inference remains readable.
- No secrets in code or config.
- Approve only after a successful CI/CD test pass in the pipeline.

## Good habits

- Every task has a Jira ticket. If missing, create one.
- Design/update data flow before coding.
- Test early and often. Keep tests simple and fast.
- If disabling a test, add a Jira ticket and link it in @Disabled, or explain why it runs manually.
- Remove unused code.
- Log with detail, including Throwables when available.
- Keep related code close by feature.
- Prefer constructor injection.
- Inject config where used, not into a "God Config class."
- Use our digital-utils package for common code.
- Clean as you go. Leave it better than you found it.
- Fix IDE inspections (yellow squiggles). Suppress only with justification.
- Update dependencies in small Pull Requests. Avoid mixing major upgrades with feature work.

## Avoiding bad habits

- Avoid committing TODOs. If necessary, add a Jira ticket and reference it.
- Do not commit commented-out tests.
- Avoid splitting task ownership across people.
- Avoid adding new dependencies without review and justification.
- Avoid single-use helpers in far-away packages.
- Avoid adding classes to distant packages just to follow a pattern.
- Small duplication is OK. Avoid large duplication.
- Avoid copy/paste from other projects. Prefer deliberate reuse without dead code.

# CI/CD

- Azure Pipelines run:
  - GitGuardian for secret leaks
  - Snyk for vulnerabilities
  - JaCoCo for code coverage
  - JUnit for unit tests

# The code itself

Create an early Pull Request in a draft state to get feedback.

## Naming

- Packages: all lowercase (no underscores).
- Classes/Interfaces/Enums/Records: UpperCamelCase.
- Methods/Fields/Locals: lowerCamelCase.
- Constants: UPPER_SNAKE_CASE.
- Prefer simple names without scope unless needed for differentiation.

Examples:

```java
// Ideal
void doSomething(User user) {
  var address = user.getAddress();
}

// Not ideal
void doSomething(User user) {
  var userAddress = user.getAddress();
}

// Ideal with scope
 void doSomething(User sender, User receiver) {
  var senderAddress = sender.getAddress();
  var receiverAddress = receiver.getAddress();
}
```

## Exceptions

- Prefer our digital-utils `DigitalRuntimeException` as a base.
- Do not add runtime exceptions to throws.
- Handle thrown exceptions centrally (e.g., `CentralExceptionMapper` in Quarkus apps).
- Fail fast with clear messages.
- When rethrowing a checked exception, wrap it in a `DigitalRuntimeException` and include the cause.
- Prefer try-with-resources.
- Never swallow exceptions. Log with context or rethrow.

## Null/Optional

- Annotate methods and parameters with `jakarta.annotation.Nonnull` or `jakarta.annotation.Nullable`.
- Prefer returning `Optional` over nulls.
- Do not use `Optional` for fields or parameters.
- Validate inputs with `Objects.requireNonNull` when appropriate, with clear messages.
- Do not use null as an error state. Throw a `DigitalRuntimeException` or return `Optional.empty()` with logging if appropriate.

## Immutability and state

- Prefer final fields and constructor injection.
- Favor immutable value types (records) and immutable collections (`List.of`, `Set.of`).
- Avoid exposing mutable internals; defensive copies for arrays/collections.

## Collections and streams

- Use streams for simple pipelines.
- Prefer loops when logic is complex or needs step debugging.
- Avoid parallel streams unless measured and safe.

## Logging

- Use slf4j with parameterized messages. Avoid string concatenation.
- Don’t log secrets or PII
- If logs might include identifiers, mask or hash them where appropriate.

## `var` keyword

For our codebases, the complexity is reduced by using the var keyword in majority of cases.

Use var by default for local variables. It reduces noise, keeps diffs small, and makes refactors easier. Pair it with clear names and small, focused methods.

- Prefer var for locals.
- Avoid reassigning vars. Consider using final to ensure this.
- var is only for locals, not fields, parameters, or return types.

* see [LVTI Style Guidelines from OpenJDK](https://openjdk.org/projects/amber/guides/lvti-style-guide) for more details.

### Why use vars

- Cleaner diffs during refactors.
- Less boilerplate around generics and stream pipelines.
- Encourages better names and readability.
- Aligns with modern Java and other languages and idioms.

### Guardrails (when to be explicit)

Numeric width/precision matters:

```java
// Numeric clarity
long timeoutMs = 5_000L;   // Prefer explicit width
float fraction = 0.25f;    // Avoid accidental double
double ratio = 1d * a / b; // Avoid int division

// Good defaults
void process(User sender) {
  final var address = sender.getAddress();
  var activeUsers = repository.findActiveUsers();
  var index = new HashMap<String, List<User>>();
  var result = service.compute(sender);
}

// Generics/raw types
var orders = new ArrayList<Order>(); // Good
var orders = new ArrayList();        // Bad: raw type

// Prefer a better name over explicit type
var discountRate = calculator.computeDiscountRate(customer);
```

## Testing

- JUnit 5. Clear test names. One behavior per test.
- Mock dependencies as needed. Avoid mocking everything.
- Verify interactions when relevant.
- Use given-when-then.
- Most tests run under Quarkus; write them like unit tests with Quarkus running.

## Formatting - don't leave a mess

- Do not commit unused imports.
- Don't commit large amounts of "commented out" code.

## Helpful tips

- Use a .env file for development environment variables.
- If a repo lacks .env.example, add one with stubs for all env vars.

# Suggested readings

- https://baeldung.com contains a wealth of knowledge for Java developers, covering the details of many of our technologies and techniques.
- [SOLID principles](https://www.baeldung.com/solid-principles)
- [Refactoring - Martin Fowler](https://martinfowler.com/books/refactoring.html)
- [Agile Threat Modelling](https://martinfowler.com/articles/agile-threat-modelling.html)
