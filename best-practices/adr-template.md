# Architecture Decision Record (ADR) Template

## Purpose
This template guides the creation of Architecture Decision Records to document important architectural decisions made during project development.

## Use Case
Use when making significant architectural decisions that will impact the project's structure, technology choices, or development practices. ADRs provide historical context and rationale for future team members.

## Template

```
# ADR-[NUMBER]: [Title - Short phrase describing the decision]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-XXX]

## Date
[YYYY-MM-DD]

## Context
[Describe the situation, problem, or opportunity that requires a decision]

What is the issue that we're addressing?
- What forces are at play? (technical, political, social, project constraints)
- What are the business requirements?
- What are the technical constraints?
- What architectural concerns exist?

## Decision
[State the decision that has been made]

We will [chosen solution].

### Rationale
[Explain why this decision was made over alternatives]

We chose this approach because:
1. [Reason 1]
2. [Reason 2]
3. [Reason 3]

## Alternatives Considered

### Alternative 1: [Name]
**Description**: [Brief description]
**Pros**:
- [Pro 1]
- [Pro 2]

**Cons**:
- [Con 1]
- [Con 2]

**Why rejected**: [Explanation]

### Alternative 2: [Name]
[Same structure as Alternative 1]

## Consequences

### Positive
- [Benefit 1]
- [Benefit 2]
- [Benefit 3]

### Negative
- [Trade-off 1]
- [Trade-off 2]

### Neutral
- [Impact 1]
- [Impact 2]

## Implementation Notes
[Technical details, migration path, or implementation guidelines if needed]

- Migration strategy: [If applicable]
- Timeline: [If applicable]
- Dependencies: [Any related ADRs or external dependencies]
- Risks: [Known risks and mitigation strategies]

## References
- [Link to related documentation]
- [Link to spike results or prototypes]
- [Link to related ADRs]
- [Link to external resources]

## Notes
[Additional context, discussion points, or future considerations]
```

## Real-World Examples

### Example 1: Database Choice

```
# ADR-001: Selection of PostgreSQL as Primary Database

## Status
Accepted

## Date
2024-01-15

## Context
We need to select a database for our e-commerce application that will handle:
- Product catalog (~100K products)
- User accounts and profiles
- Order history and transactions
- Real-time inventory management
- Complex queries with joins across multiple tables
- ACID compliance for financial transactions

The application is expected to scale to 10K concurrent users within 2 years.

## Decision
We will use PostgreSQL as our primary relational database.

### Rationale
1. **Strong ACID compliance**: Critical for handling financial transactions
2. **Rich feature set**: Advanced querying, full-text search, JSON support
3. **Proven scalability**: Successfully handles our projected load
4. **Team expertise**: Team has 5+ years of PostgreSQL experience
5. **Open source**: No licensing costs, strong community support
6. **Extension ecosystem**: PostGIS for future location features, pgvector for potential AI features

## Alternatives Considered

### Alternative 1: MySQL
**Description**: Popular open-source relational database
**Pros**:
- Widespread adoption
- Good documentation
- Slightly better read performance for simple queries

**Cons**:
- Less feature-rich than PostgreSQL
- Weaker support for complex queries
- Less robust JSON handling

**Why rejected**: PostgreSQL's advanced features and JSON support better align with our requirements.

### Alternative 2: MongoDB
**Description**: Document-oriented NoSQL database
**Pros**:
- Flexible schema
- Good horizontal scaling
- JSON-native storage

**Cons**:
- Lack of ACID transactions across documents (at the time)
- Not ideal for complex relational queries
- Team has limited NoSQL experience

**Why rejected**: Our data is highly relational, and ACID compliance is critical for financial transactions.

## Consequences

### Positive
- Reliable transaction handling for payments
- Rich querying capabilities for reporting
- JSON support allows flexibility where needed
- Strong community and tooling support
- Can use pgAdmin for database management

### Negative
- PostgreSQL can be more complex to tune than MySQL
- Requires more memory than some alternatives
- Learning curve for advanced features

### Neutral
- Need to establish backup and recovery procedures
- Will use managed PostgreSQL service (AWS RDS) to reduce operational overhead

## Implementation Notes
- **Migration strategy**: Start with development environment, then staging, then production
- **Timeline**: Development setup in Sprint 1, production migration in Sprint 3
- **Dependencies**: Need to provision AWS RDS instance
- **Risks**: 
  - Performance tuning required as data grows
  - Mitigation: Monitor query performance, add indexes as needed, implement connection pooling

## References
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [AWS RDS PostgreSQL Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html)
- Related ADRs: ADR-002 (Database Connection Pooling)

## Notes
- Re-evaluate decision at 50K users or 1M products
- Consider read replicas when read traffic exceeds 70% of primary capacity
```

### Example 2: API Design Pattern

