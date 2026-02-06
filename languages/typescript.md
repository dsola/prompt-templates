# TypeScript/JavaScript Application Template

## Purpose
This template provides comprehensive guidance for building modern TypeScript and JavaScript applications with best practices, type safety, and maintainable architecture.

## Use Case
Use when starting Node.js backends, full-stack applications, or any TypeScript/JavaScript project requiring production-ready structure.

## Template

```
Create a TypeScript/JavaScript application with the following specifications:

## Application Overview
[Describe the application's purpose and main functionality]

## Technical Stack
- Runtime: Node.js 18+ LTS
- Language: TypeScript 5.0+
- Package Manager: [npm / pnpm / yarn]
- Framework: [Express / NestJS / Fastify / Hono]
- Testing: Vitest or Jest with ts-jest
- Code Quality: ESLint + Prettier
- Type Checking: TypeScript strict mode
- Build Tool: [tsc / esbuild / tsup]

## Architecture Principles
1. Use TypeScript strict mode
2. Follow SOLID principles
3. Implement layered architecture (Controllers → Services → Repositories)
4. Use dependency injection
5. Separate concerns clearly
6. Type everything - minimal use of 'any'
7. Prefer composition over inheritance

## Project Structure

### Express/Node.js Application
```
project-name/
├── src/
│   ├── index.ts                 # Application entry point
│   ├── server.ts                # Server setup
│   ├── app.ts                   # Express app configuration
│   ├── controllers/             # Request handlers
│   │   ├── user.controller.ts
│   │   └── auth.controller.ts
│   ├── services/                # Business logic
│   │   ├── user.service.ts
│   │   └── auth.service.ts
│   ├── repositories/            # Data access layer
│   │   ├── user.repository.ts
│   │   └── base.repository.ts
│   ├── models/                  # Data models/entities
│   │   ├── user.model.ts
│   │   └── types.ts
│   ├── middleware/              # Custom middleware
│   │   ├── auth.middleware.ts
│   │   ├── error.middleware.ts
│   │   └── validation.middleware.ts
│   ├── routes/                  # Route definitions
│   │   ├── index.ts
│   │   ├── user.routes.ts
│   │   └── auth.routes.ts
│   ├── validators/              # Input validation schemas
│   │   └── user.validator.ts
│   ├── utils/                   # Utility functions
│   │   ├── logger.ts
│   │   └── helpers.ts
│   ├── config/                  # Configuration
│   │   ├── database.ts
│   │   └── environment.ts
│   ├── types/                   # TypeScript types/interfaces
│   │   ├── express.d.ts
│   │   └── common.ts
│   └── constants/               # Application constants
│       └── errors.ts
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── scripts/                     # Utility scripts
├── .env.example
├── tsconfig.json
├── package.json
├── vitest.config.ts
└── README.md
```

### NestJS Application
```
project-name/
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   ├── users/                   # Feature module
│   │   ├── users.module.ts
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── dto/
│   │   │   ├── create-user.dto.ts
│   │   │   └── update-user.dto.ts
│   │   ├── entities/
│   │   │   └── user.entity.ts
│   │   └── users.controller.spec.ts
│   ├── auth/
│   ├── common/                  # Shared code
│   │   ├── decorators/
│   │   ├── guards/
│   │   ├── filters/
│   │   └── interceptors/
│   └── config/
├── test/
└── dist/
```

## Code Style & Best Practices

### Type Definitions
```typescript
// Use interfaces for object shapes
interface User {
  id: string;
  username: string;
  email: string;
  createdAt: Date;
}

// Use type aliases for unions, intersections, and primitives
type UserId = string;
type UserRole = 'admin' | 'user' | 'guest';
type APIResponse<T> = {
  success: boolean;
  data?: T;
  error?: string;
};

// Use generics for reusable types
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  create(data: Partial<T>): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

// Use utility types
type CreateUserInput = Omit<User, 'id' | 'createdAt'>;
type UpdateUserInput = Partial<CreateUserInput>;
type UserResponse = Pick<User, 'id' | 'username' | 'email'>;

// Discriminated unions for complex types
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

// Use const assertions for literal types
const ROLES = ['admin', 'user', 'guest'] as const;
type Role = typeof ROLES[number]; // 'admin' | 'user' | 'guest'
```

### Function Definitions
```typescript
// Always type parameters and return values
function calculateTotal(price: number, quantity: number, taxRate?: number): number {
  const subtotal = price * quantity;
  const tax = taxRate ? subtotal * taxRate : 0;
  return subtotal + tax;
}

// Use async/await for asynchronous operations
async function fetchUser(userId: string): Promise<User> {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error(`Failed to fetch user: ${response.statusText}`);
    }
    return await response.json();
  } catch (error) {
    logger.error('Error fetching user', { userId, error });
    throw error;
  }
}

// Use arrow functions for callbacks and short functions
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
const sum = numbers.reduce((acc, n) => acc + n, 0);

