# Workflow: Agent Delegation

**Purpose:** Master agent selection, task delegation patterns, parallel vs sequential execution, and Task tool invocation for efficient workflows.

**Version:** 1.0
**Lines:** ~500

---

## When to Use This Skill

Load this skill when:
- Planning which agent to use for implementation
- Deciding whether to delegate work
- Invoking Task tool with specialized agents
- Determining parallel vs sequential execution
- Optimizing workflow efficiency

---

## Core Principles

1. **Delegate Specialized Work:** Use agents for their expertise
2. **Match Task to Agent:** Choose right agent for the job
3. **Parallelize When Possible:** Run independent tasks concurrently
4. **Sequence When Dependent:** Run dependent tasks in order
5. **Provide Full Context:** Give agents everything they need

---

## Section 1: Agent Responsibility Matrix

### Available Agents

| Agent | Responsibility | Layer | When to Use |
|-------|---------------|-------|-------------|
| **dev-core** | Domain + Application layers | Core | Entities, value objects, use cases, ports |
| **dev-core-domain** | Domain layer only | Core | Only entities, value objects, domain services |
| **dev-core-application** | Application layer only | Core | Only use cases, ports, DTOs |
| **dev-adapter-db** | Database adapters | Infrastructure | Repositories, ORM, migrations |
| **dev-adapter-db-prisma** | Prisma-specific DB adapters | Infrastructure | Prisma repositories, schema |
| **dev-adapter-db-drizzle** | Drizzle-specific DB adapters | Infrastructure | Drizzle repositories, schema |
| **dev-adapter-db-raw** | Raw SQL adapters | Infrastructure | Custom SQL, Knex, connection pools |
| **dev-api** | HTTP REST APIs | Infrastructure | API routes, validation, OpenAPI |
| **dev-i-cli** | Interactive CLIs | Infrastructure | CLI commands, prompts, args |
| **dev-adapter-api-client** | External API clients | Infrastructure | HTTP clients, API integration |
| **react-dev** | React web apps | Infrastructure | React components, hooks, state |
| **react-native-dev** | React Native mobile apps | Infrastructure | Mobile components, navigation |
| **backend-architect** | Architecture design | Planning | System design, folder structure |
| **review-clean-code** | Code quality review | Review | SOLID, refactoring, code smells |

---

### Agent Capabilities

**dev-core (General):**
- ✅ Create entities AND use cases together
- ✅ Full vertical slice (domain → application)
- ✅ Loads both dev-core-domain + dev-core-application skills
- ✅ Best for iterations spanning multiple layers

**dev-core-domain (Specialized):**
- ✅ Pure domain logic only
- ✅ Entities, value objects, domain services
- ❌ No use cases, no ports (application layer)
- ✅ Best for domain modeling iterations

**dev-core-application (Specialized):**
- ✅ Use cases, ports, DTOs only
- ✅ Orchestration logic
- ❌ No entities (domain layer)
- ✅ Best for use case-focused iterations

**dev-adapter-db (General):**
- ✅ All database technologies
- ✅ Detects ORM from package.json
- ✅ Loads core + tech-specific skill
- ✅ Best for repository implementations

**dev-adapter-db-prisma (Specialized):**
- ✅ Prisma-specific patterns
- ✅ Schema, migrations, client
- ❌ Won't help with Drizzle or raw SQL
- ✅ Best for Prisma-only work

**backend-architect:**
- ✅ Design folder structure
- ✅ Define module boundaries
- ✅ Plan system architecture
- ❌ Doesn't write implementation code
- ✅ Best for planning phase

**review-clean-code:**
- ✅ Review existing code
- ✅ Identify SOLID violations
- ✅ Suggest refactorings
- ❌ Doesn't implement features
- ✅ Best after writing code

---

## Section 2: Task → Agent Mapping

### Decision Tree

**Start here: What type of work?**

```
Is it PLANNING/DESIGN?
├─ YES → Architecture/folder structure? → backend-architect
├─ YES → Refine spec? → Don't delegate, handle directly
└─ NO → Continue...

Is it IMPLEMENTATION?
├─ Domain layer (entities, VOs)?
│  ├─ Domain only? → dev-core-domain
│  └─ Domain + Use cases? → dev-core
├─ Application layer (use cases, ports)?
│  ├─ Use cases only? → dev-core-application
│  └─ Use cases + Domain? → dev-core
├─ Database (repositories)?
│  ├─ Prisma? → dev-adapter-db-prisma or dev-adapter-db
│  ├─ Drizzle? → dev-adapter-db-drizzle or dev-adapter-db
│  └─ Raw SQL? → dev-adapter-db-raw or dev-adapter-db
├─ REST API (endpoints)?
│  └─ Use dev-api
├─ CLI (commands)?
│  └─ Use dev-i-cli
├─ External API integration?
│  └─ Use dev-adapter-api-client
├─ React web UI?
│  └─ Use react-dev
├─ React Native mobile?
│  └─ Use react-native-dev
└─ Multiple layers? → Use appropriate agents in sequence

Is it REVIEW/QUALITY?
└─ Code review for clean code? → review-clean-code
```

