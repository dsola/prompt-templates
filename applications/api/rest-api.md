# RESTful API Template

## Purpose
This template provides guidance for building production-ready RESTful APIs with proper design patterns, security, and scalability considerations.

## Use Case
Use when creating backend APIs that follow REST principles, require proper authentication, validation, and error handling.

## Template

```
Create a RESTful API with the following specifications:

## API Overview
[Describe the API's purpose, main resources, and business domain]

## Technical Requirements
- Language/Framework: [Node.js/Express / Python/FastAPI / Java/Spring Boot / Go/Gin / .NET/ASP.NET Core]
- Database: [PostgreSQL / MongoDB / MySQL / Redis]
- ORM/ODM: [TypeORM / Prisma / SQLAlchemy / Mongoose / GORM / Entity Framework]
- Authentication: JWT / OAuth2 / Session-based
- API Documentation: OpenAPI/Swagger
- Testing: [Jest / Pytest / JUnit / Go Test]

## Architecture Principles
1. Follow RESTful design principles (proper HTTP methods and status codes)
2. Implement layered architecture (Controllers → Services → Repositories)
3. Use dependency injection for loose coupling
4. Apply SOLID principles
5. Implement proper error handling and logging
6. Use environment-based configuration
7. Implement API versioning (v1, v2)

## Project Structure
```
src/
├── controllers/        # HTTP request handlers
├── services/          # Business logic layer
├── repositories/      # Data access layer
├── models/            # Data models/entities
├── middleware/        # Custom middleware (auth, validation, logging)
├── routes/            # API route definitions
├── utils/             # Helper functions
├── validators/        # Input validation schemas
├── config/            # Configuration files
├── types/             # TypeScript types (if applicable)
└── server.ts/main.py  # Application entry point
```

## RESTful Endpoints Design
Follow these conventions:
- `GET /api/v1/resources` - List all resources
- `GET /api/v1/resources/:id` - Get single resource
- `POST /api/v1/resources` - Create new resource
- `PUT /api/v1/resources/:id` - Update entire resource
- `PATCH /api/v1/resources/:id` - Partial update
- `DELETE /api/v1/resources/:id` - Delete resource

## Key Features to Implement

### 1. Authentication & Authorization
- JWT token-based authentication
- Refresh token mechanism
- Role-based access control (RBAC)
- API key authentication for service-to-service

### 2. Request Validation
- Input validation using schemas (Joi, Zod, Pydantic)
- Request body sanitization
- Type checking and coercion

### 3. Error Handling
- Centralized error handling middleware
- Consistent error response format
- Appropriate HTTP status codes
- Error logging and monitoring

### 4. Security
- CORS configuration
- Rate limiting
- Helmet.js for security headers (Node.js)
- SQL injection prevention
- XSS protection
- CSRF tokens for state-changing operations
- Input sanitization

### 5. Logging & Monitoring
- Structured logging (Winston, Pino, Loguru)
- Request/response logging
- Performance metrics
- Error tracking (Sentry, DataDog)
- Health check endpoints

### 6. Database
- Connection pooling
- Transaction management
- Database migrations
- Query optimization
- Proper indexing

## Response Format Standards

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation successful"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": { ... }
  }
}
```

### Pagination
```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

## Best Practices
1. Use proper HTTP status codes (200, 201, 400, 401, 403, 404, 500)
2. Implement API versioning from the start
3. Use query parameters for filtering, sorting, and pagination
4. Return meaningful error messages
5. Document all endpoints with OpenAPI/Swagger
6. Implement request rate limiting
7. Use HTTPS in production
8. Validate all inputs
9. Implement proper logging
10. Write comprehensive tests

## Testing Strategy
- Unit tests for services and utilities (80%+ coverage)
- Integration tests for API endpoints
- Load testing for performance validation
- Security testing (OWASP Top 10)

## Performance Optimization
- Implement caching (Redis)
- Use database query optimization
- Implement pagination for large datasets
- Use compression for responses
- Enable HTTP/2
- Implement CDN for static content

## Documentation
- Use OpenAPI 3.0 specification
- Generate interactive API documentation (Swagger UI)
- Include examples for all endpoints
- Document authentication flow
- Provide Postman collection
```

## Example Prompt

```
Create a RESTful API for a blog platform with the following endpoints:

Resources:
1. Users (authentication, profiles)
2. Posts (CRUD operations, pagination, search)
3. Comments (nested under posts)
4. Tags (categorization)

Use Node.js with Express and TypeScript, PostgreSQL with Prisma ORM, JWT authentication, and implement the following:
- User registration and login
- CRUD operations for posts
- Pagination and filtering
- Role-based permissions (admin, author, reader)
- Input validation with Zod
- OpenAPI documentation
- Unit and integration tests

Follow the layered architecture and best practices outlined above.
```

## References
- [REST API Tutorial](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)
- [OpenAPI Specification](https://swagger.io/specification/)
- [OWASP API Security](https://owasp.org/www-project-api-security/)
- [JSON:API Specification](https://jsonapi.org/)
