# GraphQL API Template

## Purpose
This template provides comprehensive guidance for building GraphQL APIs with proper schema design, resolvers, and performance optimization.

## Use Case
Use when building modern APIs that benefit from flexible queries, strongly-typed schemas, and efficient data fetching patterns.

## Template

```
Create a GraphQL API with the following specifications:

## API Overview
[Describe the API's purpose, main types, and data relationships]

## Technical Stack
- Language/Framework: [Node.js/Apollo Server / Python/Strawberry / Java/GraphQL Java / Go/gqlgen]
- GraphQL Implementation: [Apollo Server / GraphQL Yoga / Hot Chocolate]
- Database: [PostgreSQL / MongoDB / MySQL]
- ORM/Query Builder: [Prisma / TypeORM / Sequelize / Mongoose]
- Authentication: JWT with context
- Schema Tools: [GraphQL Code Generator / Pothos / TypeGraphQL]

## Architecture Principles
1. Schema-first or code-first approach
2. Implement DataLoader for N+1 query prevention
3. Use proper type system and nullability
4. Implement field-level authorization
5. Follow GraphQL best practices
6. Optimize with query complexity analysis
7. Implement proper error handling

## Project Structure
```
src/
├── schema/              # GraphQL schema definitions
│   ├── types/          # Type definitions
│   ├── queries/        # Query definitions
│   ├── mutations/      # Mutation definitions
│   └── subscriptions/  # Subscription definitions (optional)
├── resolvers/          # Resolver implementations
├── dataloaders/        # DataLoader instances
├── services/           # Business logic
├── models/             # Database models
├── middleware/         # Custom middleware
├── utils/              # Helper functions
├── directives/         # Custom directives
└── server.ts           # Server setup
```

## Schema Design Best Practices

### Type Definitions
```graphql
# Use clear, descriptive names
type User {
  id: ID!
  email: String!
  username: String!
  profile: Profile
  posts(first: Int = 10, after: String): PostConnection!
  createdAt: DateTime!
  updatedAt: DateTime!
}

# Use connections for pagination
type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type PostEdge {
  node: Post!
  cursor: String!
}

# Clear input types
input CreateUserInput {
  email: String!
  username: String!
  password: String!
}

# Union types for flexible returns
union SearchResult = User | Post | Comment
```

### Queries
```graphql
type Query {
  # Get single resource
  user(id: ID!): User
  
  # List with pagination
  users(
    first: Int = 10
    after: String
    filter: UserFilterInput
    orderBy: UserOrderByInput
  ): UserConnection!
  
  # Search
  search(query: String!, type: [SearchType!]): [SearchResult!]!
  
  # Health check
  health: HealthStatus!
}
```

### Mutations
```graphql
type Mutation {
  # Use input types
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!
}

# Return payload types
type CreateUserPayload {
  user: User
  errors: [Error!]
}
```

## Key Features to Implement

### 1. Authentication & Authorization
```typescript
// Context with user
interface Context {
  user?: User;
  dataloaders: DataLoaders;
}

// Field-level authorization
const resolvers = {
  Query: {
    sensitiveData: authenticated((parent, args, context) => {
      // Only authenticated users
    })
  }
};
```

### 2. DataLoader for Performance
```typescript
const userLoader = new DataLoader(async (userIds) => {
  const users = await db.users.findMany({
    where: { id: { in: userIds } }
  });
  return userIds.map(id => users.find(u => u.id === id));
});
```

### 3. Error Handling
```typescript
// Custom error classes
class ValidationError extends ApolloError {
  constructor(message: string, validationErrors: any) {
    super(message, 'VALIDATION_ERROR', { validationErrors });
  }
}

// Formatted errors
formatError: (error) => {
  return {
    message: error.message,
    code: error.extensions?.code,
    path: error.path,
  };
}
```

### 4. Query Complexity & Depth Limiting
```typescript
import { createComplexityLimitRule } from 'graphql-validation-complexity';

const server = new ApolloServer({
  validationRules: [
    createComplexityLimitRule(1000),
    depthLimit(10)
  ]
});
```

### 5. Subscriptions (Real-time)
```graphql
type Subscription {
  postAdded: Post!
  messageReceived(chatId: ID!): Message!
}
```

## Performance Optimization
1. Use DataLoader to batch and cache database queries
2. Implement query complexity analysis
3. Add depth limiting to prevent deeply nested queries
4. Use persisted queries for production
5. Implement field-level caching
6. Monitor resolver performance
7. Use query whitelisting in production

## Security Best Practices
1. Implement authentication at context level
2. Add authorization directives or middleware
3. Validate all inputs
4. Rate limit queries
5. Disable introspection in production
6. Implement query cost analysis
7. Sanitize user inputs
8. Use HTTPS only

## Testing Strategy
- Unit tests for resolvers
- Integration tests for complete queries
- Schema validation tests
- Load testing for performance
- Security testing for authorization

## Monitoring & Observability
- Apollo Studio integration
- Query performance tracking
- Error rate monitoring
- Resolver timing
- Cache hit rates
```

## Example Prompt

```
Create a GraphQL API for a social media platform with the following features:

Types:
1. User (with profile, posts, followers)
2. Post (with author, comments, likes)
3. Comment (with author, replies)
4. Like (with user reference)

Implement:
- User authentication with JWT
- CRUD operations for posts
- Nested comments system
- Real-time subscriptions for new posts
- Pagination with cursor-based approach
- DataLoader for optimized queries
- Field-level authorization
- Query complexity limiting

Use Node.js with Apollo Server, TypeScript, Prisma with PostgreSQL, and implement comprehensive error handling and validation.
```

## References
- [GraphQL Official Documentation](https://graphql.org/)
- [Apollo Server Documentation](https://www.apollographql.com/docs/apollo-server/)
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)
- [DataLoader](https://github.com/graphql/dataloader)
- [GraphQL Code Generator](https://www.graphql-code-generator.com/)