---

### Common Task Patterns

**Pattern 1: Full Feature (multiple layers)**

Task: "Implement user registration"

Approach:
1. Use `dev-core` for domain + use case (or split into dev-core-domain → dev-core-application)
2. Use `dev-adapter-db` for repository
3. Use `dev-api` for REST endpoint

Delegation:
```markdown
**Iteration 1:** Domain + Use Case
- Agent: dev-core
- Deliverable: User entity, RegisterUser use case

**Iteration 2:** Repository
- Agent: dev-adapter-db-prisma
- Deliverable: UserRepository with Prisma

**Iteration 3:** API
- Agent: dev-api
- Deliverable: POST /users endpoint
```

---

**Pattern 2: Domain-Only Work**

Task: "Create Order entity with business rules"

Approach:
- Use `dev-core-domain` (specialized for domain)

Delegation:
```markdown
**Iteration 1:** Order Entity
- Agent: dev-core-domain
- Deliverable: Order entity, OrderItem VO, Money VO
```

---

**Pattern 3: Use Case Development**

Task: "Implement checkout workflow use case"

Approach:
- Assume entities exist
- Use `dev-core-application` (specialized for use cases)

Delegation:
```markdown
**Iteration X:** Checkout Use Case
- Agent: dev-core-application
- Deliverable: CheckoutOrder use case, IPaymentGateway port
- Dependencies: Order entity (from previous iteration)
```

---

**Pattern 4: Database-Heavy Work**

Task: "Implement complex product search with filters"

Approach:
- Use `dev-adapter-db-prisma` if using Prisma
- Or `dev-adapter-db` if technology undecided

Delegation:
```markdown
**Iteration X:** Product Search Repository
- Agent: dev-adapter-db-prisma
- Deliverable: ProductRepository.search() with filters
```

---

**Pattern 5: API Development**

Task: "Create REST API for product catalog"

Approach:
- Use `dev-api` for all endpoints

Delegation:
```markdown
**Iteration X:** Product API
- Agent: dev-api
- Deliverable: GET /products, GET /products/:id, POST /products
- Dependencies: Product use cases + repository
```

---

**Pattern 6: CLI Tool**

Task: "Build interactive CLI for data import"

Approach:
- Use `dev-i-cli` for CLI-specific work

Delegation:
```markdown
**Iteration X:** Import CLI
- Agent: dev-i-cli
- Deliverable: `import` command with prompts and validation
```

---

**Pattern 7: Architecture Planning**

Task: "Plan the system design for multi-tenant app"

Approach:
- Use `backend-architect` BEFORE implementation

Delegation:
```markdown
**Planning Phase:** Architecture Design
- Agent: backend-architect
- Deliverable: Folder structure, module boundaries, design decisions
- Then: Create implementation plan from architecture
```

---

**Pattern 8: Code Quality Review**

Task: "Review authentication module for SOLID violations"

Approach:
- Use `review-clean-code` AFTER implementation

Delegation:
```markdown
**Review Phase:** Clean Code Audit
- Agent: review-clean-code
- Deliverable: List of issues, refactoring suggestions
- Then: Apply refactorings
```

---

## Section 3: Delegation Patterns

### When to Delegate

**DO delegate when:**

✅ Task matches specialized agent expertise
- Example: Database repository → dev-adapter-db

✅ Task is well-defined and self-contained
- Example: "Implement User entity" (clear scope)

✅ You're in a command workflow that supports delegation
- Example: /imp command reads iteration, delegates to agent

✅ Task can run autonomously
- Example: Agent has all context from spec

✅ Parallel execution possible
- Example: Multiple independent iterations

---

### When NOT to Delegate

❌ Task is trivial (< 5 minutes)
- Example: Adding a single method to existing class

❌ Task is exploratory (requirements unclear)
- Example: "Figure out how to implement X"

❌ You need interactive feedback
- Example: Iterating on design with user input

❌ Task spans too many layers without clear boundaries
- Example: "Build the entire app" (too broad)

❌ You're already in an agent (don't nest deeply)
- Example: Agent shouldn't spawn another agent for tiny task

---

### Delegation Workflow

**Step 1: Identify Task**
- Read iteration from implementation plan
- Extract scope, deliverables, dependencies

