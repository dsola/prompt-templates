# Prompt Templates Collection

A comprehensive collection of reusable prompt templates for building different types of applications, following best practices, and implementing proper coding standards across multiple programming languages.

## 📚 Overview

This repository contains structured prompt templates that guide AI assistants in generating high-quality code for various application types. Each template includes:

- Clear purpose and use case descriptions
- Comprehensive technical specifications
- Architecture principles and best practices
- Real-world examples
- References to official documentation

## 🗂️ Repository Structure

```
prompt-templates/
├── applications/          # Application-specific templates
│   ├── frontend/         # Frontend frameworks (React, Vue)
│   ├── api/              # API templates (REST, GraphQL)
│   ├── services/         # Backend services (Microservices)
│   └── mcp-servers/      # Model Context Protocol servers
├── best-practices/       # Development best practices
│   ├── adr-template.md   # Architecture Decision Records
│   └── code-conventions.md  # Code style guides
├── languages/            # Language-specific templates
│   ├── python.md
│   ├── typescript.md
│   └── java.md
└── examples/             # Example implementations
```

## 🚀 Application Templates

### Frontend Applications

#### [React Single Page Application](applications/frontend/react-spa.md)
Build modern React SPAs with TypeScript, state management, routing, and best practices.

**Key Topics:**
- Component architecture and hooks
- State management (Redux, Zustand, Context API)
- React Router v6
- Testing with React Testing Library
- Performance optimization

#### [Vue.js Application](applications/frontend/vue-spa.md)
Create Vue 3 applications using Composition API, Pinia, and modern patterns.

**Key Topics:**
- Composition API and composables
- Pinia state management
- Vue Router 4
- TypeScript integration
- Testing with Vitest

### API Templates

#### [RESTful API](applications/api/rest-api.md)
Design and implement production-ready REST APIs with proper patterns and security.

**Key Topics:**
- RESTful design principles
- Authentication and authorization (JWT, OAuth2)
- Input validation and error handling
- API versioning
- OpenAPI/Swagger documentation
- Rate limiting and security

#### [GraphQL API](applications/api/graphql-api.md)
Build flexible GraphQL APIs with proper schema design and performance optimization.

**Key Topics:**
- Schema design and type system
- Resolvers and DataLoader
- Authentication and authorization
- Query complexity analysis
- Subscriptions for real-time updates
- Performance optimization

### Backend Services

#### [Microservice](applications/services/microservice.md)
Create scalable microservices with proper communication patterns and observability.

**Key Topics:**
- Service boundaries and domain-driven design
- Synchronous (REST/gRPC) and asynchronous (message queues) communication
- Circuit breaker and resilience patterns
- Distributed tracing and logging
- Service discovery
- Saga pattern for distributed transactions

### MCP Servers

#### [Model Context Protocol Server](applications/mcp-servers/mcp-server.md)
Build MCP servers that expose tools and resources to AI assistants.

**Key Topics:**
- MCP protocol implementation
- Tool definition and validation
- Resource providers
- Prompt templates
- Error handling
- Security considerations

## 📋 Best Practices

### [Architecture Decision Records (ADR)](best-practices/adr-template.md)
Document important architectural decisions with context, alternatives, and consequences.

**Includes:**
- ADR template structure
- Real-world examples (database selection, API design)
- Best practices for writing ADRs
- When to create an ADR

### [Code Conventions and Style Guide](best-practices/code-conventions.md)
Establish team-wide coding standards and best practices.

**Covers:**
- General principles (SOLID, DRY, YAGNI)
- Language-specific conventions
- File organization and naming
- Testing standards
- Git conventions (commits, branches, PRs)
- Documentation requirements

## 💻 Language-Specific Templates

### [Python](languages/python.md)
Modern Python development with type hints, FastAPI/Flask/Django, and proper project structure.

**Topics:**
- Type hints and mypy
- Project structure (FastAPI, Flask, Django, libraries)
- Dataclasses and Pydantic models
- Testing with pytest
- Async/await patterns
- Configuration management
- Poetry/pip-tools for dependencies

### [TypeScript/JavaScript](languages/typescript.md)
Build type-safe Node.js applications with Express, NestJS, or other frameworks.

**Topics:**
- TypeScript strict mode and type system
- Node.js project structure
- Express and NestJS patterns
- Dependency injection
- Testing with Vitest/Jest
- Validation with Zod
- Error handling patterns

### [Java](languages/java.md)
Enterprise-grade Java applications with Spring Boot and modern Java practices.

**Topics:**
- Spring Boot 3.x architecture
- JPA/Hibernate with Spring Data
- REST API development
- Security with Spring Security
- Bean validation
- Testing with JUnit 5 and Mockito
- Maven/Gradle configuration

