# Enterprise TypeScript Project Architecture: Multi-Tenant SaaS Platform

**Platform:** NexusPlan (Project Management Platform)  
**Role:** Principal Software Architect  
**Document Status:** Approved  

---

## Business Requirements

1. **Multi-Tenancy**: The system must securely isolate data between different client organizations (tenants).
2. **SaaS Subscription Model**: Support tiered subscriptions (Free, Pro, Enterprise) dictating feature availability and usage limits.
3. **High Availability**: Target 99.99% uptime to meet enterprise SLA requirements.
4. **Global Reach**: Support localization and data residency options for European (GDPR) and US (CCPA) customers.
5. **Auditability**: Complete audit logs for all sensitive actions to comply with SOC2 requirements.

## Functional Requirements

1. **Workspace Management**: Tenants can create multiple workspaces and invite users.
2. **Project & Task Tracking**: Hierarchical management of projects, boards, lists, and tasks.
3. **Real-time Collaboration**: WebSocket-based real-time updates for task modifications and comments.
4. **Role-Based Access Control (RBAC)**: Custom roles (Admin, Manager, Member, Guest) at the tenant and workspace levels.
5. **Third-Party Integrations**: Webhooks and REST APIs to integrate with Slack, GitHub, and Jira.

## Non-Functional Requirements

1. **Performance**: API response times must be < 200ms at the 95th percentile.
2. **Scalability**: Support up to 10,000 concurrent users and 5,000 requests per second.
3. **Security**: AES-256 encryption at rest, TLS 1.3 in transit.
4. **Maintainability**: Strict linting, strongly typed contracts, minimum 80% test coverage.
5. **Observability**: Distributed tracing across all microservices and database queries.

## System Architecture

We utilize a **Modular Monolith** architecture pattern using Domain-Driven Design (DDD), designed to easily split into microservices as scaling demands.

```mermaid
graph TD
    Client[Web/Mobile Client] --> CDN[Cloudflare CDN / WAF]
    CDN --> LB[Load Balancer]
    LB --> API[Node.js/Fastify API Gateway]
    
    subgraph Modular Monolith [TypeScript Backend]
        API --> Auth[IAM Module]
        API --> Projects[Projects Module]
        API --> Billing[Billing Module]
    end
    
    Auth --> DB[(PostgreSQL Primary)]
    Projects --> DB
    Billing --> DB
    
    Projects --> Redis[(Redis Cache)]
    API --> Socket[WebSocket Server]
    Socket --> Redis
    
    DB --> Replica[(PostgreSQL Replica)]
    
    subgraph Observability
        Modular Monolith --> Prom[Prometheus]
        Modular Monolith --> Jaeger[Jaeger / OpenTelemetry]
    end
```

## Folder Structure

Our structure follows Clean Architecture principles, ensuring the domain logic is isolated from external frameworks.

```text
src/
├── app.ts                 # Application entry point
├── config/                # Environment variables and config loading
├── core/                  # Core domain models and interfaces
│   ├── exceptions/        # Custom error classes
│   └── types/             # Global TypeScript types
├── modules/               # Feature modules (Domain-Driven Design)
│   ├── iam/               # Identity & Access Management
│   │   ├── controllers/   # HTTP route handlers
│   │   ├── services/      # Business logic
│   │   ├── repositories/  # Database access layer
│   │   ├── dtos/          # Data Transfer Objects (Zod schemas)
│   │   └── tests/         # Unit and integration tests
│   ├── projects/          # Project & Task management module
│   └── billing/           # Subscription handling
├── infrastructure/        # External services and connections
│   ├── database/          # Prisma client and schema
│   ├── cache/             # Redis client
│   └── logger/            # Winston/Pino setup
├── shared/                # Code shared across modules
│   ├── middlewares/       # Express/Fastify middlewares (Auth, Logging)
│   └── utils/             # Helper functions
└── types/                 # Type declaration files (*.d.ts)
```

### Folder Breakdown:
* **`core/`**: Houses enterprise business rules, globally shared types, and base exceptions. It has zero dependencies on external libraries (like Express or PostgreSQL).
* **`modules/`**: Contains the discrete vertical slices of the application. Each module (like `iam` or `projects`) manages its own slice of logic independently.
* **`infrastructure/`**: Details the adapters that interact with the outside world (databases, third-party APIs, logging frameworks).
* **`shared/`**: Contains cross-cutting concerns like standard middlewares, string manipulation utils, etc.

## Technology Choices

