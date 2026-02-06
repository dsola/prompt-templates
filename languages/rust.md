# Rust Application Template

## Purpose
This template provides comprehensive guidance for building safe, concurrent, and high-performance Rust applications with modern patterns and best practices.

## Use Case
Use when building systems programming projects, web services, CLI tools, or any application requiring memory safety, concurrency, and performance.

## Template

```
Create a Rust application with the following specifications:

## Application Overview
[Describe the application's purpose and main functionality]

## Technical Stack
- Rust Version: 1.70+ (stable)
- Web Framework: [Actix-web / Rocket / Axum / Warp]
- ORM: [SeaORM / Diesel / sqlx]
- Database: [PostgreSQL / MySQL / MongoDB]
- Async Runtime: Tokio
- Serialization: Serde
- Testing: Built-in test framework, mockall
- CLI: clap (for CLI tools)

## Architecture Principles
1. Leverage Rust's ownership system for memory safety
2. Use type system for compile-time guarantees
3. Implement error handling with Result and custom errors
4. Apply async/await for concurrent operations
5. Follow Rust API guidelines
6. Use traits for abstraction
7. Minimize unsafe code

## Project Structure

### Web API Application
```
project-name/
├── src/
│   ├── main.rs                    # Application entry point
│   ├── lib.rs                     # Library root (if needed)
│   ├── api/                       # HTTP layer
│   │   ├── mod.rs
│   │   ├── routes.rs
│   │   ├── handlers/
│   │   │   ├── mod.rs
│   │   │   ├── user.rs
│   │   │   └── auth.rs
│   │   └── middleware/
│   │       ├── mod.rs
│   │       ├── auth.rs
│   │       └── logger.rs
│   ├── services/                  # Business logic
│   │   ├── mod.rs
│   │   ├── user_service.rs
│   │   └── auth_service.rs
│   ├── repositories/              # Data access
│   │   ├── mod.rs
│   │   ├── user_repository.rs
│   │   └── postgres/
│   │       └── user_postgres.rs
│   ├── models/                    # Domain models
│   │   ├── mod.rs
│   │   ├── user.rs
│   │   └── error.rs
│   ├── dto/                       # Data Transfer Objects
│   │   ├── mod.rs
│   │   ├── request.rs
│   │   └── response.rs
│   ├── config/                    # Configuration
│   │   ├── mod.rs
│   │   └── settings.rs
│   └── utils/                     # Utility functions
│       ├── mod.rs
│       └── validation.rs
├── migrations/                    # Database migrations
├── tests/                         # Integration tests
│   ├── api_tests.rs
│   └── common/
│       └── mod.rs
├── Cargo.toml
├── Cargo.lock
├── .env.example
└── README.md
```

## Code Style & Best Practices

### Domain Models
```rust
use chrono::{DateTime, Utc};
use serde::{Deserialize, Serialize};
use sqlx::FromRow;

#[derive(Debug, Clone, Serialize, Deserialize, FromRow)]
pub struct User {
    pub id: i64,
    pub username: String,
    pub email: String,
    #[serde(skip_serializing)]
    pub password_hash: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

impl User {
    pub fn new(username: String, email: String, password_hash: String) -> Self {
        let now = Utc::now();
        Self {
            id: 0, // Will be set by database
            username,
            email,
            password_hash,
            created_at: now,
            updated_at: now,
        }
    }
}
```

### Custom Error Types
```rust
use thiserror::Error;
use actix_web::{HttpResponse, ResponseError};
use std::fmt;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("User not found")]
    NotFound,
    
    #[error("Validation error: {0}")]
    Validation(String),
    
    #[error("Unauthorized")]
    Unauthorized,
    
    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),
    
    #[error("Internal server error")]
    Internal(String),
}

// Implement ResponseError for Actix-web
impl ResponseError for AppError {
    fn error_response(&self) -> HttpResponse {
        match self {
            AppError::NotFound => HttpResponse::NotFound().json(ErrorResponse {
                code: "NOT_FOUND",
                message: self.to_string(),
            }),
            AppError::Validation(msg) => HttpResponse::BadRequest().json(ErrorResponse {
                code: "VALIDATION_ERROR",
                message: msg.clone(),
            }),
            AppError::Unauthorized => HttpResponse::Unauthorized().json(ErrorResponse {
                code: "UNAUTHORIZED",
                message: self.to_string(),
            }),
            AppError::Database(_) => HttpResponse::InternalServerError().json(ErrorResponse {
                code: "DATABASE_ERROR",
                message: "Database operation failed".to_string(),
            }),
            AppError::Internal(_) => HttpResponse::InternalServerError().json(ErrorResponse {
                code: "INTERNAL_ERROR",
                message: "Internal server error".to_string(),
            }),
        }
    }
}

