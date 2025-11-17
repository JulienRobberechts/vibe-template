# Workflow: Project Management

**Purpose:** Master spec management, iteration tracking, project structure conventions, and git workflows for all project work.

**Version:** 1.0
**Lines:** ~500

---

## When to Use This Skill

Load this skill when:
- Creating or refining specifications
- Planning implementation iterations
- Managing project documentation
- Committing code with proper git workflow
- Creating changesets for releases

---

## Core Principles

1. **Spec-First:** All work starts with clear specifications
2. **Incremental Delivery:** Each iteration produces working software
3. **Track Progress:** Status and completion criteria always visible
4. **Follow Conventions:** Consistent structure and naming
5. **Git Discipline:** Meaningful commits, proper changesets

---

## Section 1: Spec Management

### Business Specs (`/specs/business-specs.md`)

**Purpose:** High-level product requirements, user needs, business goals

**Template:**

```markdown
# Business Specifications: [Project Name]

## Vision
What problem does this solve? Who benefits?

## Goals
- Primary goal
- Secondary goals
- Success metrics

## User Stories

### User Type 1: [Role]
- As a [role], I want [feature] so that [benefit]
- As a [role], I want [feature] so that [benefit]

### User Type 2: [Role]
- As a [role], I want [feature] so that [benefit]

## Constraints
- Budget: $X or Y hours
- Timeline: Deadline or milestones
- Technical: Browser support, platforms
- Compliance: GDPR, HIPAA, etc.

## Out of Scope
What we explicitly won't build (to avoid scope creep)

## Open Questions
- Question 1?
- Question 2?
```

**When to create:**
- Beginning of new project
- Major feature initiatives
- Before technical planning

**Who owns:** Product manager, stakeholders, or team lead

---

### Tech Specs (`/specs/tech-specs.md`)

**Purpose:** Technical architecture, design decisions, constraints

**Template:**

```markdown
# Technical Specifications: [Project Name]

## Architecture
- Pattern: Hexagonal architecture
- Layers: Domain, Application, Infrastructure
- Port/Adapter separation

## Tech Stack
- Runtime: Node.js 20+
- Language: TypeScript 5+
- Framework: Express, React, etc.
- Database: PostgreSQL, MongoDB, etc.
- ORM: Prisma, Drizzle, etc.
- Testing: Vitest, Jest, etc.

## File Structure
```
src/
  domain/           # Pure business logic
  application/      # Use cases, ports
  infrastructure/   # Adapters (DB, API, CLI, UI)
```

## Design Decisions

### Decision 1: [Topic]
**Context:** Why this matters
**Options:** A, B, C
**Chosen:** B
**Rationale:** Why B is best

### Decision 2: [Topic]
**Context:** ...
**Options:** ...
**Chosen:** ...
**Rationale:** ...

## API Design
- REST endpoints
- GraphQL schema
- CLI commands
- Event bus topics

## Data Models
- Entity relationships
- Database schema
- Migration strategy

## Security
- Authentication: JWT, OAuth, etc.
- Authorization: RBAC, policies
- Data encryption
- API rate limiting

## Performance
- Response time targets
- Scalability requirements
- Caching strategy

## Deployment
- Hosting: AWS, Vercel, etc.
- CI/CD: GitHub Actions
- Environments: dev, staging, prod

## Open Questions
- Technical question 1?
- Technical question 2?
```

**When to create:**
- After business specs
- Before implementation planning
- When architecture needs documentation

**Who owns:** Tech lead, architect, or senior developer

---

### Feature Specs (`/backlog/*.md`)

**Purpose:** Detailed feature requirements ready for implementation

**Template:**

```markdown
# Feature: [Feature Name]

**Status:** PENDING | IN PROGRESS | COMPLETED
**Created:** YYYY-MM-DD
**Completed:** YYYY-MM-DD (when done)

## Objective
What this feature accomplishes (1-2 sentences)

## User Stories
- As a [user], I want [feature] so that [benefit]
- As a [user], I want [feature] so that [benefit]

## Requirements

### Functional
- Must: Critical requirements
- Should: Important but not blocking
- Could: Nice-to-have

### Non-Functional
- Performance: Response times, throughput
- Security: Auth, validation, sanitization
- Usability: Accessibility, mobile support

## Acceptance Criteria
- [ ] Criterion 1: Specific, testable condition
- [ ] Criterion 2: Specific, testable condition
- [ ] Criterion 3: Specific, testable condition

## Technical Implementation

### Approach
Chosen technical approach and rationale

### Files to Create/Modify
- `src/domain/entities/NewEntity.ts`
- `src/application/use-cases/NewUseCase.ts`
- `src/infrastructure/api/routes/newRoute.ts`

### Technical Hints
- Use existing UserRepository pattern
- Apply validation at API layer
- Follow TDD workflow

## Tasks
- [ ] Task 1: Specific implementation task
- [ ] Task 2: Specific implementation task
- [ ] Task 3: Specific implementation task

## Manual Tests
- [ ] Test 1: User action → expected outcome
- [ ] Test 2: User action → expected outcome
- [ ] Test 3: Edge case → expected handling

## Deliverables
- Working feature in dev environment
- Automated tests (unit + integration)
- Manual test results documented
- Updated README if needed

## Dependencies
- Requires Feature X to be completed
- Needs database migration
- Depends on external API setup

## Open Questions
- Question 1?
- Question 2?

## Notes
Additional context, research findings, links
```

