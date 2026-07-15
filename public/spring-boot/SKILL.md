---
name: Spring Boot Development
description: >
  Generate, review, and debug Spring Boot applications including REST APIs, JPA
  entities, Spring Security configuration, testing, and application properties.
  Trigger when the user mentions Spring Boot, @SpringBootApplication,
  @RestController, @Entity, spring.datasource, application.yml with spring.*
  keys, or asks to build or fix a Spring Boot service, endpoint, repository, or
  configuration. Also trigger when the user wants a Java web API, microservice,
  or backend that would naturally use Spring Boot. Do not trigger for general
  Java questions unrelated to Spring, or for other frameworks such as Quarkus,
  Micronaut, or Jakarta EE without Spring.
version: 0.1.0
framework-version: "1.0"
lifecycle: draft
owner: Nguyen Huu Thanh
last-reviewed: 2026-07
---

# Spring Boot Development

Generate, review, and debug Spring Boot applications following layered architecture and Spring conventions.

## Orientation

Before writing code, identify:
- **Task type:** new project / new feature / bug fix / code review
- **Context:** Java version (default to 21 LTS), Spring Boot version (default to 3.x), build tool (Maven or Gradle)
- **Scope:** greenfield project or existing codebase

If the user has not specified, proceed with Java 21 + Spring Boot 3.x + Maven unless they correct you.

## Workflow

1. Classify the task using the table below.
2. Load the appropriate reference file (see Conditional Loading).
3. Apply the conventions in this file throughout.
4. Output code blocks annotated with their file path.

| Task type | Signal phrases |
|---|---|
| New project | "start", "scaffold", "create", "new project", "Spring Initializr" |
| New feature | "add endpoint", "add entity", "implement service", "add authentication" |
| Bug fix | "not working", "error", "exception", "fails", "wrong behavior" |
| Code review | "review", "check my code", "is this correct", "best practice" |

## Conventions

Always apply these; they are non-negotiable:

- **Layered architecture:** Controller → Service → Repository. Business logic lives in the service layer only.
- **Constructor injection:** Never `@Autowired` on fields. Declare dependencies as `final` constructor parameters; use Lombok `@RequiredArgsConstructor` where available.
- **DTOs at API boundaries:** Never expose JPA entities directly in request or response bodies. Use separate request/response DTO classes.
- **HTTP status codes:** Controllers return `ResponseEntity<T>` with explicit status. `200` for reads, `201` for creates, `204` for deletes.
- **@Transactional placement:** Service layer only. Never on controllers or repositories.
- **No hardcoded secrets:** Credentials, keys, and URLs go in `application.properties` or environment variables.

## Output Format

For each generated file:

```
// src/main/java/com/example/<package>/<ClassName>.java
<code>
```

For configuration files:

```yaml
# src/main/resources/application.yml
<yaml>
```

Provide the complete file. Do not output fragments unless the user explicitly asks for a snippet.

## Conditional Loading

If the user is **setting up a new project or adding initial dependencies**, read `references/project-setup.md` before proceeding.

If the user is **building or fixing REST controllers, DTOs, or exception handling**, read `references/rest-api.md` before proceeding.

If the user is **working with JPA entities, repositories, or database queries**, read `references/data-jpa.md` before proceeding.

If the user is **configuring authentication, authorization, or Spring Security**, read `references/security.md` before proceeding.

If the user is **writing or fixing tests**, read `references/testing.md` before proceeding.

If the user **asks for a code template**, read the appropriate file from `templates/` before proceeding.

If the user **asks to review Spring Boot code**, read `checklists/code-review.md` before proceeding.

## Edge Cases

**Ambiguous Spring version:** If the user's code shows `javax.*` imports, they are on Spring Boot 2.x (Java EE namespace). Do not silently upgrade to `jakarta.*`. Ask which version they are targeting or mirror their existing imports.

**Mixed concerns:** If the user's code has business logic in a controller or `@Transactional` on a controller method, point out the violation and offer to refactor before fixing the stated bug.

**Missing dependency:** If the generated code requires a starter not declared by the user, state the required dependency and show the Maven/Gradle snippet before showing the code that needs it.

## Self-Review Checklist

Before responding, verify:
- [ ] Business logic is in the service layer, not the controller
- [ ] Constructor injection used; no `@Autowired` field injection
- [ ] DTOs used at request/response boundaries; no entity exposure
- [ ] Correct HTTP status codes returned by controllers
- [ ] No secrets, passwords, or connection strings hardcoded in source
- [ ] If writing a test, the correct slice annotation is used (`@WebMvcTest`, `@DataJpaTest`, `@SpringBootTest`)
