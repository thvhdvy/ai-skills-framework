# Spring Data JPA

> Loaded when: user is working with JPA entities, repositories, custom queries, or database transactions.

## Entity Design

```java
@Entity
@Table(name = "orders")
@Getter
@Setter
@NoArgsConstructor
@EntityListeners(AuditingEntityListener.class)
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String customerName;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderStatus status;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();

    @CreatedDate
    @Column(updatable = false)
    private Instant createdAt;

    @LastModifiedDate
    private Instant updatedAt;
}
```

Rules:
- Use `@Column(nullable = false)` for every non-null column — JPA does not infer this from `@NotNull`
- `@Enumerated(EnumType.STRING)` prevents breakage when enum values are reordered
- Prefer `Instant` for timestamps; avoid `Date` and `Calendar`
- Initialize collection fields to avoid NPE on `add()` before persist

Enable auditing in a `@Configuration` class:
```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {}
```

## Repository Interfaces

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // Derived query — Spring generates JPQL from method name
    List<Order> findByStatusOrderByCreatedAtDesc(OrderStatus status);

    // JPQL — explicit, readable for complex queries
    @Query("SELECT o FROM Order o JOIN FETCH o.items WHERE o.id = :id")
    Optional<Order> findByIdWithItems(@Param("id") Long id);

    // Exists check — cheaper than findById() + isEmpty()
    boolean existsByCustomerNameAndStatus(String customerName, OrderStatus status);

    // Projection — return only needed columns, not the full entity
    @Query("SELECT o.id AS id, o.customerName AS customerName FROM Order o WHERE o.status = :status")
    List<OrderSummary> findSummariesByStatus(@Param("status") OrderStatus status);

    // Pagination
    Page<Order> findByStatus(OrderStatus status, Pageable pageable);
}

// Projection interface (Spring Data generates the proxy)
public interface OrderSummary {
    Long getId();
    String getCustomerName();
}
```

## Transactions

| Rule | Why |
|---|---|
| `@Transactional` on service methods | Controller has no business logic; repositories are already transactional per operation |
| `@Transactional(readOnly = true)` for read operations | Signals the DB driver/connection pool that no writes occur; enables optimizations |
| Do not catch and swallow checked exceptions inside `@Transactional` | Checked exceptions do not trigger rollback by default — use `rollbackFor` if needed |
| Never call a `@Transactional` method from another method in the same bean | Spring AOP proxy is bypassed; the transaction never starts |

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderResponse get(Long id) {
        Order order = orderRepository.findByIdWithItems(id)
            .orElseThrow(() -> new EntityNotFoundException("Order not found: " + id));
        return toResponse(order);
    }

    @Transactional    // overrides class-level readOnly = true
    public OrderResponse create(OrderRequest request) {
        Order order = new Order();
        order.setCustomerName(request.customerName());
        order.setStatus(OrderStatus.PENDING);
        return toResponse(orderRepository.save(order));
    }
}
```

## N+1 Problem

N+1 occurs when fetching a collection triggers one additional query per element.

**Detection:** Enable `show-sql: true` temporarily and count queries. If fetching 20 orders fires 21 queries, you have N+1.

**Fix options:**

| Fix | When to use |
|---|---|
| `JOIN FETCH` in JPQL | Single query needed; small associated collections |
| `@EntityGraph` on repository method | Prefer over `JOIN FETCH` for readability on named relationships |
| `@BatchSize(size = 25)` on collection field | Multiple calls acceptable; avoids cartesian products |
| Projections / DTOs via `@Query` | Read-only use cases; avoid hydrating entities at all |

```java
// @EntityGraph example
@EntityGraph(attributePaths = {"items", "items.product"})
Optional<Order> findWithItemsById(Long id);
```

Avoid `FetchType.EAGER` — it fires the join on every load, even when the association is not needed.

## Schema Management

| Setting | When to use |
|---|---|
| `ddl-auto: validate` | Production — Hibernate checks schema matches entities; fails fast on mismatch |
| `ddl-auto: update` | Never in production — silent destructive changes possible |
| `ddl-auto: create-drop` | Unit tests only |
| Flyway / Liquibase | Preferred for all environments; gives version-controlled migration history |

To add Flyway: `spring-boot-starter-data-jpa` + `flyway-core`. Migrations live in `src/main/resources/db/migration/` as `V1__init.sql`, `V2__add_column.sql`, etc.
