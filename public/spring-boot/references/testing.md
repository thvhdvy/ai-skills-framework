# Testing

> Loaded when: user is writing or debugging Spring Boot tests.

## Slice Annotation Selection

| Annotation | Loads | Use for |
|---|---|---|
| `@SpringBootTest` | Full application context | Integration tests; end-to-end flows |
| `@WebMvcTest(Controller.class)` | Web layer only (no JPA, no service beans) | Controller logic, request mapping, validation, exception handling |
| `@DataJpaTest` | JPA layer only (in-memory DB by default) | Repository queries, entity constraints |
| `@JsonTest` | JSON serialization only | DTO serialization shape |
| None (`@ExtendWith(MockitoExtension.class)`) | Nothing | Pure unit tests for service or utility classes |

Choose the narrowest slice that covers the test's scope. `@SpringBootTest` is expensive — reserve it for integration coverage.

## Controller Tests with MockMvc

```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private OrderService orderService;    // @WebMvcTest does not load service beans

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void create_returnsCreated_whenRequestIsValid() throws Exception {
        OrderRequest request = new OrderRequest("Alice", List.of());
        OrderResponse response = new OrderResponse(1L, "Alice", List.of(), OrderStatus.PENDING, Instant.now());

        when(orderService.create(any(OrderRequest.class))).thenReturn(response);

        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1L))
            .andExpect(jsonPath("$.customerName").value("Alice"));
    }

    @Test
    void create_returnsBadRequest_whenCustomerNameIsBlank() throws Exception {
        OrderRequest request = new OrderRequest("", List.of());

        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest());

        verifyNoInteractions(orderService);    // validation failed before the service was called
    }
}
```

When security is active in the test context, add `@WithMockUser` or configure `SecurityMockMvcConfigurer`.

## Repository Tests

```java
@DataJpaTest
class OrderRepositoryTest {

    @Autowired
    private OrderRepository orderRepository;

    @Test
    void findByStatus_returnsMatchingOrders() {
        Order pending = new Order();
        pending.setCustomerName("Bob");
        pending.setStatus(OrderStatus.PENDING);
        orderRepository.save(pending);

        List<Order> results = orderRepository.findByStatusOrderByCreatedAtDesc(OrderStatus.PENDING);

        assertThat(results).hasSize(1);
        assertThat(results.get(0).getCustomerName()).isEqualTo("Bob");
    }
}
```

`@DataJpaTest` uses an embedded H2 database by default. To test against the real database dialect, add `@AutoConfigureTestDatabase(replace = NONE)` and use TestContainers.

## Service Unit Tests

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @InjectMocks
    private OrderService orderService;

    @Test
    void get_throwsNotFound_whenOrderMissing() {
        when(orderRepository.findByIdWithItems(99L)).thenReturn(Optional.empty());

        assertThatThrownBy(() -> orderService.get(99L))
            .isInstanceOf(EntityNotFoundException.class)
            .hasMessageContaining("99");
    }
}
```

## TestContainers (Real Database)

For integration tests that must run against PostgreSQL/MySQL instead of H2:

```java
@SpringBootTest
@Testcontainers
@AutoConfigureTestDatabase(replace = NONE)
class OrderIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
}
```

Add `org.testcontainers:postgresql` and `org.testcontainers:junit-jupiter` to test scope.

## Test Data

| Approach | When to use |
|---|---|
| `@Sql("/data.sql")` | Declarative setup; readable; resets on rollback |
| Builder / factory method in test | Programmatic; flexible for edge cases |
| `@Transactional` on test class | Rolls back after each test; avoids teardown boilerplate |
| Shared `@BeforeEach` setup | When multiple tests share the same starting state |

Avoid sharing mutable state between tests. Each test should be independently runnable in any order.
