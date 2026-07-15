# References Index

Load exactly one file per invocation unless the task spans multiple domains (e.g., adding a secured REST endpoint that also uses JPA — load `rest-api.md` and `data-jpa.md`).

| File | Load when | Contents |
|---|---|---|
| `project-setup.md` | User is initializing a new project, adding starter dependencies, or configuring `application.yml` / `application.properties` | Starter selection, build file templates, property structure, profile setup |
| `rest-api.md` | User is building or fixing controllers, DTOs, exception handling, or API documentation | Controller patterns, DTO design, validation, error responses, OpenAPI |
| `data-jpa.md` | User is working with entities, repositories, custom queries, or database transactions | Entity design, repository methods, JPQL, N+1 prevention, auditing |
| `security.md` | User is configuring authentication, authorization, JWT, or CORS | SecurityFilterChain, JWT filter, UserDetailsService, method security |
| `testing.md` | User is writing or debugging tests | Slice annotations, MockMvc, @MockBean, TestContainers, test data |

If no condition is met, proceed with the SKILL.md body and conventions only.