**Step 2: Match to Agent**
- Use decision tree (Section 2)
- Choose most specific agent that fits
- Example: dev-core-domain > dev-core if domain-only

**Step 3: Prepare Context**
- Gather spec/iteration details
- Note dependencies (what must exist)
- List deliverables explicitly

**Step 4: Invoke Task Tool**
- Provide full context in prompt
- Specify agent type
- Set expectations clearly

**Step 5: Verify Completion**
- Check deliverables met
- Run tests
- Update iteration status

---

## Section 4: Parallel vs Sequential

### Parallel Execution

**Use parallel when tasks are INDEPENDENT:**

✅ No shared files (different modules)
✅ No data dependencies (can run simultaneously)
✅ Order doesn't matter (commutative)

**Example:**

```markdown
## Iteration 1: User Entity
**Agent:** dev-core-domain
**Files:** src/domain/entities/User.ts

## Iteration 2: Product Entity
**Agent:** dev-core-domain
**Files:** src/domain/entities/Product.ts
```

These can run in parallel because:
- Different entities (User vs Product)
- Different files (no conflict)
- No dependencies between them

**Task Tool Invocation:**

```markdown
I'll invoke both agents in parallel using the Task tool:

1. Task tool with dev-core-domain for User entity
2. Task tool with dev-core-domain for Product entity

(Both invocations in SINGLE message)
```

---

### Sequential Execution

**Use sequential when tasks are DEPENDENT:**

❌ Later task needs earlier task's output
❌ Same files being modified
❌ Build-depends-on relationship

**Example:**

```markdown
## Iteration 1: Order Entity
**Agent:** dev-core-domain
**Files:** src/domain/entities/Order.ts

## Iteration 2: IOrderRepository Port
**Agent:** dev-core-application
**Files:** src/application/ports/IOrderRepository.ts
**Dependencies:** Iteration 1 (needs Order entity)

## Iteration 3: OrderRepository Implementation
**Agent:** dev-adapter-db-prisma
**Files:** src/infrastructure/repositories/OrderRepository.ts
**Dependencies:** Iteration 2 (needs IOrderRepository port)
```

These MUST run sequentially because:
- Iteration 2 needs Order entity from Iteration 1
- Iteration 3 implements port from Iteration 2
- Clear dependency chain: 1 → 2 → 3

**Task Tool Invocation:**

```markdown
I'll invoke agents sequentially:

1. First: dev-core-domain for Order entity
   (Wait for completion)

2. Then: dev-core-application for IOrderRepository port
   (Wait for completion)

3. Finally: dev-adapter-db-prisma for repository
   (Wait for completion)
```

---

### Hybrid Approach

**Some tasks can be parallel, others sequential:**

```markdown
## Iteration 1: Domain Entities (PARALLEL)
### 1a. User Entity
**Agent:** dev-core-domain

### 1b. Product Entity
**Agent:** dev-core-domain

## Iteration 2: Use Cases (SEQUENTIAL after Iteration 1)
### 2a. RegisterUser Use Case
**Agent:** dev-core-application
**Dependencies:** Iteration 1a

### 2b. CreateProduct Use Case
**Agent:** dev-core-application
**Dependencies:** Iteration 1b
```

**Execution:**
1. Run 1a and 1b in PARALLEL (independent entities)
2. Wait for both to complete
3. Run 2a and 2b in SEQUENTIAL (or parallel if independent)

---

## Section 5: Task Tool Invocation

### Basic Template

```markdown
I'll use the Task tool to delegate this work to the [agent-name] agent.

**Task:** [Brief description]
**Agent:** [agent-type]
**Context:** [Full iteration details from spec]
```

**Then invoke Task tool with:**

```
subagent_type: [agent-name]
description: [Short 3-5 word description]
prompt: [Detailed task with full context]
```

---

### Invocation Examples

**Example 1: Domain Entity**

```markdown
I'll use the Task tool to delegate the Order entity creation to the dev-core-domain agent.

Task tool invocation:
- subagent_type: dev-core-domain
- description: Create Order domain entity
- prompt: |
    Implement the Order entity for the e-commerce system.

    **Scope:** Create Order entity with business rules

    **Requirements:**
    - Order aggregates OrderItems
    - Calculate total from items
    - Apply discount logic
    - Validate order state transitions (pending → confirmed → shipped)

    **Deliverables:**
    - Order entity class
    - OrderItem value object
    - Unit tests (100% coverage)
    - No infrastructure dependencies

    **TDD:** Write tests first, then implement

    Follow workflow-implementation skill for TDD cycle.
```

---

**Example 2: Use Case**

