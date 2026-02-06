# Code Conventions and Style Guide Template

## Purpose
This template provides a comprehensive framework for establishing team-wide code conventions, style guidelines, and best practices.

## Use Case
Use when setting up a new project or standardizing practices across an existing codebase. Adapt sections based on your technology stack and team needs.

## Template

```
# Code Conventions and Style Guide

## Overview
This document defines the coding standards and conventions for [Project Name]. Following these guidelines ensures consistency, maintainability, and quality across the codebase.

**Last Updated**: [Date]
**Applies To**: [Languages/Frameworks covered]

## General Principles

### Core Values
1. **Readability First**: Code is read more often than it's written
2. **Consistency**: Follow established patterns
3. **Simplicity**: Prefer simple solutions over clever ones
4. **Explicit Over Implicit**: Make intentions clear
5. **DRY (Don't Repeat Yourself)**: Avoid duplication
6. **YAGNI (You Aren't Gonna Need It)**: Don't over-engineer
7. **SOLID Principles**: Apply object-oriented design principles

### Code Review Standards
- All code must be reviewed by at least one team member
- Reviews should focus on logic, design, and standards compliance
- Use the project's PR template for all submissions
- Address all review comments before merging

## Language-Specific Conventions

### [Language Name - e.g., TypeScript/JavaScript]

#### File Organization
```typescript
// 1. Imports (external first, then internal)
import React from 'react';
import { useState } from 'react';

import { Button } from '@/components/Button';
import { formatDate } from '@/utils/date';

// 2. Type definitions
interface UserProps {
  id: string;
  name: string;
}

// 3. Constants
const MAX_RETRIES = 3;

// 4. Component/Class/Function definitions
export function UserProfile({ id, name }: UserProps) {
  // implementation
}

// 5. Helper functions (if any)
```

#### Naming Conventions
```typescript
// PascalCase for classes, types, interfaces, and React components
class UserService {}
interface UserData {}
type UserId = string;
function UserProfile() {}

// camelCase for variables, functions, and methods
const userName = 'John';
function getUserById(id: string) {}

// UPPER_SNAKE_CASE for constants
const API_BASE_URL = 'https://api.example.com';
const MAX_RETRY_ATTEMPTS = 3;

// kebab-case for file names
user-profile.tsx
user-service.ts

// Prefix for private properties (convention)
class User {
  private _internalState: string;
}

// Boolean prefixes: is, has, should, can
const isActive = true;
const hasPermission = false;
const shouldRender = true;
const canEdit = false;
```

#### Type Annotations
```typescript
// Always use explicit types for function parameters and return values
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}

// Use type inference for simple variables
const count = 5; // inferred as number
const message = 'hello'; // inferred as string

// Use interfaces for object shapes
interface User {
  id: string;
  name: string;
  email: string;
}

// Use type for unions and primitives
type Status = 'pending' | 'active' | 'inactive';
type UserId = string;

// Prefer unknown over any
function handleError(error: unknown) {
  if (error instanceof Error) {
    console.error(error.message);
  }
}
```

#### Functions
```typescript
// Prefer arrow functions for callbacks and methods
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2);

// Use regular functions for top-level exports and when you need 'this'
export function processData(data: string[]): ProcessedData {
  return data.map(transform);
}

// Keep functions small and focused (max 50 lines)
// Extract complex logic into separate functions

// Use descriptive parameter names
// Bad
function calc(a: number, b: number) {}

// Good
function calculateTotalPrice(basePrice: number, taxRate: number) {}

// Use object parameters for functions with many arguments
// Bad
function createUser(name: string, email: string, age: number, role: string) {}

// Good
interface CreateUserParams {
  name: string;
  email: string;
  age: number;
  role: string;
}
function createUser(params: CreateUserParams) {}
```

#### Comments
```typescript
// Use JSDoc for public APIs
/**
 * Calculates the total price including tax.
 * @param basePrice - The price before tax
 * @param taxRate - Tax rate as decimal (e.g., 0.15 for 15%)
 * @returns The total price including tax
 */
function calculateTotalPrice(basePrice: number, taxRate: number): number {
  return basePrice * (1 + taxRate);
}

// Use inline comments for complex logic only
// Don't comment obvious code
// Bad
// Increment counter by 1
counter++;

// Good (complex logic)
// Apply exponential backoff: 2^attempt * 1000ms
const delay = Math.pow(2, attempt) * 1000;

// Use TODO comments with ticket numbers
// TODO(JIRA-123): Implement error handling
// FIXME(JIRA-456): Memory leak in event handler
```

#### Error Handling
```typescript
// Use custom error classes
class ValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ValidationError';
  }
}

// Always catch and handle errors
try {
  await riskyOperation();
} catch (error) {
  if (error instanceof ValidationError) {
    // Handle validation error
  } else {
    // Handle unexpected error
    logger.error('Unexpected error', error);
    throw error;
  }
}

// Use Result types for expected errors (optional pattern)
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

