# The Meta

This guide is a living document, and is meant to give a guiding direction without being too prescriptive or bombarding the reader with hundreds of rules or dozens of pages of detail.

Always try to leave the code healthier than when you found it.

Prefer small, safe refactors alongside changes.

## Task ownership

- Take responsibility and ownership of the task.

  A project consists of many tasks, and each development task should be treated as a single unit of work - by an individual.

  This includes:

  - data flow design
  - testing
  - code
  - supporting documentation

## Using AI

Use AI:

- for tasks that you could write yourself.
- to review code you write.
- to brainstorm ideas.

Be very wary using it for anything else, for example, concepts you are unfamiliar with.

It can confidently give you an answer that is wrong and hard to spot. We need to be able to understand and catch these mistakes. It is sycophantic and will praise your ideas and agree with you.

You must understand the code you are committing.

Ultimately, we are responsible for what we, or AI, produce.

## Branch strategy

We use a branching strategy based off of the [Gitflow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).

When creating a branch, add your Jira ticket number to the branch name with a hyphen. This gives a bidirectional connection between the branch and Jira.

e.g., `feature/SS-1234-update-data-flow-diagrams`

Feature branches should be short lived, and removed after merge.

The caveat here is, a large project may run for a few sprints - then we make a feature branch for that project. Organise with DevOps to make these longer running feature branches protected.

e.g., `feature/large-project-name`,

and create feature branches off it as we build out its functionality.

e.g., `feature/SS-7890-large-project-parser`

Here's some more examples of gitflow style branches

```
feature/SS-1234-update-data-flow-diagrams
hotfix/SS-4321-prod-nullpointer
bugfix/SS-5678-fix-payment-service-constructor-injection
release/2025.10.0
```

## Commit messages

Prefix with Jira ticket and write an imperative subject; keep the subject concise and meaningful.

```
SS-1234: update saveDetails data flow
SS-1235: add validation for missing email in registration
SS-5678: refactor payment service to constructor injection
SS-9012: fix NPE in user lookup on null address
```

If there are any rollback steps, or specific notes of importance, you can add a 1 paragraph body to the commit message.

### Merge strategy

- We've typically been using MERGE, but REBASE and SQUASH has it's place.
- If you're working with others on a feature branch, coordinate with them to decide on the merge strategy. If you spend more than 60 seconds talking about it - use MERGE.

## PR reviews

- Provide feedback in PRs. This is a review stage, not just an approval stage. The review should relate to both the requirement of the task and the items listed in this document.

Typically try to keep a PR to <300 LOC of meaningful logic or less. This is not always possible, but it is a good goal.

We are reviewing the code - not the individual. We are not looking for a "perfect" solution, but rather a solution that is maintainable and easy to understand.

See https://conventionalcomments.org/ for some guidance on writing good comments.

### Java PR checklist

- Data flow updated if relevant.
- Correctness against acceptance criteria.
- Test coverage for change.
- Logging important code paths and events.
- Ensure no PII is logged.
- No disabled tests without Jira or manual instruction.
- Accept var by default. Request explicit types only when they materially improve clarity or prevent a realistic bug (typically numeric precision/width).
- Encourage descriptive names so type inference remains readable in reviews.
- Does not contain secrets!
- Only approve a PR after a successful CI/CD build pipeline.

## Good habits

- If you're working on a task - it should always have a Jira ticket. If not - make one.
- Design/update the data flow before writing any code.
- Test early and often. Tests should be simple and fast.
- If you have to disable a test, create a Jira ticket and link that Jira in the @Disabled annotation before it is committed.
  Alternately, if the test is to be run manually only, add a comment to the test explaining why it is disabled.
- Remove unused code.
- Log often - with detail (many log methods accept a Throwable - always provide this if available).
- Aim to have code close to where it's used - grouped by feature.
- Prefer constructor injection where possible.
- Inject config properties into the class that uses them - not a "God class".
- Use our digital-utils package for common code.
- Clean as you go - carefully. Leave it better than you found it.
- If IntelliJ has yellow squiggles, fix them. Treat IDE inspections as warnings unless deliberately suppressed with a justification.
- Prefer updating dependencies in small PRs; avoid major upgrades in mixed changes.

## Avoiding bad habits

- Avoid committing TODOs. If you absolutely must, add a Jira ticket, then reference it in the TODO.
- Do not commit "commented out" tests.
- Avoid splitting ownership of a task across multiple people.
- Avoid adding new dependencies. These must be reviewed and justified.
- Avoid adding single use methods in far away packages.
- Avoid adding a new class to a far away package just because the pattern is common (e.g., you might not need a new "Mapper" with 2 single use methods).
- Duplication of small amount of code is OK, but avoid duplication of large amounts of code.
- Avoid copy/paste from other projects. Reuse is great - but unused code is a code smell, as is copy/paste/subtly changed code.

# CI/CD

We have Azure Pipelines for CI/CD.

These are running:

- Git Guardian - for secret leaks
- Snyk - for vulnerabilities
- Jacoco - for code coverage
- JUnit - for unit tests

