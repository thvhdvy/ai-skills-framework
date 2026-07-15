# REST API

> Loaded when: user is building or fixing REST controllers, DTOs, exception handling, or API documentation.

## Controller Pattern

```java
@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;

    @GetMapping
    public ResponseEntity<Page<OrderResponse>> list(Pageable pageable) {
        return ResponseEntity.ok(orderService.list(pageable));
    }

    @GetMapping("/{id}")
    public ResponseEntity<OrderResponse> get(@PathVariable Long id) {
        return ResponseEntity.ok(orderService.get(id));
    }

    @PostMapping
    public ResponseEntity<OrderResponse> create(@Valid @RequestBody OrderRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(orderService.create(request));
    }

    @PutMapping("/{id}")
    public ResponseEntity<OrderResponse> update(@PathVariable Long id,
                                                 @Valid @RequestBody OrderRequest request) {
        return ResponseEntity.ok(orderService.update(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        orderService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

Rules:
- One controller per resource (not per use case)
- No business logic — delegate immediately to service
- Map collection endpoints to return `Page<T>` when the dataset can grow large

## DTO Design

Keep request and response DTOs separate. Do not reuse the same class for both.

```java
// dto/request/OrderRequest.java
public record OrderRequest(
    @NotBlank String customerName,
    @NotNull @Valid List<OrderItemRequest> items
) {}

// dto/response/OrderResponse.java
public record OrderResponse(
    Long id,
    String customerName,
    List<OrderItemResponse> items,
    OrderStatus status,
    Instant createdAt
) {}
```

Use Java records for DTOs (Spring Boot 3.x / Java 16+). Immutable, no boilerplate.

## Validation

Add `spring-boot-starter-validation` to use Bean Validation annotations.

| Annotation | Validates |
|---|---|
| `@NotNull` | Not null |
| `@NotBlank` | Not null, not empty, not whitespace (Strings) |
| `@Size(min, max)` | String length or collection size |
| `@Min` / `@Max` | Numeric range |
| `@Email` | Email format |
| `@Valid` | Cascade validation into nested objects |
| `@Positive` | Number > 0 |

Always annotate the controller parameter with `@Valid`; annotate fields on the DTO.

## Exception Handling

Centralize exception mapping in one `@ControllerAdvice` class.

```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(EntityNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return ResponseEntity.badRequest().body(new ErrorResponse(message));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unhandled exception", ex);
        return ResponseEntity.internalServerError()
            .body(new ErrorResponse("An unexpected error occurred"));
    }
}

public record ErrorResponse(String message) {}
```

Never expose stack traces or internal error details in the response body.

## Pagination

Accept `Pageable` directly in controller parameters. Spring resolves `?page=0&size=20&sort=createdAt,desc` automatically.

Enable in configuration:
```java
@Configuration
@EnableSpringDataWebSupport(pageSerializationMode = VIA_DTO)
public class WebConfig {}
```

`VIA_DTO` mode serializes `Page<T>` as a consistent JSON structure rather than exposing Spring internals.

## OpenAPI Documentation

Add `springdoc-openapi-starter-webmvc-ui` (not `springfox`) for Spring Boot 3.x.

```java
@Operation(summary = "Create order")
@ApiResponse(responseCode = "201", description = "Order created")
@ApiResponse(responseCode = "400", description = "Validation error")
@PostMapping
public ResponseEntity<OrderResponse> create(...) { ... }
```

Available at `/swagger-ui.html` and `/v3/api-docs` by default.
