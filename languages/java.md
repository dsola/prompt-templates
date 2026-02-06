# Java Application Template

## Purpose
This template provides comprehensive guidance for building enterprise-grade Java applications with Spring Boot and modern Java practices.

## Use Case
Use when building backend services, microservices, or enterprise applications with Java and the Spring ecosystem.

## Template

```
Create a Java application with the following specifications:

## Application Overview
[Describe the application's purpose and main functionality]

## Technical Stack
- Java Version: 17+ LTS (or Java 21)
- Framework: Spring Boot 3.x
- Build Tool: [Maven / Gradle]
- ORM: Spring Data JPA with Hibernate
- Database: [PostgreSQL / MySQL / MongoDB]
- Testing: JUnit 5, Mockito, Spring Boot Test
- Code Quality: SonarQube, Checkstyle
- Documentation: SpringDoc OpenAPI (Swagger)

## Architecture Principles
1. Follow Clean Architecture / Hexagonal Architecture
2. Use SOLID principles
3. Implement layered architecture (Controller → Service → Repository)
4. Use dependency injection (Spring IoC)
5. Apply design patterns appropriately
6. Follow Java coding conventions
7. Use Optional for null-safety

## Project Structure

### Spring Boot Application (Maven)
```
project-name/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/company/project/
│   │   │       ├── ProjectApplication.java
│   │   │       ├── config/              # Configuration classes
│   │   │       │   ├── SecurityConfig.java
│   │   │       │   ├── DatabaseConfig.java
│   │   │       │   └── OpenApiConfig.java
│   │   │       ├── controller/          # REST controllers
│   │   │       │   ├── UserController.java
│   │   │       │   └── AuthController.java
│   │   │       ├── service/             # Business logic
│   │   │       │   ├── UserService.java
│   │   │       │   ├── impl/
│   │   │       │   │   └── UserServiceImpl.java
│   │   │       │   └── AuthService.java
│   │   │       ├── repository/          # Data access
│   │   │       │   ├── UserRepository.java
│   │   │       │   └── RoleRepository.java
│   │   │       ├── model/               # Domain entities
│   │   │       │   ├── entity/
│   │   │       │   │   ├── User.java
│   │   │       │   │   └── Role.java
│   │   │       │   └── dto/             # Data Transfer Objects
│   │   │       │       ├── UserDTO.java
│   │   │       │       ├── CreateUserRequest.java
│   │   │       │       └── UserResponse.java
│   │   │       ├── exception/           # Custom exceptions
│   │   │       │   ├── ResourceNotFoundException.java
│   │   │       │   ├── ValidationException.java
│   │   │       │   └── GlobalExceptionHandler.java
│   │   │       ├── security/            # Security components
│   │   │       │   ├── JwtTokenProvider.java
│   │   │       │   ├── JwtAuthenticationFilter.java
│   │   │       │   └── CustomUserDetailsService.java
│   │   │       ├── util/                # Utility classes
│   │   │       │   └── DateUtils.java
│   │   │       └── validator/           # Custom validators
│   │   │           └── EmailValidator.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       └── db/
│   │           └── migration/           # Flyway migrations
│   │               └── V1__Initial_Schema.sql
│   └── test/
│       └── java/
│           └── com/company/project/
│               ├── controller/
│               ├── service/
│               └── repository/
├── pom.xml                              # Maven configuration
├── Dockerfile
└── README.md
```

## Code Style & Best Practices

### Entity Classes
```java
import jakarta.persistence.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import java.time.LocalDateTime;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true, length = 100)
    private String username;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String password;
    
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

### DTOs with Validation
```java
import jakarta.validation.constraints.*;
import lombok.Data;

@Data
public class CreateUserRequest {
    
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be between 3 and 50 characters")
    @Pattern(regexp = "^[a-zA-Z0-9_-]+$", message = "Username can only contain letters, numbers, underscore and hyphen")
    private String username;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    private String email;
    
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
}

@Data
public class UserResponse {
    private Long id;
    private String username;
    private String email;
    private Set<String> roles;
    private LocalDateTime createdAt;
    
    // Factory method for converting entity to response
    public static UserResponse from(User user) {
        UserResponse response = new UserResponse();
        response.setId(user.getId());
        response.setUsername(user.getUsername());
        response.setEmail(user.getEmail());
        response.setRoles(
            user.getRoles().stream()
                .map(Role::getName)
                .collect(Collectors.toSet())
        );
        response.setCreatedAt(user.getCreatedAt());
        return response;
    }
}
```

