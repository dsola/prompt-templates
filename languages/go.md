# Go Application Template

## Purpose
This template provides comprehensive guidance for building high-performance Go applications with idiomatic patterns, clean architecture, and modern Go practices.

## Use Case
Use when building backend services, APIs, CLI tools, or system programs with Go's concurrency and performance advantages.

## Template

```
Create a Go application with the following specifications:

## Application Overview
[Describe the application's purpose and main functionality]

## Technical Stack
- Go Version: 1.21+ (or 1.20+ minimum)
- Framework: [Gin / Echo / Chi / Fiber / net/http]
- Database: [PostgreSQL / MongoDB / MySQL]
- ORM/Query Builder: [GORM / sqlx / pgx]
- Testing: testing package, testify
- Migration: golang-migrate
- Documentation: Swagger with swag
- Configuration: Viper or envconfig

## Architecture Principles
1. Follow Go idioms and conventions
2. Use interfaces for abstraction
3. Implement clean architecture layers
4. Leverage Go's concurrency (goroutines, channels)
5. Keep packages small and focused
6. Use dependency injection
7. Error handling with explicit checks

## Project Structure

### Standard Go API Application
```
project-name/
├── cmd/
│   └── api/
│       └── main.go                 # Application entry point
├── internal/                       # Private application code
│   ├── api/                       # HTTP handlers
│   │   ├── handler/
│   │   │   ├── user_handler.go
│   │   │   └── auth_handler.go
│   │   ├── middleware/
│   │   │   ├── auth.go
│   │   │   ├── logger.go
│   │   │   └── recovery.go
│   │   └── router/
│   │       └── router.go
│   ├── service/                   # Business logic
│   │   ├── user_service.go
│   │   └── auth_service.go
│   ├── repository/                # Data access layer
│   │   ├── user_repository.go
│   │   └── postgres/
│   │       └── user_postgres.go
│   ├── model/                     # Domain models
│   │   ├── user.go
│   │   └── error.go
│   ├── dto/                       # Data Transfer Objects
│   │   ├── request/
│   │   │   └── user_request.go
│   │   └── response/
│   │       └── user_response.go
│   └── config/                    # Configuration
│       └── config.go
├── pkg/                           # Public reusable packages
│   ├── logger/
│   │   └── logger.go
│   ├── validator/
│   │   └── validator.go
│   └── database/
│       └── postgres.go
├── migrations/                    # Database migrations
│   ├── 000001_create_users_table.up.sql
│   └── 000001_create_users_table.down.sql
├── tests/                        # Integration tests
├── scripts/                      # Utility scripts
├── docs/                         # API documentation
├── .env.example
├── go.mod
├── go.sum
├── Makefile
└── README.md
```

## Code Style & Best Practices

### Domain Models
```go
package model

import (
    "time"
    "errors"
)

// User represents a user entity
type User struct {
    ID        int64     `json:"id" db:"id"`
    Username  string    `json:"username" db:"username"`
    Email     string    `json:"email" db:"email"`
    Password  string    `json:"-" db:"password"` // Never expose in JSON
    CreatedAt time.Time `json:"created_at" db:"created_at"`
    UpdatedAt time.Time `json:"updated_at" db:"updated_at"`
}

// Validation errors
var (
    ErrInvalidUsername = errors.New("invalid username")
    ErrInvalidEmail    = errors.New("invalid email")
    ErrUserNotFound    = errors.New("user not found")
)

// Validate validates user data
func (u *User) Validate() error {
    if len(u.Username) < 3 || len(u.Username) > 50 {
        return ErrInvalidUsername
    }
    
    if !isValidEmail(u.Email) {
        return ErrInvalidEmail
    }
    
    return nil
}
```

### DTOs (Data Transfer Objects)
```go
package dto

import "time"

// CreateUserRequest represents user creation request
type CreateUserRequest struct {
    Username string `json:"username" binding:"required,min=3,max=50"`
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required,min=8"`
}