#[derive(Serialize)]
struct ErrorResponse {
    code: &'static str,
    message: String,
}

pub type Result<T> = std::result::Result<T, AppError>;
```

### DTOs with Validation
```rust
use serde::{Deserialize, Serialize};
use validator::Validate;

#[derive(Debug, Deserialize, Validate)]
pub struct CreateUserRequest {
    #[validate(length(min = 3, max = 50))]
    pub username: String,
    
    #[validate(email)]
    pub email: String,
    
    #[validate(length(min = 8))]
    pub password: String,
}

#[derive(Debug, Serialize)]
pub struct UserResponse {
    pub id: i64,
    pub username: String,
    pub email: String,
    pub created_at: DateTime<Utc>,
}

impl From<User> for UserResponse {
    fn from(user: User) -> Self {
        Self {
            id: user.id,
            username: user.username,
            email: user.email,
            created_at: user.created_at,
        }
    }
}

#[derive(Serialize)]
pub struct ApiResponse<T> {
    pub success: bool,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub data: Option<T>,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub error: Option<ErrorInfo>,
}

impl<T> ApiResponse<T> {
    pub fn success(data: T) -> Self {
        Self {
            success: true,
            data: Some(data),
            error: None,
        }
    }
    
    pub fn error(code: String, message: String) -> Self {
        Self {
            success: false,
            data: None,
            error: Some(ErrorInfo { code, message }),
        }
    }
}
```

### Repository Trait and Implementation
```rust
use async_trait::async_trait;
use crate::models::{User, AppError, Result};

#[async_trait]
pub trait UserRepository: Send + Sync {
    async fn create(&self, user: User) -> Result<User>;
    async fn find_by_id(&self, id: i64) -> Result<Option<User>>;
    async fn find_by_email(&self, email: &str) -> Result<Option<User>>;
    async fn update(&self, user: User) -> Result<User>;
    async fn delete(&self, id: i64) -> Result<()>;
}

// PostgreSQL implementation
use sqlx::{PgPool, postgres::PgRow, Row};

pub struct PostgresUserRepository {
    pool: PgPool,
}

impl PostgresUserRepository {
    pub fn new(pool: PgPool) -> Self {
        Self { pool }
    }
}

#[async_trait]
impl UserRepository for PostgresUserRepository {
    async fn create(&self, mut user: User) -> Result<User> {
        let row = sqlx::query(
            r#"
            INSERT INTO users (username, email, password_hash, created_at, updated_at)
            VALUES ($1, $2, $3, $4, $5)
            RETURNING id, username, email, password_hash, created_at, updated_at
            "#
        )
        .bind(&user.username)
        .bind(&user.email)
        .bind(&user.password_hash)
        .bind(&user.created_at)
        .bind(&user.updated_at)
        .fetch_one(&self.pool)
        .await?;
        
        Ok(User {
            id: row.get("id"),
            username: row.get("username"),
            email: row.get("email"),
            password_hash: row.get("password_hash"),
            created_at: row.get("created_at"),
            updated_at: row.get("updated_at"),
        })
    }
    
    async fn find_by_id(&self, id: i64) -> Result<Option<User>> {
        let user = sqlx::query_as::<_, User>(
            "SELECT id, username, email, password_hash, created_at, updated_at FROM users WHERE id = $1"
        )
        .bind(id)
        .fetch_optional(&self.pool)
        .await?;
        
        Ok(user)
    }
    
    async fn find_by_email(&self, email: &str) -> Result<Option<User>> {
        let user = sqlx::query_as::<_, User>(
            "SELECT id, username, email, password_hash, created_at, updated_at FROM users WHERE email = $1"
        )
        .bind(email)
        .fetch_optional(&self.pool)
        .await?;
        
        Ok(user)
    }
}
```

### Service Layer
```rust
use std::sync::Arc;
use bcrypt::{hash, verify, DEFAULT_COST};
use crate::models::{User, AppError, Result};
use crate::dto::{CreateUserRequest, UserResponse};
use crate::repositories::UserRepository;

pub struct UserService {
    repository: Arc<dyn UserRepository>,
}

impl UserService {
    pub fn new(repository: Arc<dyn UserRepository>) -> Self {
        Self { repository }
    }
    