### Repository Layer
```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import java.util.Optional;
import java.util.List;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    Optional<User> findByUsername(String username);
    
    Optional<User> findByEmail(String email);
    
    boolean existsByUsername(String username);
    
    boolean existsByEmail(String email);
    
    @Query("SELECT u FROM User u JOIN FETCH u.roles WHERE u.username = :username")
    Optional<User> findByUsernameWithRoles(@Param("username") String username);
    
    @Query("SELECT u FROM User u WHERE LOWER(u.username) LIKE LOWER(CONCAT('%', :search, '%')) " +
           "OR LOWER(u.email) LIKE LOWER(CONCAT('%', :search, '%'))")
    List<User> searchUsers(@Param("search") String search);
}
```

### Service Layer
```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;
import java.util.Optional;

@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)
public class UserServiceImpl implements UserService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final RoleRepository roleRepository;
    
    @Override
    public Optional<UserResponse> getUserById(Long id) {
        log.debug("Fetching user with id: {}", id);
        return userRepository.findById(id)
            .map(UserResponse::from);
    }
    
    @Override
    public List<UserResponse> getAllUsers() {
        log.debug("Fetching all users");
        return userRepository.findAll().stream()
            .map(UserResponse::from)
            .toList();
    }
    
    @Override
    @Transactional
    public UserResponse createUser(CreateUserRequest request) {
        log.info("Creating new user with username: {}", request.getUsername());
        
        // Validate unique constraints
        if (userRepository.existsByUsername(request.getUsername())) {
            throw new ValidationException("Username already exists");
        }
        
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new ValidationException("Email already exists");
        }
        
        // Create user entity
        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));
        
        // Assign default role
        Role userRole = roleRepository.findByName("ROLE_USER")
            .orElseThrow(() -> new IllegalStateException("Default role not found"));
        user.getRoles().add(userRole);
        
        // Save and return
        User savedUser = userRepository.save(user);
        log.info("User created successfully with id: {}", savedUser.getId());
        
        return UserResponse.from(savedUser);
    }
    
    @Override
    @Transactional
    public UserResponse updateUser(Long id, UpdateUserRequest request) {
        log.info("Updating user with id: {}", id);
        
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
        
        // Update fields
        if (request.getEmail() != null && !request.getEmail().equals(user.getEmail())) {
            if (userRepository.existsByEmail(request.getEmail())) {
                throw new ValidationException("Email already exists");
            }
            user.setEmail(request.getEmail());
        }
        
        User updatedUser = userRepository.save(user);
        log.info("User updated successfully: {}", id);
        
        return UserResponse.from(updatedUser);
    }
    
    @Override
    @Transactional
    public void deleteUser(Long id) {
        log.info("Deleting user with id: {}", id);
        
        if (!userRepository.existsById(id)) {
            throw new ResourceNotFoundException("User not found with id: " + id);
        }
        
        userRepository.deleteById(id);
        log.info("User deleted successfully: {}", id);
    }
}
```

### Controller Layer
```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
@Tag(name = "User Management", description = "Endpoints for managing users")
public class UserController {
    
    private final UserService userService;
    
    @GetMapping
    @Operation(summary = "Get all users")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<ApiResponse<List<UserResponse>>> getAllUsers() {
        List<UserResponse> users = userService.getAllUsers();
        return ResponseEntity.ok(ApiResponse.success(users));
    }
    
    @GetMapping("/{id}")
    @Operation(summary = "Get user by ID")
    public ResponseEntity<ApiResponse<UserResponse>> getUserById(@PathVariable Long id) {
        return userService.getUserById(id)
            .map(user -> ResponseEntity.ok(ApiResponse.success(user)))
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    @Operation(summary = "Create new user")
    @ResponseStatus(HttpStatus.CREATED)
    public ResponseEntity<ApiResponse<UserResponse>> createUser(
            @Valid @RequestBody CreateUserRequest request) {
        UserResponse user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(ApiResponse.success(user, "User created successfully"));
    }
    
    @PutMapping("/{id}")
    @Operation(summary = "Update user")
    public ResponseEntity<ApiResponse<UserResponse>> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UpdateUserRequest request) {
        UserResponse user = userService.updateUser(id, request);
        return ResponseEntity.ok(ApiResponse.success(user, "User updated successfully"));
    }
    
    @DeleteMapping("/{id}")
    @Operation(summary = "Delete user")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<ApiResponse<Void>> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.ok(ApiResponse.success(null, "User deleted successfully"));
    }
}
```