```
# ADR-005: Adoption of GraphQL for Client-Facing API

## Status
Accepted

## Date
2024-02-20

## Context
Our mobile and web clients currently make multiple REST API calls to render complex views, leading to:
- Over-fetching: Receiving unnecessary data
- Under-fetching: Multiple round trips to get related data
- API versioning challenges
- Mobile app performance issues on slow networks

We need a more efficient API pattern that allows clients to request exactly what they need.

## Decision
We will implement GraphQL as our client-facing API layer, while maintaining REST APIs for third-party integrations.

### Rationale
1. **Reduced network overhead**: Clients fetch exactly what they need in one request
2. **Strong typing**: Schema provides clear contract and enables code generation
3. **Better developer experience**: GraphQL Playground for API exploration
4. **Backward compatibility**: Can add fields without versioning
5. **Mobile optimization**: Significantly reduces data transfer for mobile clients

## Alternatives Considered

### Alternative 1: REST with field selection
**Description**: Enhance REST API with field selection query parameters
**Pros**:
- Simpler to implement
- Team already familiar with REST

**Cons**:
- Still requires multiple endpoints for related data
- Complex to implement field selection for nested resources
- Doesn't solve the multiple round-trip problem

**Why rejected**: Doesn't fully address the performance issues.

### Alternative 2: Backend for Frontend (BFF) pattern
**Description**: Create specific backend services for each client type
**Pros**:
- Tailored responses for each client
- Can use REST internally

**Cons**:
- Multiple codebases to maintain
- Duplicated logic across BFFs
- Increased operational complexity

**Why rejected**: Higher maintenance overhead than GraphQL.

## Consequences

### Positive
- 60% reduction in mobile data transfer (based on prototype)
- Single endpoint for all data queries
- Improved mobile app performance
- Better frontend developer experience
- Self-documenting API schema

### Negative
- Learning curve for team (2-3 weeks)
- More complex server-side caching strategy needed
- Need to implement query complexity analysis
- Requires DataLoader to prevent N+1 queries

### Neutral
- REST APIs will remain for webhook integrations and third-party access
- Need to establish GraphQL best practices and conventions

## Implementation Notes
- **Technology**: Apollo Server with TypeScript
- **Migration strategy**: 
  - Phase 1: Implement GraphQL alongside existing REST (parallel)
  - Phase 2: Migrate web app to GraphQL (Sprint 10-12)
  - Phase 3: Migrate mobile apps to GraphQL (Sprint 13-15)
  - Phase 4: Deprecate redundant REST endpoints (Sprint 16)
- **Timeline**: 6 sprints for complete migration
- **Dependencies**: 
  - ADR-006: Selection of Apollo Server
  - ADR-007: GraphQL Schema Design Guidelines
- **Risks**:
  - Query complexity attacks → Mitigate with query depth limiting and cost analysis
  - Team learning curve → Mitigate with training sessions and pair programming

## References
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)
- [Apollo Server Documentation](https://www.apollographql.com/docs/apollo-server/)
- [Production Ready GraphQL](https://book.productionreadygraphql.com/)
- Prototype results: [Internal Confluence Link]

## Notes
- Monitor query performance and complexity in production
- Plan for GraphQL subscriptions in Q3 for real-time features
- Consider GraphQL Code Generator for type-safe client code
```

## Best Practices for Writing ADRs

1. **Keep it concise**: ADRs should be readable in 5-10 minutes
2. **Write when deciding**: Document decisions when they're made, not after
3. **One decision per ADR**: Don't combine multiple decisions
4. **Be specific**: Provide enough detail for future readers to understand the context
5. **Update status**: Mark as superseded when decisions change
6. **Link related ADRs**: Create a web of knowledge
7. **Include dates**: Track when decisions were made
8. **Version control**: Store ADRs in the repository with the code
9. **Review process**: Have ADRs reviewed by relevant stakeholders before acceptance
10. **Number sequentially**: Use ADR-001, ADR-002, etc.

## ADR Organization

```
docs/
└── adr/
    ├── README.md              # Index of all ADRs
    ├── template.md            # ADR template
    ├── 0001-record-architecture-decisions.md
    ├── 0002-database-selection.md
    ├── 0003-api-design-pattern.md
    └── 0004-deployment-strategy.md
```

## When to Write an ADR

Write an ADR when making decisions about:
- Technology selection (languages, frameworks, databases)
- Architectural patterns (microservices, event-driven, etc.)
- Development practices (testing strategy, CI/CD)
- Security approaches
- Data management strategies
- API design decisions
- Third-party service selections
- Performance optimization strategies

## When NOT to Write an ADR

Don't write ADRs for:
- Routine implementation details
- Temporary or easily reversible decisions
- Decisions that don't affect the architecture
- Team processes (use other documentation)

## Example Prompt

```
Create an ADR for choosing between monolithic and microservices architecture for a new SaaS application. The application needs to handle user management, payment processing, and content delivery. Consider team size (5 developers), time to market requirements (3 months MVP), and expected scale (100K users in year 1). Include at least 2 alternatives and document the trade-offs clearly.
```

## References
- [ADR GitHub Organization](https://adr.github.io/)
- [Michael Nygard's ADR Article](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [ADR Tools](https://github.com/npryce/adr-tools)
- [Thoughtworks Technology Radar - ADRs](https://www.thoughtworks.com/radar/techniques/lightweight-architecture-decision-records)
