# Project Setup

> Loaded when: user is initializing a new project, adding starter dependencies, or configuring application properties.

## Starter Selection

Recommend starters based on stated needs:

| Need | Starter |
|---|---|
| REST API | `spring-boot-starter-web` |
| Database (JPA) | `spring-boot-starter-data-jpa` + JDBC driver |
| Security | `spring-boot-starter-security` |
| Validation | `spring-boot-starter-validation` |
| Testing | `spring-boot-starter-test` (included by default) |
| Actuator / health | `spring-boot-starter-actuator` |
| Caching | `spring-boot-starter-cache` |

Always recommend `spring-boot-devtools` (scope: runtime, optional) for local development.

## Maven pom.xml Structure

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>3.3.0</version>
</parent>

<properties>
  <java.version>21</java.version>
</properties>

<dependencies>
  <!-- add starters here -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>

<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

## application.yml Structure

Prefer `application.yml` over `application.properties` for readability. Use this baseline:

```yaml
spring:
  application:
    name: my-service

  datasource:
    url: ${DB_URL:jdbc:postgresql://localhost:5432/mydb}
    username: ${DB_USER:postgres}
    password: ${DB_PASSWORD:}
    driver-class-name: org.postgresql.Driver

  jpa:
    hibernate:
      ddl-auto: validate          # never 'create-drop' in production
    show-sql: false
    open-in-view: false           # always disable; prevents N+1 via lazy loading leak

server:
  port: 8080
  shutdown: graceful

logging:
  level:
    root: INFO
    com.example: DEBUG
```

## Profile Configuration

Use Spring profiles for environment-specific values, not conditional logic in code.

```
src/main/resources/
├── application.yml          # shared defaults
├── application-dev.yml      # local development overrides
├── application-test.yml     # test overrides (loaded by @ActiveProfiles("test"))
└── application-prod.yml     # production overrides (never commit secrets here)
```

Activate profiles:
- Local: `SPRING_PROFILES_ACTIVE=dev` in IDE run config or `spring.profiles.active=dev` in application.yml
- CI/CD: pass as environment variable `SPRING_PROFILES_ACTIVE=prod`

## Package Structure

Follow this convention for a service named `order-service`:

```
com.example.orderservice
├── OrderServiceApplication.java
├── config/           # @Configuration beans, beans that cross layers
├── controller/       # @RestController classes
├── dto/
│   ├── request/      # inbound DTOs
│   └── response/     # outbound DTOs
├── entity/           # @Entity classes
├── exception/        # custom exceptions, @ControllerAdvice handler
├── repository/       # JpaRepository interfaces
└── service/          # @Service classes + interfaces
```
