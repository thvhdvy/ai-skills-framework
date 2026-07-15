# JPA Entity Template

Standard audited entity with generated primary key. Replace `<Resource>` and field declarations with your actual names.

```java
// src/main/java/com/example/<package>/entity/<Resource>.java
@Entity
@Table(name = "<resources>")
@Getter
@Setter
@NoArgsConstructor
@EntityListeners(AuditingEntityListener.class)
public class <Resource> {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String fieldOne;

    @Column(nullable = false)
    @Enumerated(EnumType.STRING)
    private <Status>Status status;

    // One-to-many example — owned side
    @OneToMany(mappedBy = "<resource>", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<<Child>> children = new ArrayList<>();

    // Many-to-one example — foreign key side
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "parent_id", nullable = false)
    private <Parent> parent;

    @CreatedDate
    @Column(updatable = false, nullable = false)
    private Instant createdAt;

    @LastModifiedDate
    @Column(nullable = false)
    private Instant updatedAt;
}
```

## Checklist Before Using

- [ ] `@Table(name = ...)` uses snake_case plural matching the actual DDL table name
- [ ] Every `@Column` that maps to a NOT NULL column has `nullable = false`
- [ ] Enum fields use `EnumType.STRING`, not `EnumType.ORDINAL`
- [ ] Collection fields initialized to `new ArrayList<>()` to prevent NPE
- [ ] `@ManyToOne` and `@OneToOne` use `FetchType.LAZY`; never default (EAGER)
- [ ] `@JpaAuditing` is enabled in a `@Configuration` class

## Lombok Notes

- `@Getter` + `@Setter` on class level covers all fields; fine for entities
- Do not use `@Data` on entities — `equals()` / `hashCode()` generated from all fields causes issues with Hibernate proxies and collections
- Do not use `@EqualsAndHashCode` unless the natural key is stable and immutable; default identity-based equality is safer for entities