| Layer | Technology | Justification |
|-------|------------|---------------|
| **Runtime** | Node.js (v20 LTS) | Excellent ecosystem, non-blocking I/O ideal for real-time SaaS. |
| **Language** | TypeScript (v5.x) | Type safety, advanced refactoring, prevents entire classes of runtime errors. |
| **Web Framework** | Fastify | 20% faster than Express, native JSON schema validation, lower overhead. |
| **Database** | PostgreSQL (v16) | ACID compliance, JSONB support for dynamic schemas, extremely reliable. |
| **ORM** | Prisma | Exceptional TS integration, type-safe queries, great developer experience. |
| **Validation** | Zod | TypeScript-first schema declaration and validation. |
| **Testing** | Jest & Supertest | Industry standard, great mocking capabilities. |
| **Caching/PubSub**| Redis | Fast in-memory store for session management and WebSocket pub/sub. |

## TypeScript Usage

Strict mode (`"strict": true`) is universally enabled. We heavily utilize TypeScript features to construct highly robust code.

### Types and Interfaces
```typescript
// Base Entity Interface
export interface BaseEntity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

// Tenant-specific Entity
export interface TenantEntity extends BaseEntity {
  tenantId: string;
}

// User Model
export interface User extends TenantEntity {
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
}
```

### Generics & Utility Types
```typescript
// Generic Paginated Response wrapper
export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    hasMore: boolean;
  };
}

// Utility Types for DTOs to reuse the base User model safely
export type CreateUserDTO = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;
export type UpdateUserDTO = Partial<CreateUserDTO>;
```

### Design Patterns (Repository Pattern & Dependency Injection)
```typescript
export interface IRepository<T> {
  findById(id: string, tenantId: string): Promise<T | null>;
  create(data: Omit<T, 'id'>): Promise<T>;
  update(id: string, tenantId: string, data: Partial<T>): Promise<T>;
  delete(id: string, tenantId: string): Promise<boolean>;
}

// Dependency Injection ensures testability
export class UserRepository implements IRepository<User> {
  constructor(private readonly prisma: PrismaClient) {}
  
  async findById(id: string, tenantId: string): Promise<User | null> {
    return this.prisma.user.findUnique({
      where: { id_tenantId: { id, tenantId } }
    });
  }
}
```

## API Design

### RESTful Principles
- **Resource-Oriented Navigation**: Hierarchical routes (`/api/v1/workspaces/:workspaceId/projects/:projectId`).
- **Versioning**: Explicit versioning in the URL mapping (`/api/v1/...`) to prevent breaking changes for API consumers.
- **Standardized Responses**: Using JSend format to ensure front-ends have a predictable structure.

### Validation with Zod
```typescript
export const CreateProjectSchema = z.object({
  name: z.string().min(3).max(100),
  description: z.string().optional(),
  status: z.enum(['PLANNING', 'ACTIVE', 'COMPLETED']),
  dueDate: z.string().datetime().optional()
});

export type CreateProjectInput = z.infer<typeof CreateProjectSchema>;
```

### Authentication & Authorization
- **Authentication**: JWT-based stateless authentication. Short-lived access tokens (15m) alongside HTTP-only, Secure refresh tokens (7d) to mitigate XSS and CSRF.
- **Authorization**: Middleware extracting `tenantId` and `role` from the verified JWT.

```typescript
export const requireRole = (allowedRoles: Role[]) => {
  return async (req: FastifyRequest, reply: FastifyReply) => {
    const userRole = req.user.role;
    if (!allowedRoles.includes(userRole)) {
      throw new ForbiddenError('Insufficient permissions');
    }
  };
};
```

## Database Design

### Multi-Tenancy Strategy
We leverage **Row-Level Tenancy (Shared Database, Shared Schema)**. Every client shares the database, but isolated access is enforced at the application layer through a mandated `tenantId` column on nearly all tables.

### Prisma Schema Example
```prisma
model Organization {
  id        String   @id @default(uuid())
  name      String
  users     User[]
  projects  Project[]
  createdAt DateTime @default(now())
}

model User {
  id             String       @id @default(uuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  email          String
  role           Role         @default(MEMBER)
  
  // Composite unique index prevents duplicate emails per tenant
  @@unique([email, organizationId])
  // Indexing foreign keys accelerates joins and specific tenant data lookups
  @@index([organizationId])
}

model Project {
  id             String       @id @default(uuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  name           String
  
  @@index([organizationId])
}

enum Role {
  ADMIN
  MANAGER
  MEMBER
  GUEST
}
```

