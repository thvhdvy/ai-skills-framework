# Code Review Checklist

> Run this checklist at: the start of a Spring Boot code review request, before commenting on any specific issue.

## Required Checks

- [ ] **Layered architecture respected**
      If failed: business logic found in controller or repository — offer to extract to service layer.

- [ ] **Constructor injection used throughout**
      If failed: `@Autowired` on fields present — rewrite with `final` fields and constructor / `@RequiredArgsConstructor`.

- [ ] **No JPA entities in API responses or request bodies**
      If failed: entity class used as `@RequestBody` or returned from controller — introduce request/response DTOs.

- [ ] **@Transactional on service layer only**
      If failed: found on controller method or repository method — move to service; repository operations are already transactional per operation.

- [ ] **No hardcoded secrets or credentials**
      If failed: passwords, tokens, connection strings in source — move to environment variables or `application.yml` with `${ENV_VAR}` substitution.

- [ ] **Exception handling centralized**
      If failed: `try/catch` blocks in controllers returning error strings — consolidate into `@ControllerAdvice`.

- [ ] **HTTP status codes are explicit and correct**
      If failed: all responses return 200, or error cases return 200 — correct the `ResponseEntity` status.

- [ ] **FetchType.LAZY on all @ManyToOne and @OneToOne**
      If failed: `FetchType.EAGER` present or default accepted — change to LAZY and ensure association is fetched when needed via JOIN FETCH or @EntityGraph.

- [ ] **open-in-view is disabled**
      If failed: `spring.jpa.open-in-view` is true (default) — add `spring.jpa.open-in-view=false`; OSIV masks N+1 problems and holds DB connections across the view layer.

## Advisory Checks

- [ ] `@Enumerated(EnumType.STRING)` used on all enum fields (not ORDINAL).
- [ ] Pagination used for list endpoints that could grow large (`Pageable` parameter).
- [ ] Tests exist for the happy path and at least one validation/error case.
- [ ] `ddl-auto` is not `create`, `create-drop`, or `update` in the production profile.
- [ ] Lombok `@Data` is not used on JPA entities (breaks proxy-based equality).
- [ ] Spring Boot version is 3.x and `javax.*` imports have not crept in alongside `jakarta.*`.