### Exception Handling
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        log.error("Resource not found: {}", ex.getMessage());
        ErrorResponse error = new ErrorResponse(
            "RESOURCE_NOT_FOUND",
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        log.error("Validation error: {}", ex.getMessage());
        ErrorResponse error = new ErrorResponse(
            "VALIDATION_ERROR",
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleMethodArgumentNotValid(
            MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        ErrorResponse error = new ErrorResponse(
            "VALIDATION_ERROR",
            "Validation failed",
            errors
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        log.error("Unexpected error occurred", ex);
        ErrorResponse error = new ErrorResponse(
            "INTERNAL_ERROR",
            "An unexpected error occurred"
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

### Configuration Classes
```java
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("User Management API")
                .version("1.0")
                .description("REST API for user management"));
    }
}

@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### Testing
```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@DisplayName("User Service Tests")
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private PasswordEncoder passwordEncoder;
    
    @InjectMocks
    private UserServiceImpl userService;
    
    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }
    
    @Test
    @DisplayName("Should create user successfully")
    void testCreateUser_Success() {
        // Given
        CreateUserRequest request = new CreateUserRequest();
        request.setUsername("testuser");
        request.setEmail("test@example.com");
        request.setPassword("password123");
        
        User user = new User();
        user.setId(1L);
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        
        when(userRepository.existsByUsername(anyString())).thenReturn(false);
        when(userRepository.existsByEmail(anyString())).thenReturn(false);
        when(passwordEncoder.encode(anyString())).thenReturn("encoded_password");
        when(userRepository.save(any(User.class))).thenReturn(user);
        
        // When
        UserResponse response = userService.createUser(request);
        
        // Then
        assertNotNull(response);
        assertEquals("testuser", response.getUsername());
        assertEquals("test@example.com", response.getEmail());
        verify(userRepository).save(any(User.class));
    }
    
    @Test
    @DisplayName("Should throw exception when username exists")
    void testCreateUser_UsernameExists() {
        // Given
        CreateUserRequest request = new CreateUserRequest();
        request.setUsername("existinguser");
        
        when(userRepository.existsByUsername(anyString())).thenReturn(true);
        
        // When & Then
        assertThrows(ValidationException.class, () -> {
            userService.createUser(request);
        });
        
        verify(userRepository, never()).save(any(User.class));
    }
}
```

### Application Properties (YAML)
```yaml
spring:
  application:
    name: user-management-service
  
  datasource:
    url: jdbc:postgresql://localhost:5432/userdb
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver
  
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
  
  flyway:
    enabled: true
    locations: classpath:db/migration

server:
  port: 8080
  error:
    include-message: always
    include-stacktrace: on_param

logging:
  level:
    root: INFO
    com.company.project: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

## Best Practices Summary

1. **Use Lombok** - Reduce boilerplate code
2. **Follow naming conventions** - Classes (PascalCase), methods (camelCase)
3. **Use Optional** - For nullable return types
4. **Validate inputs** - Use Bean Validation annotations
5. **Handle exceptions** - Global exception handler
6. **Write tests** - Aim for 80%+ coverage
7. **Use interfaces** - For service layer contracts
8. **Proper logging** - Use SLF4J with appropriate levels
9. **Transaction management** - Use @Transactional appropriately
10. **Security** - Implement proper authentication and authorization
```

## Example Prompt

```
Create a Spring Boot REST API for an e-commerce order management system with:

Features:
- User authentication with JWT
- Product catalog management
- Shopping cart functionality
- Order processing workflow
- Payment integration
- Order history and tracking

Technical requirements:
- Java 17
- Spring Boot 3.x
- Spring Security with JWT
- PostgreSQL with Spring Data JPA
- Redis for caching
- Bean validation
- OpenAPI documentation
- Unit and integration tests
- Docker containerization

Include:
- Layered architecture
- Exception handling
- Logging
- Database migrations (Flyway)
- API documentation
- 80%+ test coverage
```

## References
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [Spring Security](https://spring.io/projects/spring-security)
- [Java Best Practices](https://www.baeldung.com/)
- [Effective Java (Joshua Bloch)](https://www.oreilly.com/library/view/effective-java/9780134686097/)