**When to create:**
- During feature planning
- Before implementation
- When refining backlog

**Workflow:**
1. Create from template (`/backlog/_feature-template.md`)
2. Refine with `/feat-refine` command
3. Implement with `/feat-imp` command
4. Mark COMPLETED when done

---

### Implementation Plans (`/docs/implementation.md`)

**Purpose:** Step-by-step iteration plan for building features

**Template:**

```markdown
# Implementation Plan: [Feature/System Name]

**Created:** YYYY-MM-DD
**Status:** IN PROGRESS

## Architecture

### File Structure
```
src/
  domain/
    entities/
      - Order.ts
      - OrderItem.ts
    value-objects/
      - Money.ts
  application/
    use-cases/
      - CreateOrder.ts
      - CancelOrder.ts
    ports/
      - IOrderRepository.ts
  infrastructure/
    repositories/
      - OrderRepository.ts
```

### Module Boundaries
- **Domain:** Order, OrderItem, Money (pure logic)
- **Application:** CreateOrder, CancelOrder (orchestration)
- **Infrastructure:** OrderRepository (Prisma adapter)

### Key Design Decisions
- Use value objects for Money to enforce currency rules
- Repository returns domain entities (no ORM leakage)
- Use cases handle transaction boundaries

---

## Iterations

### Iteration 1: Domain Layer - Order Entity

**Status:** COMPLETED
**Completed:** 2025-11-15

**Scope:** Create core Order entity with business rules

**TDD Phases:**
- Plan: Design Order entity with items and total calculation
- Test: Write tests for addItem, removeItem, calculateTotal
- Implement: Order entity with value objects
- Refactor: Extract calculation logic to methods
- Review: Verify SOLID principles, no infrastructure deps

**Deliverables:**
- [x] Order entity class
- [x] OrderItem value object
- [x] Money value object
- [x] Unit tests (100% coverage)
- [x] No infrastructure dependencies

**Dependencies:** None

**Questions:** None

**Agent:** dev-core-domain (parallelizable)

---

### Iteration 2: Application Layer - CreateOrder Use Case

**Status:** IN PROGRESS
**Started:** 2025-11-16

**Scope:** Implement use case for creating orders

**TDD Phases:**
- Plan: Design CreateOrder use case with port for repository
- Test: Write tests for successful creation and validation errors
- Implement: Use case with input validation
- Refactor: Extract validation logic
- Review: Check dependency inversion (port, not concrete adapter)

**Deliverables:**
- [ ] CreateOrder use case
- [ ] IOrderRepository port interface
- [ ] Input DTO validation
- [ ] Unit tests with mocked repository
- [ ] Error handling for invalid inputs

**Dependencies:** Iteration 1 (Order entity)

**Questions:** None

**Agent:** dev-core-application (parallelizable)

---

### Iteration 3: Infrastructure Layer - Order Repository

**Status:** PENDING

**Scope:** Implement Prisma repository adapter

**TDD Phases:**
- Plan: Design Prisma schema and repository implementation
- Test: Write integration tests with test database
- Implement: Repository with Prisma client
- Refactor: Extract entity mapping logic
- Review: Verify no domain logic in repository

**Deliverables:**
- [ ] OrderRepository implements IOrderRepository
- [ ] Prisma schema for orders table
- [ ] Entity ↔ ORM model mapping
- [ ] Integration tests with test DB
- [ ] Transaction support

**Dependencies:** Iteration 2 (IOrderRepository port)

**Questions:** None

**Agent:** dev-adapter-db-prisma (parallelizable)

---

### Iteration 4: API Layer - Order Endpoints

**Status:** PENDING

**Scope:** Expose CreateOrder via REST API

**TDD Phases:**
- Plan: Design POST /orders endpoint with Zod validation
- Test: Write API integration tests
- Implement: Express route with use case invocation
- Refactor: Extract validation middleware
- Review: Check error handling and status codes

**Deliverables:**
- [ ] POST /orders endpoint
- [ ] Zod schema for request validation
- [ ] API integration tests
- [ ] Error responses (400, 500)
- [ ] OpenAPI documentation

**Dependencies:** Iteration 3 (OrderRepository)

**Questions:** None

**Agent:** dev-api (parallelizable)

---

## Planning Principles

1. **Incremental Delivery:** Each iteration = working feature
2. **TDD Mandatory:** Tests before code, always
3. **Dependency Order:** Foundation → features → polish
4. **Simple First:** Skip optional unless requested
5. **Assume Best Practices:** Only ask if genuinely ambiguous
```