// UserResponse represents user API response
type UserResponse struct {
    ID        int64     `json:"id"`
    Username  string    `json:"username"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

// APIResponse is the standard API response wrapper
type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   *APIError   `json:"error,omitempty"`
}

// APIError represents an API error
type APIError struct {
    Code    string `json:"code"`
    Message string `json:"message"`
}
```

### Repository Interface and Implementation
```go
package repository

import (
    "context"
    "project/internal/model"
)

// UserRepository defines the interface for user data access
type UserRepository interface {
    Create(ctx context.Context, user *model.User) error
    GetByID(ctx context.Context, id int64) (*model.User, error)
    GetByEmail(ctx context.Context, email string) (*model.User, error)
    Update(ctx context.Context, user *model.User) error
    Delete(ctx context.Context, id int64) error
    List(ctx context.Context, limit, offset int) ([]*model.User, error)
}

// PostgreSQL implementation
package postgres

import (
    "context"
    "database/sql"
    "project/internal/model"
    "project/internal/repository"
)

type userRepository struct {
    db *sql.DB
}

// NewUserRepository creates a new user repository
func NewUserRepository(db *sql.DB) repository.UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) Create(ctx context.Context, user *model.User) error {
    query := `
        INSERT INTO users (username, email, password, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5)
        RETURNING id
    `
    
    err := r.db.QueryRowContext(
        ctx,
        query,
        user.Username,
        user.Email,
        user.Password,
        user.CreatedAt,
        user.UpdatedAt,
    ).Scan(&user.ID)
    
    if err != nil {
        return err
    }
    
    return nil
}

func (r *userRepository) GetByID(ctx context.Context, id int64) (*model.User, error) {
    query := `
        SELECT id, username, email, password, created_at, updated_at
        FROM users
        WHERE id = $1
    `
    
    user := &model.User{}
    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &user.Password,
        &user.CreatedAt,
        &user.UpdatedAt,
    )
    
    if err == sql.ErrNoRows {
        return nil, model.ErrUserNotFound
    }
    
    if err != nil {
        return nil, err
    }
    
    return user, nil
}
```

### Service Layer
```go
package service

import (
    "context"
    "time"
    "project/internal/model"
    "project/internal/repository"
    "project/internal/dto"
    "golang.org/x/crypto/bcrypt"
)

// UserService handles user business logic
type UserService struct {
    repo   repository.UserRepository
    logger Logger
}

// NewUserService creates a new user service
func NewUserService(repo repository.UserRepository, logger Logger) *UserService {
    return &UserService{
        repo:   repo,
        logger: logger,
    }
}

// CreateUser creates a new user
func (s *UserService) CreateUser(ctx context.Context, req *dto.CreateUserRequest) (*dto.UserResponse, error) {
    s.logger.Info("Creating user", "username", req.Username)
    
    // Hash password
    hashedPassword, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
    if err != nil {
        s.logger.Error("Failed to hash password", "error", err)
        return nil, err
    }
    
    // Create user model
    user := &model.User{
        Username:  req.Username,
        Email:     req.Email,
        Password:  string(hashedPassword),
        CreatedAt: time.Now(),
        UpdatedAt: time.Now(),
    }
    
    // Validate
    if err := user.Validate(); err != nil {
        return nil, err
    }
    
    // Save to repository
    if err := s.repo.Create(ctx, user); err != nil {
        s.logger.Error("Failed to create user", "error", err)
        return nil, err
    }
    
    s.logger.Info("User created successfully", "id", user.ID)
    
    // Return response
    return &dto.UserResponse{
        ID:        user.ID,
        Username:  user.Username,
        Email:     user.Email,
        CreatedAt: user.CreatedAt,
    }, nil
}

// GetUserByID retrieves a user by ID
func (s *UserService) GetUserByID(ctx context.Context, id int64) (*dto.UserResponse, error) {
    user, err := s.repo.GetByID(ctx, id)
    if err != nil {
        return nil, err
    }
    
    return &dto.UserResponse{
        ID:        user.ID,
        Username:  user.Username,
        Email:     user.Email,
        CreatedAt: user.CreatedAt,
    }, nil
}
```

### HTTP Handlers (Gin Framework)
```go
package handler

import (
    "net/http"
    "strconv"
    "project/internal/service"
    "project/internal/dto"
    "github.com/gin-gonic/gin"
)

// UserHandler handles user-related HTTP requests
type UserHandler struct {
    service *service.UserService
}

// NewUserHandler creates a new user handler
func NewUserHandler(service *service.UserService) *UserHandler {
    return &UserHandler{service: service}
}

// CreateUser godoc
// @Summary Create a new user
// @Description Create a new user with the provided information
// @Tags users
// @Accept json
// @Produce json
// @Param user body dto.CreateUserRequest true "User information"
// @Success 201 {object} dto.APIResponse
// @Failure 400 {object} dto.APIResponse
// @Failure 500 {object} dto.APIResponse
// @Router /api/v1/users [post]
func (h *UserHandler) CreateUser(c *gin.Context) {
    var req dto.CreateUserRequest
    
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, dto.APIResponse{
            Success: false,
            Error: &dto.APIError{
                Code:    "VALIDATION_ERROR",
                Message: err.Error(),
            },
        })
        return
    }
    
    user, err := h.service.CreateUser(c.Request.Context(), &req)
    if err != nil {
        statusCode := http.StatusInternalServerError
        errCode := "INTERNAL_ERROR"
        
        // Map domain errors to HTTP status codes
        switch err {
        case model.ErrInvalidUsername, model.ErrInvalidEmail:
            statusCode = http.StatusBadRequest
            errCode = "VALIDATION_ERROR"
        }
        
        c.JSON(statusCode, dto.APIResponse{
            Success: false,
            Error: &dto.APIError{
                Code:    errCode,
                Message: err.Error(),
            },
        })
        return
    }
    
    c.JSON(http.StatusCreated, dto.APIResponse{
        Success: true,
        Data:    user,
    })
}

// GetUser godoc
// @Summary Get user by ID
// @Description Get user information by ID
// @Tags users
// @Produce json
// @Param id path int true "User ID"
// @Success 200 {object} dto.APIResponse
// @Failure 404 {object} dto.APIResponse
// @Failure 500 {object} dto.APIResponse
// @Router /api/v1/users/{id} [get]
func (h *UserHandler) GetUser(c *gin.Context) {
    id, err := strconv.ParseInt(c.Param("id"), 10, 64)
    if err != nil {
        c.JSON(http.StatusBadRequest, dto.APIResponse{
            Success: false,
            Error: &dto.APIError{
                Code:    "INVALID_ID",
                Message: "Invalid user ID",
            },
        })
        return
    }
    
    user, err := h.service.GetUserByID(c.Request.Context(), id)
    if err != nil {
        if err == model.ErrUserNotFound {
            c.JSON(http.StatusNotFound, dto.APIResponse{
                Success: false,
                Error: &dto.APIError{
                    Code:    "NOT_FOUND",
                    Message: "User not found",
                },
            })
            return
        }
        
        c.JSON(http.StatusInternalServerError, dto.APIResponse{
            Success: false,
            Error: &dto.APIError{
                Code:    "INTERNAL_ERROR",
                Message: "Internal server error",
            },
        })
        return
    }
    
    c.JSON(http.StatusOK, dto.APIResponse{
        Success: true,
        Data:    user,
    })
}
```

### Middleware
```go
package middleware

import (
    "time"
    "github.com/gin-gonic/gin"
    "go.uber.org/zap"
)

// Logger returns a middleware that logs HTTP requests
func Logger(logger *zap.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        path := c.Request.URL.Path
        query := c.Request.URL.RawQuery
        
        c.Next()
        
        latency := time.Since(start)
        
        logger.Info("HTTP Request",
            zap.String("method", c.Request.Method),
            zap.String("path", path),
            zap.String("query", query),
            zap.Int("status", c.Writer.Status()),
            zap.Duration("latency", latency),
            zap.String("ip", c.ClientIP()),
        )
    }
}

// Recovery returns a middleware that recovers from panics
func Recovery(logger *zap.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        defer func() {
            if err := recover(); err != nil {
                logger.Error("Panic recovered",
                    zap.Any("error", err),
                    zap.String("path", c.Request.URL.Path),
                )
                
                c.JSON(500, dto.APIResponse{
                    Success: false,
                    Error: &dto.APIError{
                        Code:    "INTERNAL_ERROR",
                        Message: "Internal server error",
                    },
                })
            }
        }()
        
        c.Next()
    }
}
```

### Configuration
```go
package config

import (
    "github.com/spf13/viper"
)

// Config holds application configuration
type Config struct {
    Server   ServerConfig
    Database DatabaseConfig
    JWT      JWTConfig
    Log      LogConfig
}

type ServerConfig struct {
    Port         string
    ReadTimeout  int
    WriteTimeout int
}

type DatabaseConfig struct {
    Host     string
    Port     string
    User     string
    Password string
    DBName   string
    SSLMode  string
}

type JWTConfig struct {
    Secret     string
    Expiration int
}

type LogConfig struct {
    Level string
}

// Load loads configuration from environment variables
func Load() (*Config, error) {
    viper.AutomaticEnv()
    
    config := &Config{
        Server: ServerConfig{
            Port:         viper.GetString("PORT"),
            ReadTimeout:  viper.GetInt("READ_TIMEOUT"),
            WriteTimeout: viper.GetInt("WRITE_TIMEOUT"),
        },
        Database: DatabaseConfig{
            Host:     viper.GetString("DB_HOST"),
            Port:     viper.GetString("DB_PORT"),
            User:     viper.GetString("DB_USER"),
            Password: viper.GetString("DB_PASSWORD"),
            DBName:   viper.GetString("DB_NAME"),
            SSLMode:  viper.GetString("DB_SSLMODE"),
        },
        JWT: JWTConfig{
            Secret:     viper.GetString("JWT_SECRET"),
            Expiration: viper.GetInt("JWT_EXPIRATION"),
        },
        Log: LogConfig{
            Level: viper.GetString("LOG_LEVEL"),
        },
    }
    
    return config, nil
}
```

### Testing
```go
package service_test

import (
    "context"
    "testing"
    "project/internal/model"
    "project/internal/service"
    "project/internal/dto"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
)

// Mock repository
type MockUserRepository struct {
    mock.Mock
}

func (m *MockUserRepository) Create(ctx context.Context, user *model.User) error {
    args := m.Called(ctx, user)
    return args.Error(0)
}

func (m *MockUserRepository) GetByID(ctx context.Context, id int64) (*model.User, error) {
    args := m.Called(ctx, id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*model.User), args.Error(1)
}

// Tests
func TestUserService_CreateUser(t *testing.T) {
    mockRepo := new(MockUserRepository)
    logger := NewTestLogger()
    service := service.NewUserService(mockRepo, logger)
    
    ctx := context.Background()
    req := &dto.CreateUserRequest{
        Username: "testuser",
        Email:    "test@example.com",
        Password: "password123",
    }
    
    mockRepo.On("Create", ctx, mock.AnythingOfType("*model.User")).Return(nil)
    
    user, err := service.CreateUser(ctx, req)
    
    assert.NoError(t, err)
    assert.NotNil(t, user)
    assert.Equal(t, "testuser", user.Username)
    mockRepo.AssertExpectations(t)
}

func TestUserService_GetUserByID_NotFound(t *testing.T) {
    mockRepo := new(MockUserRepository)
    logger := NewTestLogger()
    service := service.NewUserService(mockRepo, logger)
    
    ctx := context.Background()
    
    mockRepo.On("GetByID", ctx, int64(999)).Return(nil, model.ErrUserNotFound)
    
    user, err := service.GetUserByID(ctx, 999)
    
    assert.Error(t, err)
    assert.Nil(t, user)
    assert.Equal(t, model.ErrUserNotFound, err)
    mockRepo.AssertExpectations(t)
}
```

### Main Application
```go
package main

import (
    "context"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
    
    "project/internal/api/handler"
    "project/internal/api/middleware"
    "project/internal/config"
    "project/internal/repository/postgres"
    "project/internal/service"
    "project/pkg/database"
    "project/pkg/logger"
    
    "github.com/gin-gonic/gin"
)

func main() {
    // Load configuration
    cfg, err := config.Load()
    if err != nil {
        log.Fatal("Failed to load config:", err)
    }
    
    // Initialize logger
    logger := logger.New(cfg.Log.Level)
    defer logger.Sync()
    
    // Connect to database
    db, err := database.NewPostgresDB(cfg.Database)
    if err != nil {
        logger.Fatal("Failed to connect to database", "error", err)
    }
    defer db.Close()
    
    // Initialize repositories
    userRepo := postgres.NewUserRepository(db)
    
    // Initialize services
    userService := service.NewUserService(userRepo, logger)
    
    // Initialize handlers
    userHandler := handler.NewUserHandler(userService)
    
    // Setup router
    router := gin.New()
    router.Use(middleware.Logger(logger))
    router.Use(middleware.Recovery(logger))
    
    // API routes
    v1 := router.Group("/api/v1")
    {
        users := v1.Group("/users")
        {
            users.POST("", userHandler.CreateUser)
            users.GET("/:id", userHandler.GetUser)
        }
    }
    
    // Create server
    srv := &http.Server{
        Addr:         ":" + cfg.Server.Port,
        Handler:      router,
        ReadTimeout:  time.Duration(cfg.Server.ReadTimeout) * time.Second,
        WriteTimeout: time.Duration(cfg.Server.WriteTimeout) * time.Second,
    }
    
    // Start server
    go func() {
        logger.Info("Starting server", "port", cfg.Server.Port)
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            logger.Fatal("Failed to start server", "error", err)
        }
    }()
    
    // Graceful shutdown
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    
    logger.Info("Shutting down server...")
    
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    if err := srv.Shutdown(ctx); err != nil {
        logger.Fatal("Server forced to shutdown", "error", err)
    }
    
    logger.Info("Server exited")
}
```

### Makefile
```makefile
.PHONY: build run test clean migrate-up migrate-down docker-build

build:
	go build -o bin/api cmd/api/main.go

run:
	go run cmd/api/main.go

test:
	go test -v -cover ./...

test-coverage:
	go test -v -coverprofile=coverage.out ./...
	go tool cover -html=coverage.out

lint:
	golangci-lint run

migrate-up:
	migrate -path migrations -database "postgresql://user:pass@localhost:5432/dbname?sslmode=disable" up

migrate-down:
	migrate -path migrations -database "postgresql://user:pass@localhost:5432/dbname?sslmode=disable" down

docker-build:
	docker build -t api:latest .

clean:
	rm -rf bin/
	go clean -cache
```

## Best Practices Summary

1. **Follow Go conventions** - Use gofmt, golint, go vet
2. **Use interfaces** - For abstraction and testing
3. **Error handling** - Always check errors, don't panic
4. **Context usage** - Pass context for cancellation
5. **Dependency injection** - Constructor injection pattern
6. **Testing** - Table-driven tests, use testify
7. **Naming** - Short, descriptive names
8. **Packages** - Small, focused packages
9. **Concurrency** - Use goroutines and channels wisely
10. **Documentation** - Write clear comments and godoc
```

## Example Prompt

```
Create a Go REST API for a task management system with:

Features:
- User authentication with JWT
- CRUD operations for tasks
- Task filtering and search
- PostgreSQL database
- Comprehensive error handling
- Unit and integration tests
- API documentation with Swagger

Technical requirements:
- Go 1.21+
- Gin web framework
- GORM for ORM
- golang-migrate for migrations
- zap for logging
- testify for testing
- Docker containerization

Include:
- Clean architecture (handler → service → repository)
- Dependency injection
- Middleware (auth, logging, recovery)
- Configuration management
- Graceful shutdown
- 80%+ test coverage
```

## References
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Standard Go Project Layout](https://github.com/golang-standards/project-layout)
- [Go by Example](https://gobyexample.com/)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
