# Service Template

Standard service class for a `<Resource>` entity with read-only default transaction and write overrides.

```java
// src/main/java/com/example/<package>/service/<Resource>Service.java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class <Resource>Service {

    private final <Resource>Repository <resource>Repository;

    public Page<<ResourceResponse>> list(Pageable pageable) {
        return <resource>Repository.findAll(pageable).map(this::toResponse);
    }

    public <ResourceResponse> get(Long id) {
        return <resource>Repository.findById(id)
            .map(this::toResponse)
            .orElseThrow(() -> new EntityNotFoundException("<Resource> not found: " + id));
    }

    @Transactional
    public <ResourceResponse> create(<ResourceRequest> request) {
        <Resource> entity = new <Resource>();
        // populate entity from request
        entity.setFieldOne(request.fieldOne());
        return toResponse(<resource>Repository.save(entity));
    }

    @Transactional
    public <ResourceResponse> update(Long id, <ResourceRequest> request) {
        <Resource> entity = <resource>Repository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("<Resource> not found: " + id));
        // apply changes from request
        entity.setFieldOne(request.fieldOne());
        return toResponse(entity);    // save is implicit; entity is already managed
    }

    @Transactional
    public void delete(Long id) {
        if (!<resource>Repository.existsById(id)) {
            throw new EntityNotFoundException("<Resource> not found: " + id);
        }
        <resource>Repository.deleteById(id);
    }

    private <ResourceResponse> toResponse(<Resource> entity) {
        return new <ResourceResponse>(
            entity.getId(),
            entity.getFieldOne(),
            entity.getCreatedAt()
        );
    }
}
```

## Notes

- `readOnly = true` at class level; each write method overrides with `@Transactional`.
- The `update` method does not call `save()` — the entity is already in the persistence context and changes are flushed automatically at transaction commit.
- `toResponse()` is private. If another service needs the same mapping, extract a `<Resource>Mapper` class rather than making it public or duplicating the logic.
- Throw `EntityNotFoundException` (from `jakarta.persistence`) or a custom domain exception; let `GlobalExceptionHandler` map it to HTTP 404.