**When to create:**
- After specs are complete
- Before implementation starts
- Using `/plan` command

**Iteration Format:**
- Sequential: Iteration 1, 2, 3, etc.
- Status: PENDING → IN PROGRESS → COMPLETED
- Agent: Specify if parallelizable via specialized agent
- Clear dependencies between iterations

---

## Section 2: Iteration Tracking

### Status Values

**PENDING:** Not started yet
- Blocked by dependencies
- Waiting in backlog
- Ready but not prioritized

**IN PROGRESS:** Currently being implemented
- Active development
- Tests being written
- Code being reviewed

**COMPLETED:** Done and verified
- All deliverables met
- Tests passing
- Committed to git
- Timestamp added

---

### Completion Criteria

**An iteration is COMPLETED when:**

- [ ] All deliverables checked off
- [ ] All automated tests pass (`npm test`)
- [ ] All manual tests completed and documented
- [ ] Code committed with proper message
- [ ] Changeset added (if user-facing change)
- [ ] `docs/implementation.md` updated with status
- [ ] Timestamp added: "Completed: YYYY-MM-DD"
- [ ] Deviations from plan documented
- [ ] Unresolved issues noted

**Never mark COMPLETED if:**
- Tests are failing
- Manual tests incomplete
- Code not committed
- Known bugs exist

---

### Approach Research

**When planning iteration, research approaches:**

```markdown
## Iteration X: Feature Name

### Approach Research

**Option 1: Direct Implementation**
- Method: Build feature with existing patterns
- Pros: Fast, simple, no new dependencies
- Cons: May require manual work
- Effort: 2 hours

**Option 2: Use Library X**
- Method: Integrate third-party library
- Pros: Battle-tested, feature-rich
- Cons: Extra dependency, learning curve
- Effort: 4 hours (including setup)

**Option 3: Custom Framework**
- Method: Build reusable abstraction
- Pros: Tailored to our needs, reusable
- Cons: More upfront work, maintenance burden
- Effort: 8 hours

**Recommendation:** Option 1
**Rationale:** Simplest approach, meets requirements, no new deps
```

**List 2-4 options, recommend one, explain why**

---

## Section 3: Project Structure

### Standard File Locations

**Specifications:**
```
/specs/
  business-specs.md       # Product requirements
  tech-specs.md           # Architecture decisions

/backlog/
  _feature-template.md    # Template for new features
  feature-auth.md         # Feature: Authentication
  feature-payments.md     # Feature: Payment processing

/docs/
  implementation.md       # Current implementation plan
  README.md              # Project overview and setup
```

**Source Code:**
```
/src/
  domain/                 # Pure business logic
    entities/
    value-objects/
    services/
    __tests__/

  application/            # Use cases, ports
    use-cases/
    ports/
    dtos/
    __tests__/

  infrastructure/         # Adapters
    api/                  # HTTP REST API
    cli/                  # CLI commands
    repositories/         # Database
    clients/              # External APIs
    __tests__/
```

**Tests:**
```
# Tests live next to source code
src/domain/entities/__tests__/Order.test.ts
src/infrastructure/repositories/__tests__/OrderRepository.integration.test.ts
```

---

### Folder Conventions

**Hexagonal Architecture Layers:**

1. **Domain (`src/domain/`):**
   - Entities: `Order.ts`, `User.ts`
   - Value objects: `Money.ts`, `Email.ts`
   - Domain services: `PricingService.ts`
   - **NO dependencies on infrastructure**

2. **Application (`src/application/`):**
   - Use cases: `CreateOrder.ts`, `CancelOrder.ts`
   - Ports: `IOrderRepository.ts`, `IEmailService.ts`
   - DTOs: `CreateOrderInput.ts`, `OrderOutput.ts`
   - **Depends on domain, defines ports for infrastructure**

3. **Infrastructure (`src/infrastructure/`):**
   - API adapters: `routes/orderRoutes.ts`
   - DB adapters: `repositories/OrderRepository.ts`
   - External API clients: `clients/StripeClient.ts`
   - CLI adapters: `commands/createOrder.ts`
   - **Implements ports from application layer**

---