function parseJson(text: string): Result<unknown> {
  try {
    return { success: true, data: JSON.parse(text) };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}
```

## Code Organization

### Project Structure
```
src/
├── components/       # Reusable UI components
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   ├── Button.styles.ts
│   │   └── index.ts
│   └── index.ts     # Barrel export
├── features/        # Feature-based modules
│   └── user/
│       ├── components/
│       ├── hooks/
│       ├── services/
│       └── types.ts
├── hooks/           # Shared custom hooks
├── services/        # API and external services
├── utils/           # Utility functions
├── types/           # Shared type definitions
├── constants/       # Application constants
└── config/          # Configuration files
```

### Module Exports
```typescript
// Use named exports
export function Button() {}
export { TextField } from './TextField';

// Avoid default exports except for pages/routes
// Exception: Next.js pages, React lazy components
export default function HomePage() {}

// Use barrel exports (index.ts) for cleaner imports
// components/index.ts
export { Button } from './Button';
export { TextField } from './TextField';

// Usage
import { Button, TextField } from '@/components';
```

## Testing Standards

### Test Structure
```typescript
import { describe, it, expect, beforeEach } from 'vitest';

describe('UserService', () => {
  describe('getUserById', () => {
    it('should return user when found', async () => {
      // Arrange
      const userId = '123';
      const expectedUser = { id: userId, name: 'John' };
      
      // Act
      const result = await userService.getUserById(userId);
      
      // Assert
      expect(result).toEqual(expectedUser);
    });

    it('should throw error when user not found', async () => {
      // Arrange & Act & Assert
      await expect(
        userService.getUserById('invalid')
      ).rejects.toThrow('User not found');
    });
  });
});
```

### Test Coverage
- Aim for 80%+ code coverage
- 100% coverage for critical business logic
- Focus on testing behavior, not implementation
- Write tests for edge cases and error scenarios

### Test Naming
```typescript
// Pattern: should [expected behavior] when [condition]
it('should return empty array when no users exist', () => {});
it('should throw ValidationError when email is invalid', () => {});
it('should cache results after first call', () => {});
```

## Git Conventions

### Branch Naming
```
feature/user-authentication
bugfix/login-timeout
hotfix/security-patch
refactor/user-service
docs/api-documentation
```

### Commit Messages
Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add user authentication
fix: resolve login timeout issue
docs: update API documentation
refactor: simplify user service logic
test: add unit tests for user service
chore: update dependencies
perf: optimize database queries
style: format code with prettier
```

Format:
```
<type>(<scope>): <subject>

<body>

<footer>
```

Example:
```
feat(auth): implement JWT token refresh

- Add refresh token endpoint
- Update token middleware
- Add tests for token refresh

Closes #123
```

### Pull Request Guidelines
1. Use the PR template
2. Link related issues
3. Keep PRs small (< 400 lines when possible)
4. Provide context in description
5. Ensure CI passes
6. Request review from relevant team members
7. Address all comments before merging

## Documentation Standards

### Code Documentation
```typescript
/**
 * User authentication service.
 * Handles login, logout, and token management.
 */
export class AuthService {
  /**
   * Authenticates a user with email and password.
   * @param email - User's email address
   * @param password - User's password
   * @returns Authentication token and user data
   * @throws {ValidationError} If credentials are invalid
   * @throws {NetworkError} If authentication service is unavailable
   * @example
   * ```typescript
   * const result = await authService.login('user@example.com', 'password123');
   * console.log(result.token);
   * ```
   */
  async login(email: string, password: string): Promise<AuthResult> {
    // implementation
  }
}
```

### README Standards
Every feature module should have a README:
```markdown
# Feature Name

## Purpose
Brief description of what this feature does.

## Usage
Code examples showing how to use this feature.

## API Reference
List of exported functions/components with descriptions.

## Testing
How to run tests for this feature.

## Dependencies
External dependencies and why they're needed.
```

## Performance Guidelines

1. **Avoid Premature Optimization**: Profile before optimizing
2. **Lazy Loading**: Load code only when needed
3. **Memoization**: Use React.memo, useMemo, useCallback appropriately
4. **Bundle Size**: Monitor bundle size, code-split large dependencies
5. **Database Queries**: Optimize N+1 queries, use proper indexes
6. **Caching**: Implement caching for expensive operations

## Security Guidelines

1. **Input Validation**: Validate all user inputs
2. **SQL Injection**: Use parameterized queries
3. **XSS Prevention**: Sanitize user-generated content
4. **Authentication**: Use secure token storage
5. **HTTPS**: Always use HTTPS in production
6. **Dependencies**: Keep dependencies updated
7. **Secrets**: Never commit secrets, use environment variables
8. **CORS**: Configure CORS properly

## Accessibility Guidelines

1. **Semantic HTML**: Use proper HTML elements
2. **ARIA Labels**: Add labels for screen readers
3. **Keyboard Navigation**: Ensure all functionality is keyboard accessible
4. **Color Contrast**: Follow WCAG 2.1 AA standards
5. **Focus Management**: Implement proper focus indicators

## Tools and Enforcement

### Linters
- ESLint with TypeScript plugin
- Prettier for code formatting
- Stylelint for CSS (if applicable)

### Pre-commit Hooks
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

### CI/CD Checks
- Linting
- Type checking
- Unit tests (80%+ coverage)
- Integration tests
- Security scanning
- Bundle size check

## Exceptions

When you need to deviate from these conventions:
1. Document the reason with a comment
2. Discuss with the team
3. Update this guide if it becomes a pattern

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any
// Reason: Third-party library types are incorrect
const result: any = externalLibrary.process();
```

## Resources

- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
- [Clean Code by Robert C. Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)

## Revision History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2024-01-01 | Initial version | Team Lead |
| 1.1 | 2024-02-01 | Added testing standards | Team Lead |
```

## Example Prompt

```
Create a code conventions document for a Python/FastAPI backend project. Include:
- PEP 8 compliance guidelines
- Type hinting standards
- Module organization
- Testing conventions (pytest)
- Documentation standards (docstrings)
- Error handling patterns
- API endpoint naming
- Database query best practices
- Security guidelines specific to FastAPI

Make it comprehensive but practical for a team of 5 developers.
```

## References
- [PEP 8 - Python Style Guide](https://pep8.org/)
- [Google Style Guides](https://google.github.io/styleguide/)
- [Airbnb Style Guides](https://github.com/airbnb/javascript)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Clean Code Principles](https://clean-code-developer.com/)