    pub async fn create_user(&self, req: CreateUserRequest) -> Result<UserResponse> {
        // Validate request
        req.validate()
            .map_err(|e| AppError::Validation(e.to_string()))?;
        
        // Check if user already exists
        if let Some(_) = self.repository.find_by_email(&req.email).await? {
            return Err(AppError::Validation("Email already exists".to_string()));
        }
        
        // Hash password
        let password_hash = hash(req.password.as_bytes(), DEFAULT_COST)
            .map_err(|e| AppError::Internal(e.to_string()))?;
        
        // Create user
        let user = User::new(req.username, req.email, password_hash);
        let created_user = self.repository.create(user).await?;
        
        Ok(UserResponse::from(created_user))
    }
    
    pub async fn get_user_by_id(&self, id: i64) -> Result<UserResponse> {
        let user = self.repository
            .find_by_id(id)
            .await?
            .ok_or(AppError::NotFound)?;
        
        Ok(UserResponse::from(user))
    }
    
    pub async fn authenticate(&self, email: &str, password: &str) -> Result<UserResponse> {
        let user = self.repository
            .find_by_email(email)
            .await?
            .ok_or(AppError::Unauthorized)?;
        
        // Verify password
        let valid = verify(password.as_bytes(), &user.password_hash)
            .map_err(|e| AppError::Internal(e.to_string()))?;
        
        if !valid {
            return Err(AppError::Unauthorized);
        }
        
        Ok(UserResponse::from(user))
    }
}
```

### HTTP Handlers (Actix-web)
```rust
use actix_web::{web, HttpResponse, Responder};
use std::sync::Arc;
use crate::services::UserService;
use crate::dto::{CreateUserRequest, ApiResponse};

pub struct UserHandler {
    service: Arc<UserService>,
}

impl UserHandler {
    pub fn new(service: Arc<UserService>) -> Self {
        Self { service }
    }
}

pub async fn create_user(
    service: web::Data<Arc<UserService>>,
    req: web::Json<CreateUserRequest>,
) -> impl Responder {
    match service.create_user(req.into_inner()).await {
        Ok(user) => HttpResponse::Created().json(ApiResponse::success(user)),
        Err(e) => e.error_response(),
    }
}

pub async fn get_user(
    service: web::Data<Arc<UserService>>,
    path: web::Path<i64>,
) -> impl Responder {
    let user_id = path.into_inner();
    
    match service.get_user_by_id(user_id).await {
        Ok(user) => HttpResponse::Ok().json(ApiResponse::success(user)),
        Err(e) => e.error_response(),
    }
}

// Route configuration
pub fn configure_routes(cfg: &mut web::ServiceConfig) {
    cfg.service(
        web::scope("/api/v1/users")
            .route("", web::post().to(create_user))
            .route("/{id}", web::get().to(get_user))
    );
}
```

### Configuration
```rust
use config::{Config, ConfigError, Environment, File};
use serde::Deserialize;

#[derive(Debug, Deserialize, Clone)]
pub struct Settings {
    pub server: ServerSettings,
    pub database: DatabaseSettings,
    pub jwt: JwtSettings,
    pub log: LogSettings,
}

#[derive(Debug, Deserialize, Clone)]
pub struct ServerSettings {
    pub host: String,
    pub port: u16,
}

#[derive(Debug, Deserialize, Clone)]
pub struct DatabaseSettings {
    pub url: String,
    pub max_connections: u32,
}

#[derive(Debug, Deserialize, Clone)]
pub struct JwtSettings {
    pub secret: String,
    pub expiration_hours: i64,
}

#[derive(Debug, Deserialize, Clone)]
pub struct LogSettings {
    pub level: String,
}

impl Settings {
    pub fn new() -> Result<Self, ConfigError> {
        let mut builder = Config::builder()
            .add_source(File::with_name("config/default").required(false))
            .add_source(Environment::with_prefix("APP").separator("__"));
        
        builder.build()?.try_deserialize()
    }
}
```

### Testing
```rust
#[cfg(test)]
mod tests {
    use super::*;
    use mockall::predicate::*;
    use mockall::mock;
    
    // Mock repository
    mock! {
        UserRepo {}
        
        #[async_trait]
        impl UserRepository for UserRepo {
            async fn create(&self, user: User) -> Result<User>;
            async fn find_by_id(&self, id: i64) -> Result<Option<User>>;
            async fn find_by_email(&self, email: &str) -> Result<Option<User>>;
            async fn update(&self, user: User) -> Result<User>;
            async fn delete(&self, id: i64) -> Result<()>;
        }
    }
    
