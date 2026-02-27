---
name: spring-boot-testing
---
# Spring Boot Testing Setup Guide

This skill provides comprehensive documentation for setting up unit and integration testing in Spring Boot applications, emphasizing auto-configuration and best practices from the Baeldung reference patterns.

## Overview

Spring Boot provides powerful testing support with auto-configured test annotations that bootstrap only the necessary parts of the application context. This approach ensures fast, focused tests while maintaining the benefits of Spring's dependency injection.

**Note**: For Spring Boot 3.4+, use `@MockitoBean` instead of `@MockBean` for mocking dependencies. `@MockitoBean` provides better performance and more consistent behavior with the latest Mockito features.

## Dependencies

Add these essential testing dependencies to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

The `spring-boot-starter-test` includes:

- JUnit 5
- Mockito
- MockitoBean (Spring Boot 3.4+)
- AssertJ
- Hamcrest
- Spring Test & Spring Boot Test

## Testing Strategy

### 1. Unit Tests

- **Purpose**: Test individual components in isolation
- **Speed**: Fast
- **Dependencies**: Mocked
- **Recommended Annotations**: `@ExtendWith(SpringExtension.class)`, `@WebMvcTest`, `@DataJpaTest`

### 2. Integration Tests

- **Purpose**: Test interaction between layers
- **Speed**: Slower
- **Dependencies**: Real or test database
- **Recommended Annotations**: `@SpringBootTest`

## Auto-Configured Test Annotations

### Controller Layer Testing

Use `@WebMvcTest` for focused controller tests:

```java
@ExtendWith(SpringExtension.class)
@WebMvcTest(EmployeeRestController.class)
public class EmployeeRestControllerTest {

    @Autowired
    private MockMvc mvc;

    @MockitoBean  // Use MockitoBean for Spring Boot 3.4+
    private EmployeeService service;

    @Test
    public void givenEmployees_whenGetEmployees_thenReturnJsonArray() throws Exception {
        Employee alex = new Employee("alex");
        List<Employee> allEmployees = Arrays.asList(alex);

        given(service.getAllEmployees()).willReturn(allEmployees);

        mvc.perform(get("/api/employees")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$", hasSize(1)))
                .andExpect(jsonPath("$[0].name", is(alex.getName())));
    }
}
```

**Benefits**:

- Auto-configures MockMvc
- Loads only controller-related beans
- Mocks dependencies with `@MockitoBean` (Spring Boot 3.4+) or `@MockBean` (older versions)
- Fast execution

### Persistence Layer Testing

Use `@DataJpaTest` for repository tests:

```java
@ExtendWith(SpringExtension.class)
@DataJpaTest
public class EmployeeRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private EmployeeRepository employeeRepository;

    @Test
    public void whenFindByName_thenReturnEmployee() {
        Employee alex = new Employee("alex");
        entityManager.persist(alex);
        entityManager.flush();

        Employee found = employeeRepository.findByName(alex.getName());

        assertThat(found.getName()).isEqualTo(alex.getName());
    }
}
```

**Auto-configuration includes**:

- In-memory H2 database
- Hibernate, Spring Data, DataSource
- Entity scanning
- SQL logging

### Service Layer Testing

Use `@ExtendWith` with `@MockBean` for service tests:

```java
@ExtendWith(SpringExtension.class)
public class EmployeeServiceTest {

    @MockitoBean  // Use MockitoBean for Spring Boot 3.4+
    private EmployeeRepository employeeRepository;

    @Autowired
    private EmployeeService employeeService;

    @BeforeEach
    public void setUp() {
        Employee alex = new Employee("alex");
        when(employeeRepository.findByName(alex.getName())).thenReturn(alex);
    }

    @Test
    public void whenValidName_thenEmployeeShouldBeFound() {
        String name = "alex";
        Employee found = employeeService.getEmployeeByName(name);

        assertThat(found.getName()).isEqualTo(name);
    }
}
```

### Full Integration Testing

Use `@SpringBootTest` for end-to-end tests:

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
@TestPropertySource(locations = "classpath:application-integrationtest.properties")
public class EmployeeRestControllerIntegrationTest {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private EmployeeRepository repository;