### Build Commands

**Development:**
```bash
npm run dev           # Start dev server with hot reload
npm run build         # Build for production
npm test              # Run all tests
npm test -- --watch   # Run tests in watch mode
```

**Type Checking:**
```bash
npm run typecheck     # Run TypeScript compiler without emitting
tsc --noEmit          # Alternative type check
```

**Linting:**
```bash
npm run lint          # Run ESLint
npm run lint:fix      # Auto-fix linting issues
```

---

## Section 4: Git Workflow

### Branch Naming

**Convention:** `jrob/[feature-name]`

Examples:
```bash
git checkout -b jrob/user-authentication
git checkout -b jrob/payment-integration
git checkout -b jrob/fix-login-bug
```

**Guidelines:**
- Prefix: `jrob/` (user-specific)
- Feature name: Lowercase, hyphen-separated
- Descriptive but concise
- Avoid issue numbers in branch name

---

### Commit Messages

**Format:** `[type]: [description]`

**Types:**
- `feat:` New feature or iteration
- `fix:` Bug fix
- `test:` Adding or updating tests
- `refactor:` Code restructuring (no behavior change)
- `docs:` Documentation only
- `chore:` Tooling, dependencies, config
- `spec:` Specification updates

**Examples:**

```bash
# Feature implementation
git commit -m "feat: iteration 1 - order entity with business rules"

# Bug fix
git commit -m "fix: validation error on empty order items"

# Test addition
git commit -m "test: add integration tests for order repository"

# Refactoring
git commit -m "refactor: extract price calculation to domain service"

# Spec update
git commit -m "spec: feat user authentication requirements"
```

**Guidelines:**
- Concise (50 chars or less for subject)
- Imperative mood ("add" not "added")
- No period at end
- Body for details (if needed)

---

### Committing Changes

**Workflow:**

```bash
# 1. Check what's changed
git status

# 2. Stage specific files (preferred over `git add .`)
git add src/domain/entities/Order.ts
git add src/domain/entities/__tests__/Order.test.ts

# 3. Commit with message
git commit -m "feat: iteration 1 - order entity

- Add Order entity with items collection
- Add calculateTotal method
- Implement Money value object
- Unit tests with 100% coverage

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>"

# 4. Push to remote (if needed)
git push -u origin jrob/order-entity
```

**Best Practices:**
- Commit working code only (all tests pass)
- Commit per iteration or logical unit
- Don't commit half-done work
- Include co-author attribution for AI assistance

---

### Changesets

**Purpose:** Track changes for version releases and changelogs

**When to create:**
- User-facing feature added
- Bug fix that affects users
- Breaking change (API, behavior)
- Performance improvement users notice

**When NOT to create:**
- Internal refactoring
- Test updates only
- Documentation changes
- Dev tooling updates

---

**Format:**

```bash
# 1. Create changeset file
# File: .changeset/0000-order-creation.md

---
"vibe-template": patch
---

Add order creation feature with validation and persistence.
```

**Version types:**
- **patch:** Bug fixes, minor improvements (1.0.0 → 1.0.1)
- **minor:** New features, backwards compatible (1.0.0 → 1.1.0)
- **major:** Breaking changes (1.0.0 → 2.0.0)

---

**Workflow:**

```bash
# After committing feature
# Create changeset file manually

# File: .changeset/0001-user-auth.md
cat > .changeset/0001-user-auth.md << 'EOF'
---
"vibe-template": minor
---

Add user authentication with JWT tokens and session management.
EOF

# Stage and commit changeset
git add .changeset/0001-user-auth.md
git commit -m "chore: add changeset for user auth"
```

**Guidelines:**
- Sequential numbering: 0000, 0001, 0002, etc.
- Short description in filename: `0001-user-auth.md`
- User-facing description in content
- One changeset per feature/fix
- Add with separate commit after feature commit

---

## Quick Reference

**Spec Locations:**
- Business: `/specs/business-specs.md`
- Tech: `/specs/tech-specs.md`
- Features: `/backlog/*.md`
- Implementation: `/docs/implementation.md`

**Iteration Status:**
- PENDING → IN PROGRESS → COMPLETED

**Completion Checklist:**
- [ ] All deliverables done
- [ ] All tests pass
- [ ] Manual tests completed
- [ ] Code committed
- [ ] Changeset added (if user-facing)
- [ ] Status updated in docs

**Git Commands:**
```bash
git checkout -b jrob/feature-name
git commit -m "feat: iteration X - description"
# Create changeset if needed
git push -u origin jrob/feature-name
```

**Build Commands:**
```bash
npm test              # Run tests
npm run dev           # Start dev server
npm run typecheck     # Type check
```

---

**End of Skill: workflow-project**
