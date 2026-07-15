# REST Controller Template

Standard CRUD controller for a `<Resource>` entity. Replace `<Resource>`, `<resource>`, `<ResourceRequest>`, `<ResourceResponse>`, `<ResourceService>`, and the path string with your actual names.

```java
// src/main/java/com/example/<package>/controller/<Resource>Controller.java
@RestController
@RequestMapping("/api/v1/<resources>")
@RequiredArgsConstructor
public class <Resource>Controller {

    private final <Resource>Service <resource>Service;

    @GetMapping
    public ResponseEntity<Page<<ResourceResponse>>> list(Pageable pageable) {
        return ResponseEntity.ok(<resource>Service.list(pageable));
    }

    @GetMapping("/{id}")
    public ResponseEntity<<ResourceResponse>> get(@PathVariable Long id) {
        return ResponseEntity.ok(<resource>Service.get(id));
    }

    @PostMapping
    public ResponseEntity<<ResourceResponse>> create(
            @Valid @RequestBody <ResourceRequest> request) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(<resource>Service.create(request));
    }

    @PutMapping("/{id}")
    public ResponseEntity<<ResourceResponse>> update(
            @PathVariable Long id,
            @Valid @RequestBody <ResourceRequest> request) {
        return ResponseEntity.ok(<resource>Service.update(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        <resource>Service.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

## Request DTO

```java
// src/main/java/com/example/<package>/dto/request/<ResourceRequest>.java
public record <ResourceRequest>(
    @NotBlank String fieldOne,
    @NotNull AnotherType fieldTwo
) {}
```

## Response DTO

```java
// src/main/java/com/example/<package>/dto/response/<ResourceResponse>.java
public record <ResourceResponse>(
    Long id,
    String fieldOne,
    AnotherType fieldTwo,
    Instant createdAt
) {}
```

## Mapping Notes

- Write a `toResponse(<Resource> entity)` private method in the service, not in the controller.
- If mapping is complex or repeated across services, introduce a `<Resource>Mapper` class or use MapStruct.
- Never call `toResponse()` inside a loop with lazy-loaded relationships — ensure the relationship is fetched before the transaction closes.
