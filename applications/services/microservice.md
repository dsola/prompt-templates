# Microservice Template

## Purpose
This template guides the creation of production-ready microservices with proper service boundaries, communication patterns, and observability.

## Use Case
Use when building distributed systems with independent, scalable services that communicate via APIs or message queues.

## Template

```
Create a microservice with the following specifications:

## Service Overview
[Describe the service's bounded context, responsibilities, and business domain]

## Technical Requirements
- Language/Framework: [Node.js/NestJS / Python/FastAPI / Java/Spring Boot / Go / .NET]
- Communication: [REST / gRPC / Message Queue]
- Message Broker: [RabbitMQ / Apache Kafka / AWS SQS / Redis Streams]
- Database: [PostgreSQL / MongoDB / MySQL] (per-service database)
- Service Discovery: [Consul / Eureka / Kubernetes DNS]
- API Gateway: [Kong / Ambassador / Traefik / AWS API Gateway]

## Architecture Principles
1. Follow Single Responsibility Principle - one service, one domain
2. Database per service pattern
3. API-first design with contracts
4. Asynchronous communication for inter-service calls when possible
5. Implement Circuit Breaker pattern
6. Use saga pattern for distributed transactions
7. Event-driven architecture where appropriate

## Project Structure
```
service-name/
├── src/
│   ├── api/              # API layer (REST/gRPC handlers)
│   ├── application/      # Application services (use cases)
│   ├── domain/           # Domain models and business logic
│   ├── infrastructure/   # External services, database, message queue
│   │   ├── database/
│   │   ├── messaging/
│   │   └── clients/      # External API clients
│   ├── config/           # Configuration
│   └── main.ts
├── tests/
├── docker/
├── k8s/                  # Kubernetes manifests
└── docs/
```

## Key Features to Implement

### 1. Service Communication

#### Synchronous (REST/gRPC)
```typescript
// With circuit breaker
class UserServiceClient {
  private circuitBreaker: CircuitBreaker;
  
  async getUser(id: string): Promise<User> {
    return this.circuitBreaker.execute(async () => {
      const response = await fetch(`${this.baseUrl}/users/${id}`);
      return response.json();
    });
  }
}
```

#### Asynchronous (Message Queue)
```typescript
// Event publisher
class OrderService {
  async createOrder(order: Order) {
    await this.repository.save(order);
    
    // Publish event
    await this.eventBus.publish('order.created', {
      orderId: order.id,
      customerId: order.customerId,
      total: order.total
    });
  }
}

// Event consumer
class InventoryService {
  @EventHandler('order.created')
  async handleOrderCreated(event: OrderCreatedEvent) {
    await this.reserveInventory(event.orderId);
  }
}
```

### 2. Health Checks & Readiness Probes
```typescript
// Health endpoint
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    service: 'order-service',
    version: process.env.VERSION
  });
});

// Readiness endpoint
app.get('/ready', async (req, res) => {
  const dbOk = await checkDatabase();
  const queueOk = await checkMessageQueue();
  
  if (dbOk && queueOk) {
    res.status(200).json({ ready: true });
  } else {
    res.status(503).json({ ready: false });
  }
});
```

### 3. Distributed Tracing
```typescript
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('order-service');

async function processOrder(orderId: string) {
  const span = tracer.startSpan('processOrder');
  try {
    // Business logic
    span.addEvent('order.validated');
    await validateOrder(orderId);
    
    span.addEvent('payment.processing');
    await processPayment(orderId);
  } catch (error) {
    span.recordException(error);
    throw error;
  } finally {
    span.end();
  }
}
```

### 4. Service Discovery
```typescript
// Register service
async function registerService() {
  await consul.agent.service.register({
    name: 'order-service',
    address: process.env.SERVICE_HOST,
    port: parseInt(process.env.SERVICE_PORT),
    check: {
      http: `http://${process.env.SERVICE_HOST}:${process.env.SERVICE_PORT}/health`,
      interval: '10s'
    }
  });
}