```markdown
I'll delegate the CreateOrder use case to the dev-core-application agent.

Task tool invocation:
- subagent_type: dev-core-application
- description: Implement CreateOrder use case
- prompt: |
    Implement the CreateOrder use case.

    **Scope:** Create use case for order creation workflow

    **Requirements:**
    - Accept order items and customer ID as input
    - Validate input (items not empty, customer exists)
    - Create Order entity
    - Save via IOrderRepository port
    - Return order ID

    **Deliverables:**
    - CreateOrder use case class
    - IOrderRepository port interface
    - CreateOrderInput DTO
    - Unit tests with mocked repository
    - Error handling for validation failures

    **Dependencies:**
    - Order entity exists (from Iteration 1)

    **TDD:** Write tests first, then implement

    Follow workflow-implementation skill for TDD cycle.
```

---

**Example 3: Repository**

```markdown
I'll delegate the repository implementation to the dev-adapter-db-prisma agent.

Task tool invocation:
- subagent_type: dev-adapter-db-prisma
- description: Implement OrderRepository with Prisma
- prompt: |
    Implement the OrderRepository using Prisma.

    **Scope:** Database adapter for Order persistence

    **Requirements:**
    - Implement IOrderRepository port
    - Use Prisma client for DB operations
    - Map between Order entity and Prisma model
    - Support transactions
    - Handle errors (DB connection, constraints)

    **Deliverables:**
    - OrderRepository class implementing IOrderRepository
    - Prisma schema for orders and order_items tables
    - Entity ↔ ORM model mapping functions
    - Integration tests with test database
    - Transaction support for multi-table operations

    **Dependencies:**
    - IOrderRepository port (from Iteration 2)
    - Order entity (from Iteration 1)

    **TDD:** Write integration tests first

    Follow workflow-implementation skill for TDD cycle.
```

---

**Example 4: API Endpoint**

```markdown
I'll delegate the API implementation to the dev-api agent.

Task tool invocation:
- subagent_type: dev-api
- description: Implement POST /orders endpoint
- prompt: |
    Implement the POST /orders REST API endpoint.

    **Scope:** HTTP API for order creation

    **Requirements:**
    - POST /orders endpoint
    - Accept JSON body with items array and customerId
    - Validate request with Zod schema
    - Invoke CreateOrder use case
    - Return 201 Created with order ID
    - Return 400 Bad Request for validation errors
    - Return 500 Internal Server Error for system errors

    **Deliverables:**
    - POST /orders route handler
    - Zod schema for request validation
    - API integration tests (supertest)
    - Error response formatting
    - OpenAPI documentation for endpoint

    **Dependencies:**
    - CreateOrder use case (from Iteration 2)
    - OrderRepository (from Iteration 3)

    **TDD:** Write API tests first

    Follow workflow-implementation skill for TDD cycle.
```

---

**Example 5: Parallel Invocation**

```markdown
I'll invoke multiple agents in parallel for independent tasks.

(In SINGLE message, invoke both Task tools):

1. Task tool for User entity:
   - subagent_type: dev-core-domain
   - description: Create User entity
   - prompt: [Full context for User entity]

2. Task tool for Product entity:
   - subagent_type: dev-core-domain
   - description: Create Product entity
   - prompt: [Full context for Product entity]

These tasks are independent (different entities, different files),
so they can run in parallel for efficiency.
```

---

### Prompt Best Practices

**DO:**
- ✅ Provide full context (scope, requirements, deliverables)
- ✅ List dependencies explicitly
- ✅ Specify TDD requirement
- ✅ Reference workflow-implementation skill
- ✅ Be specific about deliverables
- ✅ Include acceptance criteria

**DON'T:**
- ❌ Be vague ("implement the feature")
- ❌ Assume agent knows context
- ❌ Skip deliverables list
- ❌ Forget to mention TDD
- ❌ Omit dependencies
- ❌ Use placeholders or "TBD"

---

## Quick Reference

**Agent Selection:**
- Domain only → dev-core-domain
- Use cases only → dev-core-application
- Domain + Use cases → dev-core
- DB (Prisma) → dev-adapter-db-prisma or dev-adapter-db
- REST API → dev-api
- CLI → dev-i-cli
- External API → dev-adapter-api-client
- React web → react-dev
- React Native → react-native-dev
- Architecture → backend-architect
- Code review → review-clean-code

**Parallel vs Sequential:**
- Independent tasks → Parallel (single message, multiple Task tools)
- Dependent tasks → Sequential (wait between invocations)

**Invocation Template:**
```
subagent_type: [agent-name]
description: [3-5 words]
prompt: |
  Scope: [What to build]
  Requirements: [Functional requirements]
  Deliverables: [Checklist]
  Dependencies: [What must exist]
  TDD: Write tests first
```

---

**End of Skill: workflow-agent-delegation**
