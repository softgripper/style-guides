# The Meta
This guide is a living document, and is meant to give guidance and a goal without being prescriptive or bombarding the reader with hundreds of rules or dozens of pages of detail.

## Task ownership
- Take responsiblity and owernship of tasks.
  This includes dataflow design, testing and code as a single unit, by a single person, including any supporting documentation.


## PR reviews
- Provide feedback in PRs.  This is a review stage, not just an approval stage.  The review should relate to both the requirement of the task and the items listed in this document.


## Branch strategy
- We use a branching strategy based off of the [Gitflow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).


- Add your JIRA ticket number to the branch name with a hyphen.  This gives a bidirectional connection between the branch and the ticket.

  eg: `feature/SS-1234-update-data-flow-diagrams`


## Commit messages
- Commit messages should be meaningful and descriptive, and written in the imperative tense.
- Commit messages should end with the JIRA ticket number.  This gives a bidirectional connection between the commit and the ticket.

  eg: `update saveDetails data flow (SS-1234)`


## Good habbits
- If you're working on a task - it should have a JIRA ticket.
- Design/update the data flow before writing any code.
- Test early and often.  Tests should be simple and fast.
- If you have to disable a test create a JIRA ticket for the test - and link that JIRA in the @Disabled annotation before it is committed.
  Alternately, if the test is to be run manually only, add a comment to the test explaining why it is disabled.
- Remove unused code.
- Log often - with detail (many log methods accept a Throwable - always provide this if available).
- Aim to have code close to where it's used - grouped by feature.
- Inject config properties into the class that uses them - not a "God class".
- User our digital-utils package for common code.
- Clean as you go - carefully.


## Avoiding bad habbits
- Do not commit "commented out" tests!

- Do not commit unused imports.  (Alt-Shift-O in IntelliJ will do this for you)

- Avoid splitting ownership of a task across multiple people.

- Avoid adding new dependencies.  These must be reviewed and justified.

- Avoid adding single use methods in far away packages.

- Avoid adding a new class to a far away package just because the pattern is common.

- Duplication of small amount of code is OK, but avoid duplication of large amounts of code.


# formatting
- TODO (we should just automate this via some gradle plugin)
https://github.com/google/google-java-format


# The code itself
## Naming
- Packages: all lower-case (no underscores).
- Classes/Interfaces/Enums/Records: UpperCamelCase.
- Methods/Fields/Locals: lowerCamelCase.
- Constants: UPPER_SNAKE_CASE.

Keep naming basic without scope unless differentiation is required.

Eg,
```java
// Ideal
void doSomething(User sender) {
  var address = user.getAddress();
}

// Less good
void doSomething(User sender) {
  var senderAddress = sender.getAddress();
}

// Ideal with scope
 void doSomething(User sender, User reciever) {
  var senderAddress = sender.getAddress();
  var recieverAddress = receiver.getAddress();
}
```

## `var` keyword

For our codebases, the complexity is reduced by using the var keyword in majority of cases.

Almost always prefer the var keyword over the explicit type declaration.


### Pros
- Smaller PRs - less files changed when renaming or moving classes, or changing return types of methods.
- Generics code easier to reason about and consume.
- Encourages better variable names, and variables line up.
- Easier to identify common code patterns shared between entities of different types.
- Easier to work with the return values of Stream pipelines.
- It's more concise (way less typing, way less reading).
- It is a language design pattern similar to many other languages eg, `let` in Typescript, `auto` in C++, and `var` in C#.


### Cons
- You may forget the type (your IDE will tell you in the vast majority of cases, and you will be unable to compile).


### When should I pay close attention to edge cases for var keyword?
- [When assigning some primitive literals](https://openjdk.org/projects/amber/guides/lvti-style-guide#G7)

```java
// Good
var users = repository.findActiveUsers();

// Be explicit when numeric literals can mislead
long timeoutMs = 1_000L;

// Avoid confusing generics
Map<String, List<User>> index = computeIndex();
```


## Validating requests
TODO - show examples from the mobile-phone-add implementation. (hibernate annotation + records)

## Exceptions
- Try to always use our digital-utils `DigitalException` base class for exceptions.
- Do not add runtime exceptions to method signatures.
- We deal with exceptions in the CentralExceptionMapper classes (or similar) for our Quarkus applications.
- Fail fast with clear exception messages.
- If rethrowing from a checked exception, wrap it in a DigitalException.  Always provide it as a "throwable" parameter.


## Null/Optional
- Annotate methods and arguments with jakarta.annotation.Nonnull or jakarta.annotation.Nullable.

- Prefer returning Optionals over nulls - Don’t return null for "no value"
- Do not use Optionals for params or fields.

- Consider validating inputs with Objects.requireNonNull and clear messages depending on the context.

- If returning null is the error state, consider throwing a DigitalException.

## Immutability and state

- Prefer final fields and constructor injection.
- Favor immutable value types (records) and immutable collections (List.of, Set.of).
- Avoid exposing mutable internals; defensive copies for arrays/collections.

## Collections and streams

- Use streams for simple pipelines; prefer loops when logic is complex or needs debugging.
- Avoid parallel streams unless you’ve measured benefit and ensured thread-safety.

## Logging
- Use slf4j with parameterized messages; avoid string concatenation in logs.
- Don’t log secrets or PII;


## Testing
- JUnit 5; clear test names; one behavior per test.