// Type function parameters with objects for many arguments
interface CreateUserOptions {
  username: string;
  email: string;
  role?: UserRole;
  isActive?: boolean;
}

async function createUser(options: CreateUserOptions): Promise<User> {
  const { username, email, role = 'user', isActive = true } = options;
  // Implementation
}

// Use function overloads for complex signatures
function format(value: string): string;
function format(value: number): string;
function format(value: Date): string;
function format(value: string | number | Date): string {
  if (typeof value === 'string') return value;
  if (typeof value === 'number') return value.toString();
  return value.toISOString();
}
```

### Classes and OOP
```typescript
// Use classes for entities and services
class UserService {
  constructor(
    private readonly repository: UserRepository,
    private readonly logger: Logger
  ) {}

  async getUserById(id: string): Promise<User> {
    this.logger.info('Fetching user', { id });
    
    const user = await this.repository.findById(id);
    if (!user) {
      throw new NotFoundError(`User ${id} not found`);
    }
    
    return user;
  }

  async createUser(input: CreateUserInput): Promise<User> {
    this.validateUserInput(input);
    
    const user = await this.repository.create({
      ...input,
      createdAt: new Date()
    });
    
    this.logger.info('User created', { userId: user.id });
    return user;
  }

  private validateUserInput(input: CreateUserInput): void {
    if (!input.email.includes('@')) {
      throw new ValidationError('Invalid email format');
    }
  }
}

// Abstract classes for base functionality
abstract class BaseRepository<T> implements Repository<T> {
  constructor(protected readonly model: Model<T>) {}

  async findById(id: string): Promise<T | null> {
    return this.model.findById(id);
  }

  async findAll(): Promise<T[]> {
    return this.model.find();
  }

  abstract create(data: Partial<T>): Promise<T>;
  abstract update(id: string, data: Partial<T>): Promise<T>;
  abstract delete(id: string): Promise<void>;
}
```

### Error Handling
```typescript
// Custom error classes
export class ApplicationError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number = 500
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends ApplicationError {
  constructor(message: string, public readonly field?: string) {
    super(message, 'VALIDATION_ERROR', 400);
  }
}

export class NotFoundError extends ApplicationError {
  constructor(message: string) {
    super(message, 'NOT_FOUND', 404);
  }
}

export class UnauthorizedError extends ApplicationError {
  constructor(message: string = 'Unauthorized') {
    super(message, 'UNAUTHORIZED', 401);
  }
}

// Error handling middleware
import { Request, Response, NextFunction } from 'express';

export function errorHandler(
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
): void {
  if (error instanceof ApplicationError) {
    res.status(error.statusCode).json({
      success: false,
      error: {
        code: error.code,
        message: error.message,
        ...(error instanceof ValidationError && { field: error.field })
      }
    });
    return;
  }

  // Log unexpected errors
  logger.error('Unexpected error', { error, path: req.path });

  res.status(500).json({
    success: false,
    error: {
      code: 'INTERNAL_ERROR',
      message: 'An unexpected error occurred'
    }
  });
}

// Try-catch with proper typing
async function processData(input: string): Promise<Result<ProcessedData>> {
  try {
    const data = await fetchData(input);
    const processed = transform(data);
    return { success: true, data: processed };
  } catch (error) {
    if (error instanceof ValidationError) {
      return { success: false, error };
    }
    throw error; // Re-throw unexpected errors
  }
}
```

### Validation with Zod
```typescript
import { z } from 'zod';

// Define schemas
const userSchema = z.object({
  username: z.string().min(3).max(50),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'user', 'guest']).default('user')
});

const createUserSchema = z.object({
  body: userSchema
});

// Infer TypeScript types from schemas
type User = z.infer<typeof userSchema>;
type CreateUserRequest = z.infer<typeof createUserSchema>;

// Validation middleware
import { Request, Response, NextFunction } from 'express';
import { AnyZodObject } from 'zod';

export function validate(schema: AnyZodObject) {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params
      });
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        res.status(400).json({
          success: false,
          error: {
            code: 'VALIDATION_ERROR',
            message: 'Invalid input',
            details: error.errors
          }
        });
        return;
      }
      next(error);
    }
  };
}

// Usage in routes
router.post('/users', validate(createUserSchema), createUserController);
```

### Dependency Injection
```typescript
// Define interfaces
interface IUserRepository {
  findById(id: string): Promise<User | null>;
  create(user: CreateUserInput): Promise<User>;
}

interface IUserService {
  getUser(id: string): Promise<User>;
  createUser(input: CreateUserInput): Promise<User>;
}

// Implementation
class UserService implements IUserService {
  constructor(
    private readonly repository: IUserRepository,
    private readonly logger: ILogger,
    private readonly emailService: IEmailService
  ) {}

  async createUser(input: CreateUserInput): Promise<User> {
    const user = await this.repository.create(input);
    await this.emailService.sendWelcomeEmail(user.email);
    this.logger.info('User created', { userId: user.id });
    return user;
  }
}