# The code itself

Create an early PR in a draft state to get feedback early.

## Naming

- Packages: all lowercase (no underscores).
- Classes/Interfaces/Enums/Records: UpperCamelCase.
- Methods/Fields/Locals: lowerCamelCase.
- Constants: UPPER_SNAKE_CASE.

For variables, keep naming basic without scope unless differentiation is required. This helps spot common patterns and can help the IDE identify duplicated code in the project - which is ripe for a refactor.

e.g.,

```java
// Ideal
void doSomething(User user) {
  var address = user.getAddress();
}

// Less good
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

- Try to always use our digital-utils `DigitalException` base class for exceptions.
- Do not add runtime exceptions to method signature throws.
- We deal with handling thrown exceptions in the CentralExceptionMapper classes (or similar) for our Quarkus applications.
- Fail fast with clear exception messages.
- If re-throwing from a checked exception, wrap it in a DigitalException. Always provide it as a "throwable" parameter.
- Prefer "try-with-resources" and never swallow exceptions;
- Avoid swallowing exceptions (e.g., base Exception type, with no logging).

## Null/Optional

- Annotate methods and arguments with jakarta.annotation.Nonnull or jakarta.annotation.Nullable.
- Prefer returning Optionals over nulls - Do not return null for "no value".
- Do not use Optionals for params or fields.
- Consider validating inputs with Objects.requireNonNull and clear messages depending on the context.
- Do not return null as error state, consider throwing a DigitalException or logging the error then returning an `Optional.empty()`.

## Immutability and state

- Prefer final fields and constructor injection.
- Favor immutable value types (records) and immutable collections (List.of, Set.of).
- Avoid exposing mutable internals; defensive copies for arrays/collections.

## Collections and streams

- Use streams for simple pipelines; prefer loops when logic is getting hard to grok or needs debugging.
- Avoid parallel streams unless you’ve measured benefit and ensured thread-safety.

## Logging

- Use slf4j with parameterized messages; avoid string concatenation in logs.
- Don’t log secrets or PII
- If logs might include identifiers, mask or hash them.

## `var` keyword

For our codebases, the complexity is reduced by using the var keyword in majority of cases.

Use var by default for local variables. It reduces noise, keeps diffs small, and makes refactors easier. Pair it with clear names and small, focused methods.

- Prefer var for local variables.
- Prefer final var for locals that must not be reassigned.
- Avoid reassigning vars.
- var is only for locals, not fields, parameters, or return types.

* see [LVTI Style Guidelines from OpenJDK](https://openjdk.org/projects/amber/guides/lvti-style-guide) for more details.

### Why use vars

Cleaner diffs during refactors (renames/moves/return type changes).
Less boilerplate around generics and stream pipelines.
Encourages better variable names and more readable code.
Aligns with modern languages and idioms.

### Guardrails (when to be explicit)

Numeric width/precision matters:

```java
// Numeric/primitive clarity
long timeoutMs = 5_000L; // Prefer explicit for width
float fraction = 0.25f; // Avoid accidental double
double ratio = 1d * a / b; // Avoid int division
```

Avoid raw types: always use generics or the diamond operator on the RHS.
If the initializer doesn’t communicate enough and a better name can’t fix it, consider an explicit type (rare).
var x = null is illegal; use an explicit type (or restructure).

```java
// Good defaults
void process(User sender) {
  final var address = sender.getAddress();
  var activeUsers = repository.findActiveUsers();
  var index = new HashMap<String, List<User>>();
  var result = service.compute(sender); // Name + context carry intent
}

// Generics/raw types
var orders = new ArrayList<Order>(); // Good
var orders = new ArrayList(); // Bad: infers raw type

// Readability: prefer a better name; avoid explicit types unless it truly helps
var discountRate = calculator.computeDiscountRate(customer);
```

## Testing

- JUnit 5; clear test names; aim for one behavior per test.
- Mock out dependencies where possible.
- Assert mocks are called as expected if relevant for the test.
- Avoid mocking "the kitchen sink" of dependencies.
- Use the "given-when-then" pattern for tests.
- Most of our tests are Quarkus tests - which are basically written as unit tests with Quarkus running.

## formatting - don't leave a mess

- Do not commit unused imports. (Alt-Shift-O in IntelliJ will do this for you)
- Don't commit large amounts of "commented out" code.
- RADAR Item - try to automate this via CI/CD, a gradle plugin https://github.com/google/google-java-format or project level IDE settings and take the onus away from the developer.

## Helpful tips

- Use a .env file to store development environment variables.
- If a repo doesn't have a .env.example - please create one with stubs for all the environment variables.

# Suggested readings

- https://baeldung.com contains a wealth of knowledge for Java developers, covering the details of many of our technologies and techniques.
- [SOLID principles](https://www.baeldung.com/solid-principles)
- [Refactoring - Martin Folwer](https://martinfowler.com/books/refactoring.html)
- [Agile Threat Modelling](https://martinfowler.com/articles/agile-threat-modelling.html)