    #[tokio::test]
    async fn test_create_user_success() {
        // Arrange
        let mut mock_repo = MockUserRepo::new();
        let test_user = User::new(
            "testuser".to_string(),
            "test@example.com".to_string(),
            "hashed_password".to_string(),
        );
        
        mock_repo
            .expect_find_by_email()
            .with(eq("test@example.com"))
            .times(1)
            .returning(|_| Ok(None));
        
        mock_repo
            .expect_create()
            .times(1)
            .returning(move |user| Ok(user));
        
        let service = UserService::new(Arc::new(mock_repo));
        
        let req = CreateUserRequest {
            username: "testuser".to_string(),
            email: "test@example.com".to_string(),
            password: "password123".to_string(),
        };
        
        // Act
        let result = service.create_user(req).await;
        
        // Assert
        assert!(result.is_ok());
        let user = result.unwrap();
        assert_eq!(user.username, "testuser");
        assert_eq!(user.email, "test@example.com");
    }
    
    #[tokio::test]
    async fn test_get_user_not_found() {
        // Arrange
        let mut mock_repo = MockUserRepo::new();
        
        mock_repo
            .expect_find_by_id()
            .with(eq(999))
            .times(1)
            .returning(|_| Ok(None));
        
        let service = UserService::new(Arc::new(mock_repo));
        
        // Act
        let result = service.get_user_by_id(999).await;
        
        // Assert
        assert!(result.is_err());
        match result {
            Err(AppError::NotFound) => (),
            _ => panic!("Expected NotFound error"),
        }
    }
}
```

### Main Application
```rust
use actix_web::{web, App, HttpServer, middleware};
use sqlx::postgres::PgPoolOptions;
use std::sync::Arc;
use tracing_subscriber;

mod api;
mod config;
mod dto;
mod models;
mod repositories;
mod services;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // Initialize logger
    tracing_subscriber::fmt::init();
    
    // Load configuration
    let settings = config::Settings::new()
        .expect("Failed to load configuration");
    
    // Create database pool
    let pool = PgPoolOptions::new()
        .max_connections(settings.database.max_connections)
        .connect(&settings.database.url)
        .await
        .expect("Failed to connect to database");
    
    // Run migrations
    sqlx::migrate!("./migrations")
        .run(&pool)
        .await
        .expect("Failed to run migrations");
    
    // Initialize repositories
    let user_repository = Arc::new(
        repositories::PostgresUserRepository::new(pool.clone())
    );
    
    // Initialize services
    let user_service = Arc::new(
        services::UserService::new(user_repository)
    );
    
    // Server address
    let server_addr = format!("{}:{}", settings.server.host, settings.server.port);
    
    println!("Starting server at {}", server_addr);
    
    // Start HTTP server
    HttpServer::new(move || {
        App::new()
            .wrap(middleware::Logger::default())
            .wrap(middleware::Compress::default())
            .app_data(web::Data::new(user_service.clone()))
            .configure(api::handlers::configure_routes)
    })
    .bind(&server_addr)?
    .run()
    .await
}
```

### Cargo.toml
```toml
[package]
name = "my-api"
version = "0.1.0"
edition = "2021"

[dependencies]
actix-web = "4"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres", "chrono"] }
chrono = { version = "0.4", features = ["serde"] }
bcrypt = "0.15"
validator = { version = "0.16", features = ["derive"] }
thiserror = "1"
async-trait = "0.1"
tracing = "0.1"
tracing-subscriber = "0.3"
config = "0.13"

[dev-dependencies]
mockall = "0.12"
```

## Best Practices Summary

1. **Use ownership system** - Leverage borrow checker for safety
2. **Handle errors explicitly** - Use Result and custom error types
3. **Use traits** - For abstraction and polymorphism
4. **Async/await** - For concurrent operations
5. **Test thoroughly** - Unit and integration tests
6. **Use type system** - Encode invariants in types
7. **Avoid unwrap** - Use proper error handling
8. **Documentation** - Write doc comments
9. **Follow conventions** - Use clippy and rustfmt
10. **Minimize unsafe** - Use only when necessary
```

## Example Prompt

```
Create a Rust REST API for a real-time chat application with:

Features:
- User authentication with JWT
- WebSocket support for real-time messaging
- Message persistence
- Room management
- PostgreSQL database
- Comprehensive error handling
- Unit and integration tests

Technical requirements:
- Rust 1.70+
- Actix-web framework
- SQLx for database access
- Tokio async runtime
- Serde for serialization
- JWT authentication
- WebSocket support
- Docker containerization

Include:
- Clean architecture
- Custom error types
- Repository pattern
- Async/await throughout
- Proper testing with mocks
- Configuration management
```

## References
- [The Rust Programming Language Book](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- [Actix-web Documentation](https://actix.rs/)
- [SQLx Documentation](https://github.com/launchbadge/sqlx)
