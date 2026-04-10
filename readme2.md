building & verifying project java

cd Backend_API/simpleapi
.\mvnw.cmd clean verify

## Testing

> 2-1 What are testcontainers?

**Testcontainers** is a Java library that provides lightweight, disposable Docker containers for testing. It allows you to run real services (databases, message queues, etc.) in Docker during tests instead of using mocks or in-memory databases.

### Key Benefits:

1. **Real services** - Test with actual PostgreSQL, MySQL, Redis, etc., not mocks
2. **Isolation** - Each test runs in its own container (no state pollution)
3. **Cleanup** - Containers are automatically stopped and removed after tests
4. **Consistency** - Tests run the same way locally and in CI/CD pipelines
5. **No setup pain** - No need to manually install databases

### Example in Your Project:

In `pom.xml`, you have:
```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>${testcontainers.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>${testcontainers.version}</version>
    <scope>test</scope>
</dependency>
```

This enables your integration tests to:
- Spin up a real PostgreSQL container
- Run tests against it
- Automatically clean up when tests finish

### How It Works:

```
Test starts → Docker container launched → Run test → Container destroyed
```

**Note:** In your current setup, we're using **H2 (in-memory database)** for faster tests, but testcontainers is available if you want real PostgreSQL testing later.