// Simple DI container
class Container {
  private services = new Map<string, any>();

  register<T>(name: string, factory: () => T): void {
    this.services.set(name, factory);
  }

  resolve<T>(name: string): T {
    const factory = this.services.get(name);
    if (!factory) {
      throw new Error(`Service ${name} not registered`);
    }
    return factory();
  }
}

// Usage
const container = new Container();

container.register('userRepository', () => new UserRepository(db));
container.register('userService', () => new UserService(
  container.resolve('userRepository'),
  logger,
  emailService
));
```

### Testing with Vitest
```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { UserService } from './user.service';
import type { IUserRepository } from './user.repository';

describe('UserService', () => {
  let userService: UserService;
  let mockRepository: IUserRepository;

  beforeEach(() => {
    // Create mock repository
    mockRepository = {
      findById: vi.fn(),
      create: vi.fn(),
      update: vi.fn(),
      delete: vi.fn()
    };

    userService = new UserService(mockRepository, logger);
  });

  describe('getUserById', () => {
    it('should return user when found', async () => {
      // Arrange
      const mockUser = {
        id: '1',
        username: 'testuser',
        email: 'test@example.com',
        createdAt: new Date()
      };
      vi.mocked(mockRepository.findById).mockResolvedValue(mockUser);

      // Act
      const result = await userService.getUserById('1');

      // Assert
      expect(result).toEqual(mockUser);
      expect(mockRepository.findById).toHaveBeenCalledWith('1');
    });

    it('should throw NotFoundError when user not found', async () => {
      // Arrange
      vi.mocked(mockRepository.findById).mockResolvedValue(null);

      // Act & Assert
      await expect(userService.getUserById('999')).rejects.toThrow(NotFoundError);
    });
  });

  describe('createUser', () => {
    it('should create user with valid input', async () => {
      // Arrange
      const input: CreateUserInput = {
        username: 'newuser',
        email: 'new@example.com'
      };
      const createdUser = { id: '1', ...input, createdAt: new Date() };
      vi.mocked(mockRepository.create).mockResolvedValue(createdUser);

      // Act
      const result = await userService.createUser(input);

      // Assert
      expect(result).toEqual(createdUser);
      expect(mockRepository.create).toHaveBeenCalledWith(
        expect.objectContaining(input)
      );
    });
  });
});
```

### Configuration Management
```typescript
import { z } from 'zod';

// Define configuration schema
const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
  REDIS_URL: z.string().url().optional()
});

// Parse and validate environment variables
export const config = envSchema.parse(process.env);

// Type-safe access
export type Config = z.infer<typeof envSchema>;

// Alternative: class-based config
export class AppConfig {
  readonly port: number;
  readonly databaseUrl: string;
  readonly jwtSecret: string;
  readonly environment: 'development' | 'production' | 'test';

  constructor() {
    this.port = parseInt(process.env.PORT || '3000');
    this.databaseUrl = this.required('DATABASE_URL');
    this.jwtSecret = this.required('JWT_SECRET');
    this.environment = (process.env.NODE_ENV as any) || 'development';
  }

  private required(key: string): string {
    const value = process.env[key];
    if (!value) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
    return value;
  }
}
```

## Build Configuration

### tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

### package.json scripts
```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "lint": "eslint src --ext .ts",
    "lint:fix": "eslint src --ext .ts --fix",
    "format": "prettier --write \"src/**/*.ts\"",
    "type-check": "tsc --noEmit",
    "clean": "rm -rf dist"
  }
}
```

## Best Practices Summary

1. **Enable TypeScript strict mode** - Catch more errors at compile time
2. **Avoid 'any' type** - Use 'unknown' if type is truly unknown
3. **Use interfaces for public APIs** - Better for extending
4. **Implement error handling** - Custom error classes hierarchy
5. **Validate inputs** - Use Zod or similar validation library
6. **Write tests** - Aim for 80%+ coverage
7. **Use dependency injection** - Improves testability
8. **Follow naming conventions** - Consistent naming across codebase
9. **Document complex logic** - TSDoc comments for public APIs
10. **Use async/await** - Avoid callback hell
```

## Example Prompt

```
Create a TypeScript Express API for a blog platform with:

Features:
- User authentication (JWT)
- CRUD operations for posts and comments
- Post search and filtering
- File upload for images
- Rate limiting
- PostgreSQL with TypeORM
- Input validation with Zod
- Comprehensive error handling
- Unit and integration tests

Technical requirements:
- TypeScript 5.0+ with strict mode
- Express.js
- TypeORM with PostgreSQL
- Redis for caching
- Zod for validation
- Vitest for testing
- Winston for logging
- Docker for containerization

Include:
- Layered architecture (controllers, services, repositories)
- Dependency injection
- API documentation with Swagger
- Type-safe configuration
- Proper error handling
- 80%+ test coverage
```

## References
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [Zod Documentation](https://zod.dev/)