### [Go](languages/go.md)
High-performance Go applications with idiomatic patterns and clean architecture.

**Topics:**
- Go project structure and conventions
- Gin/Echo/Chi web frameworks
- Database access with sqlx/GORM
- Goroutines and channels
- Interface-based design
- Table-driven testing
- Dependency injection patterns

### [Rust](languages/rust.md)
Safe, concurrent Rust applications with zero-cost abstractions.

**Topics:**
- Ownership and borrowing
- Actix-web/Rocket/Axum frameworks
- Async/await with Tokio
- SQLx for database access
- Custom error types with thiserror
- Testing with mockall
- Type-safe configurations

## 🎯 How to Use These Templates

### 1. Choose the Right Template

Browse the repository structure and select the template that matches your needs:
- Building a frontend? → Check `applications/frontend/`
- Creating an API? → Check `applications/api/`
- Need coding standards? → Check `best-practices/`
- Language-specific guidance? → Check `languages/`

### 2. Customize the Template

Each template includes placeholders in square brackets like `[Framework Name]` or `[Describe your application]`. Replace these with your specific requirements.

### 3. Example Usage

**Simple Example:**
```
Use the React SPA template to create a task management application with:
- User authentication
- CRUD operations for tasks
- Task filtering and sorting
- Redux Toolkit for state management
- Tailwind CSS for styling
```

**Detailed Example:**
```
Following the Python template, create a FastAPI application for a blog platform with:

Features:
- User registration and authentication (JWT)
- CRUD operations for posts and comments
- Post search and filtering
- PostgreSQL database with SQLAlchemy
- Pydantic models for validation
- pytest for testing

Technical requirements:
- Python 3.11
- FastAPI with async/await
- SQLAlchemy 2.0 (async)
- Poetry for dependency management
- Ruff for linting
- mypy for type checking
- 80%+ test coverage

Follow the project structure and best practices outlined in the Python template.
```

## 🏗️ Template Components

Each template typically includes:

### 1. **Purpose**
Clear description of what the template is for and when to use it.

### 2. **Technical Stack**
Recommended technologies, frameworks, and tools with version recommendations.

### 3. **Architecture Principles**
Core architectural concepts and design patterns to follow.

### 4. **Project Structure**
Detailed directory structure with explanations for each component.

### 5. **Code Examples**
Practical code samples demonstrating key concepts and patterns.

### 6. **Best Practices**
Curated list of dos and don'ts specific to the template's domain.

### 7. **Testing Strategy**
Guidelines for unit, integration, and end-to-end testing.

### 8. **Security Guidelines**
Security considerations and best practices.

### 9. **References**
Links to official documentation and additional resources.

## 🔍 Finding the Right Template

### By Application Type
- **Frontend Web Apps** → `applications/frontend/`
- **Backend APIs** → `applications/api/`
- **Distributed Systems** → `applications/services/`
- **AI Integration** → `applications/mcp-servers/`

### By Technology
- **React/Vue** → `applications/frontend/`
- **Python** → `languages/python.md` or FastAPI examples
- **Node.js/TypeScript** → `languages/typescript.md`
- **Java/Spring** → `languages/java.md`
- **Go** → `languages/go.md`
- **Rust** → `languages/rust.md`

### By Task
- **Starting a new project** → Choose relevant application template + language template
- **Documenting decisions** → `best-practices/adr-template.md`
- **Setting up code standards** → `best-practices/code-conventions.md`
- **Learning best practices** → Browse all templates in your domain

## 🤝 Contributing

Feel free to contribute additional templates, improvements, or examples:

1. Follow the existing template structure
2. Include comprehensive examples
3. Add proper references
4. Ensure templates are technology-agnostic where possible
5. Provide real-world use cases

## 📖 Additional Resources

### General Software Development
- [Clean Code by Robert C. Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
- [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
- [The Pragmatic Programmer](https://www.amazon.com/Pragmatic-Programmer-journey-mastery-Anniversary/dp/0135957052)

### Architecture and Design
- [Building Microservices by Sam Newman](https://samnewman.io/books/building_microservices/)
- [Domain-Driven Design by Eric Evans](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)
- [Software Architecture Patterns by Mark Richards](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/)

### Web Development
- [MDN Web Docs](https://developer.mozilla.org/)
- [Web.dev by Google](https://web.dev/)
- [Frontend Masters](https://frontendmasters.com/)

### API Design
- [REST API Tutorial](https://restfulapi.net/)
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)
- [API Security Best Practices](https://owasp.org/www-project-api-security/)

## 📝 License

This repository is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🎉 Acknowledgments

These templates are compiled from industry best practices, official documentation, and real-world experience building production applications across various domains and technologies.

---

**Happy Coding!** 🚀

If you find these templates useful, consider starring ⭐ this repository!