### Database Commands
```bash
# Initialize Prisma in the project
npx prisma init

# Create and apply a migration for local development
npx prisma migrate dev --name init_schema

# Generate the TypeScript Prisma Client
npx prisma generate

# Apply migrations in a CI/CD or production environment
npx prisma migrate deploy

# Seed the database with initial roles/admin
npx prisma db seed
```

### Indexing & Transactions
- **Indexing**: All `organizationId` foreign keys are indexed. We utilize composite indexes (e.g., `[organizationId, status]`) for frequently filtered queries.
- **Transactions**: Complex logical operations spanning multiple tables (like organization onboarding) are strictly bound in Prisma Interactive Transactions to maintain database integrity.

## Security

1. **JWT & Session Management**: RSA-signed JWTs (RS256). Active session management via Redis blocklisting for early logout/revocation.
2. **RBAC**: Deep integration of Role-Based Access Control down to the database query scope to avoid Insecure Direct Object References (IDOR).
3. **OWASP Protections**:
   - Injection: Wholly eliminated via Prisma parameterized query constructs.
   - XSS: API exclusively outputs `application/json`; UI/Frontend handles strict escaping.
   - Rate Limiting: Redis-backed IP-level limits, plus strict tenant-level burst quotas to prevent noisy-neighbor impact.

## Testing

### 1. Unit Testing
Used for isolated business logic using **Jest**. External dependencies like Prisma are thoroughly mocked.
```typescript
describe('ProjectService', () => {
  it('should accurately compute project progression', () => {
    // Setup mock repository & data
    // Assert calculation logic natively
  });
});
```

### 2. Integration Testing
Verifies module boundaries against a real PostgreSQL instance. We spin up ephemeral DBs using **Testcontainers** for reproducible CI pipelines.

### 3. End-to-End (E2E) Testing
Uses **Supertest** to simulate real HTTP requests through the full server stack without bypassing any middleware.
```typescript
describe('POST /api/v1/projects', () => {
  it('should return 401 if request is unauthenticated', async () => {
    await request(app.server)
      .post('/api/v1/projects')
      .send({ name: 'Alpha Project' })
      .expect(401);
  });
});
```

## Monitoring

### Logging
Structured JSON logging configured via **Pino**. Every request features an attached `traceId`, `tenantId`, and `userId` implicitly forwarded throughout service lifecycles.

### Metrics & Tracing
- **Prometheus Metrics**: Monitoring Node.js garbage collection, event-loop lag, plus custom SaaS metrics (active organizations, feature flag invocation).
- **OpenTelemetry Tracing**: Used to map spans across database calls and cache hits allowing visual bottleneck isolation in Grafana/Jaeger.

## CI/CD

Implemented through robust **GitHub Actions** pipelines.

### GitHub Actions Workflow (`.github/workflows/ci.yml`)
```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: user
          POSTGRES_PASSWORD: password
          POSTGRES_DB: nexusplan_test
        ports:
          - 5432:5432
      redis:
        image: redis:7
        ports:
          - 6379:6379
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
          cache: 'npm'
          
      - name: Install Dependencies
        run: npm ci
        
      - name: Code Quality
        run: |
          npm run lint
          npm run typecheck
          
      - name: Database Setup
        env:
          DATABASE_URL: postgresql://user:password@localhost:5432/nexusplan_test
        run: |
          npx prisma generate
          npx prisma migrate deploy
          
      - name: Run Tests
        env:
          DATABASE_URL: postgresql://user:password@localhost:5432/nexusplan_test
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: testing-secret
        run: npm run test:coverage

  deploy:
    needs: build-and-test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build and Push Docker Image
        run: |
          docker build -t registry.example.com/api:${{ github.sha }} .
          # docker push registry.example.com/api:${{ github.sha }}
```

## Docker

Multi-stage `Dockerfile` aimed at optimizing for a secure, minimal production container footprint.

```dockerfile
# Stage 1: Build Environment
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
COPY prisma ./prisma/
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production Runtime
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
# Copy strictly what's necessary from builder
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma

USER node
EXPOSE 3000
CMD ["npm", "start"]
```

### Docker Compose Local Stack (`docker-compose.yml`)
```yaml
version: '3.8'
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/nexusplan
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
  postgres:
    image: postgres:16
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=nexusplan
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  pgdata:
```

### Docker Commands
```bash
# Build and run the local development stack
docker-compose up -d --build

# View container logs
docker-compose logs -f api

# Stop the stack
docker-compose down
```

## Kubernetes