// Discover service
async function discoverService(serviceName: string) {
  const services = await consul.agent.service.list();
  return services[serviceName];
}
```

### 5. Configuration Management
```typescript
// Environment-based configuration
export const config = {
  service: {
    name: process.env.SERVICE_NAME,
    port: parseInt(process.env.PORT || '3000'),
    environment: process.env.NODE_ENV
  },
  database: {
    host: process.env.DB_HOST,
    port: parseInt(process.env.DB_PORT || '5432'),
    name: process.env.DB_NAME
  },
  messageQueue: {
    url: process.env.QUEUE_URL,
    exchange: process.env.QUEUE_EXCHANGE
  },
  external: {
    userServiceUrl: process.env.USER_SERVICE_URL,
    paymentServiceUrl: process.env.PAYMENT_SERVICE_URL
  }
};
```

## Communication Patterns

### 1. Synchronous Request-Response
- Use for: Real-time queries, immediate responses needed
- Pattern: REST API or gRPC
- Consider: Timeout handling, circuit breakers, retries

### 2. Asynchronous Event-Driven
- Use for: Business events, eventual consistency
- Pattern: Publish-Subscribe with message broker
- Consider: Idempotency, ordering, dead letter queues

### 3. Saga Pattern (Distributed Transactions)
```typescript
// Orchestration-based saga
class OrderSaga {
  async execute(orderData: CreateOrderInput) {
    const order = await this.createOrder(orderData);
    
    try {
      await this.reserveInventory(order.id);
      await this.processPayment(order.id);
      await this.confirmOrder(order.id);
    } catch (error) {
      // Compensating transactions
      await this.cancelPayment(order.id);
      await this.releaseInventory(order.id);
      await this.cancelOrder(order.id);
      throw error;
    }
  }
}
```

## Resilience Patterns

### Circuit Breaker
```typescript
const circuitBreaker = new CircuitBreaker(externalApiCall, {
  timeout: 3000,        // Request timeout
  errorThresholdPercentage: 50,
  resetTimeout: 30000   // Try again after 30s
});
```

### Retry with Exponential Backoff
```typescript
async function retryWithBackoff(fn: Function, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await delay(Math.pow(2, i) * 1000); // Exponential backoff
    }
  }
}
```

## Best Practices
1. Each service owns its data - no shared databases
2. Use API contracts (OpenAPI, Protobuf)
3. Implement correlation IDs for request tracing
4. Use semantic versioning for APIs
5. Implement graceful shutdown
6. Log structured JSON logs
7. Externalize configuration
8. Implement health checks
9. Use containerization (Docker)
10. Implement proper monitoring and alerting

## Observability (Three Pillars)

### Logging
- Structured JSON logging
- Correlation IDs across services
- Log levels (DEBUG, INFO, WARN, ERROR)
- Centralized log aggregation (ELK, Loki)

### Metrics
- Request rate, duration, error rate
- Database connection pool metrics
- Queue depth and lag
- Custom business metrics
- Use Prometheus/Grafana

### Tracing
- Distributed tracing with OpenTelemetry
- Span context propagation
- Jaeger or Zipkin for visualization

## Testing Strategy
- Unit tests (70%+ coverage)
- Integration tests with test containers
- Contract tests between services (Pact)
- End-to-end tests for critical flows
- Chaos engineering for resilience testing

## Deployment & Scaling
- Containerize with Docker
- Deploy on Kubernetes
- Horizontal pod autoscaling
- Blue-green or canary deployments
- Service mesh (Istio, Linkerd) for advanced scenarios
```

## Example Prompt

```
Create an Order Management microservice with the following:

Domain: E-commerce order processing
Responsibilities:
- Create and manage orders
- Validate order data
- Communicate with Inventory service (async)
- Communicate with Payment service (sync)
- Implement saga pattern for order fulfillment

Technical:
- Node.js with NestJS and TypeScript
- PostgreSQL for order storage
- RabbitMQ for event publishing
- REST API for external communication
- gRPC for internal service communication
- Circuit breaker for external calls
- Distributed tracing with OpenTelemetry
- Health checks and metrics

Include proper error handling, retry logic, compensation transactions, and comprehensive logging.
```

## References
- [Microservices Patterns (Chris Richardson)](https://microservices.io/patterns/)
- [Building Microservices (Sam Newman)](https://samnewman.io/books/building_microservices/)
- [12-Factor App](https://12factor.net/)
- [OpenTelemetry](https://opentelemetry.io/)
- [Circuit Breaker Pattern](https://martinfowler.com/bliki/CircuitBreaker.html)