    @Test
    public void givenEmployees_whenGetEmployees_thenStatus200() throws Exception {
        createTestEmployee("bob");

        mvc.perform(get("/api/employees")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().contentTypeCompatibleWith(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$[0].name", is("bob")));
    }

    private void createTestEmployee(String name) {
        Employee employee = new Employee(name);
        repository.save(employee);
    }
}
```

## Test Configuration

### Test-Specific Configuration

Use `@TestConfiguration` for test-specific beans:

```java
@TestConfiguration
public class EmployeeServiceTestConfig {

    @Bean
    @Primary
    public EmployeeService employeeService() {
        return new EmployeeServiceImpl() {
            // Test-specific implementation
        };
    }
}
```

Import in tests:

```java
@ExtendWith(SpringExtension.class)
@Import(EmployeeServiceTestConfig.class)
public class EmployeeServiceTest {
    // Test implementation
}
```

### Test Properties

Create test-specific configuration files. Spring Boot supports both `.properties` and `.yml` formats:

**For `.properties` format (application-integrationtest.properties)**:

```properties
spring.datasource.url=jdbc:h2:mem:test
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
logging.level.org.hibernate.SQL=DEBUG
```

**For `.yml` format (application-integrationtest.yml)**:

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:test
  jpa:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.H2Dialect
    hibernate:
      ddl-auto: create-drop
logging:
  level:
    org.hibernate.SQL: DEBUG
```

**Note**: Use the same format as your main application configuration. If your project uses `application.yml`, create `application-integrationtest.yml` for consistency.

## Specialized Auto-Configured Tests

### WebFlux Testing

```java
@WebFluxTest(HomeController.class)
public class HomeControllerTest {
    // Test reactive controllers
}
```

### JSON Testing

```java
@JsonTest
public class EmployeeJsonTest {
    @Autowired
    private JacksonTester<Employee> json;

    @Test
    public void employeeSerializationTest() throws IOException {
        Employee employee = new Employee("John");
        assertThat(json.write(employee)).isEqualToJson("expected.json");
    }
}
```

### REST Client Testing

```java
@RestClientTest(EmployeeClient.class)
public class EmployeeClientTest {
    @Autowired
    private MockRestServiceServer server;

    @Autowired
    private EmployeeClient client;
}
```

### Database Testing

#### JDBC Testing

```java
@JdbcTest
public class EmployeeJdbcTest {
    @Autowired
    private JdbcTemplate jdbcTemplate;
}
```

#### MongoDB Testing

```java
@DataMongoTest
public class EmployeeMongoTest {
    @Autowired
    private MongoTemplate mongoTemplate;
}
```

## Best Practices

### 1. Test Organization

- Keep unit tests in `src/test/java`
- Separate integration tests with `@Test` annotation profiles
- Use descriptive test names

### 2. Auto-Configuration Priority

Always prefer auto-configured test annotations:

- Use `@WebMvcTest` instead of `@SpringBootTest` for controller tests
- Use `@DataJpaTest` instead of `@SpringBootTest` for repository tests
- Use `@SpringBootTest` only when necessary

### 3. Mocking Strategy

- Mock external dependencies with `@MockitoBean` (Spring Boot 3.4+) or `@MockBean` (older versions)
- Use `@TestConfiguration` for test-specific beans
- Avoid mocking the layer you're testing

### 4. Test Data Management

- Use `TestEntityManager` for JPA tests
- Clean up test data in `@AfterEach`
- Use `@Transactional` for rollback behavior

### 5. Performance Considerations

- Run unit tests separately from integration tests
- Use in-memory databases for testing
- Limit `@SpringBootTest` usage

## Test Execution Profiles

### Maven Configuration

```xml
<profiles>
    <profile>
        <id>unit-tests</id>
        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <configuration>
                        <includes>
                            <include>**/*Test.java</include>
                        </includes>
                        <excludes>
                            <exclude>**/*IntegrationTest.java</exclude>
                        </excludes>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
    <profile>
        <id>integration-tests</id>
        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-failsafe-plugin</artifactId>
                    <configuration>
                        <includes>
                            <include>**/*IntegrationTest.java</include>
                        </includes>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

### Running Tests

```bash
# Unit tests only
mvn test -Punit-tests

# Integration tests only
mvn verify -Pintegration-tests

# All tests
mvn test
```

## Common Patterns

### Service Layer with Repository Mock

```java
@ExtendWith(SpringExtension.class)
public class EmployeeServiceTest {

    @MockitoBean  // Use MockitoBean for Spring Boot 3.4+
    private EmployeeRepository repository;

    @InjectMocks
    @Autowired
    private EmployeeService service;

    @Test
    public void whenFindById_thenReturnEmployee() {
        // Given
        Employee employee = new Employee("John");
        when(repository.findById(1L)).thenReturn(Optional.of(employee));

        // When
        Employee found = service.findById(1L);

        // Then
        assertThat(found.getName()).isEqualTo("John");
    }
}
```

### Controller with Service Mock

```java
@WebMvcTest(EmployeeController.class)
public class EmployeeControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockitoBean  // Use MockitoBean for Spring Boot 3.4+
    private EmployeeService service;

    @Test
    public void getEmployee_shouldReturnEmployee() throws Exception {
        Employee employee = new Employee("John");
        when(service.findById(1L)).thenReturn(employee);

        mockMvc.perform(get("/employees/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is("John")));
    }
}
```

### Repository Integration Test

```java
@DataJpaTest
@TestPropertySource(properties = {
    "spring.jpa.hibernate.ddl-auto=create-drop"
})
public class EmployeeRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private EmployeeRepository repository;

    @Test
    public void findByName_shouldReturnEmployee() {
        Employee employee = new Employee("John");
        entityManager.persistAndFlush(employee);

        Optional<Employee> found = repository.findByName("John");

        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("John");
    }
}
```

## Conclusion

Spring Boot's auto-configured test annotations provide a powerful, efficient way to structure tests. By choosing the right annotation for each test type and leveraging auto-configuration, you can create fast, maintainable test suites that provide comprehensive coverage while minimizing setup complexity.

Key takeaways:

1. **Prefer auto-configured annotations** over manual configuration
2. **Choose the right test slice** for the layer you're testing
3. **Mock dependencies** appropriately with `@MockitoBean` (Spring Boot 3.4+) or `@MockBean` (older versions)
4. **Separate test types** for better performance and organization
5. **Use test-specific configuration** for specialized test needs