Deployment is modeled via Helm charts, focusing heavily on resiliency.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexusplan-api
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: api
          image: registry.example.com/api:v1.2.0
          resources:
            requests:
              cpu: 200m
              memory: 256Mi
            limits:
              cpu: 500m
              memory: 512Mi
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
```

### Kubernetes Operation Commands
```bash
# Create the production namespace
kubectl create namespace nexusplan

# Apply ConfigMaps and Secrets
kubectl apply -f k8s/configmap.yaml -n nexusplan
kubectl apply -f k8s/secret.yaml -n nexusplan

# Apply the Deployment and Service
kubectl apply -f k8s/deployment.yaml -n nexusplan
kubectl apply -f k8s/service.yaml -n nexusplan

# Check deployment rollout status
kubectl rollout status deployment/nexusplan-api -n nexusplan

# Monitor pod scaling
kubectl get pods -w -n nexusplan

# Tail application logs
kubectl logs -f deployment/nexusplan-api -n nexusplan
```

## Deployment Strategy

We employ a **Zero-Downtime Rolling Update** mechanism within Kubernetes. 
Crucially, database migrations run via `PreSync` hooks or init-containers. We heavily adhere to the "Expand and Contract" database pattern—never making destructive DB alterations (like dropping columns) until the codebase has fully migrated past its reliance on them, ensuring concurrent old/new app versions function safely.

## Scaling Strategy

1. **Stateless App Layer**: Node.js APIs maintain absolutely no local state; scaling horizontally happens automatically via Kubernetes Horizontal Pod Autoscaler (HPA) targeting 70% CPU utilization thresholds.
2. **Database Read Replicas**: Write-heavy operations target the primary PostgreSQL node, while reporting or deep-fetch queries utilize Read Replicas.
3. **Caching**: Redis is utilized extensively to cache complex DB aggregations, rate-limiting states, and session data to radically reduce DB strain.

## Team Workflow

1. **Branching**: Trunk-based development. Feature branches are strictly short-lived.
2. **Commits**: `commitlint` enforces Conventional Commits (`feat:`, `fix:`, `refactor:`) to guarantee automated changelogs.
3. **Code Reviews**: Mandatory minimum of 2 approvals per PR. CI must be green, and tests must be included.
4. **Documentation**: OpenAPI (Swagger) specifications are autogenerated directly from Fastify and Zod validation schemas, guaranteeing the docs can never drift from the code implementation.

### Git & CLI Workflow
```bash
# 1. Update main and branch out
git checkout main
git pull origin main
git checkout -b feat/task-board

# 2. Make changes and run local checks
npm run lint
npm run test

# 3. Commit utilizing Conventional Commits
git add .
git commit -m "feat(projects): implement task board drag and drop"

# 4. Push branch and open Pull Request
git push origin feat/task-board
```

## Common Enterprise Challenges

1. **Multi-Tenant Data Bleed (Cross-Tenant Access)**: 
   *Solution*: Strict abstraction at the repository layer. The controller injects the contextual `tenantId` extracted from the verified JWT into every repository call. The application fundamentally rejects reading `tenantId` from request bodies.
2. **Extremely Slow TypeScript Compilation**: 
   *Solution*: By utilizing `tsup` (esbuild) we achieve sub-second builds, while isolating raw type-checking (`tsc --noEmit`) to a parallel, non-blocking CI pipeline stage.
3. **Long-Running Operations Blocking the Event Loop**:
   *Solution*: API endpoints default to paginated responses. Heavy data exports are offloaded asynchronously via BullMQ workers, subsequently pushing the completed status back to clients over WebSockets.

## Lessons Learned

1. **Prisma over Raw SQL for Predictability**: While raw SQL is demonstrably faster, Prisma's immense type safety saved our team from thousands of runtime errors over the project lifecycle. For the 1% of queries requiring maximum optimization, we drop down to raw SQL safely managed inside Prisma's typed interfaces.
2. **Start Modular, Extract Later**: Immediately jumping into Microservices injects massive, unnecessary operational overhead. The Modular Monolith pattern allowed high iteration speed with distinct architectural boundaries, letting us surgically extract services (e.g., Billing) into isolated services only when they verifiably required independent scaling constraints.
3. **Strict TypeScript From Day One**: Applying `strictNullChecks` and `noImplicitAny` retroactively to a large enterprise codebase is a nightmare. Enforcing absolute strictness initially marginally slows scaffolding but comprehensively eliminates "Cannot read properties of undefined" disasters in production.