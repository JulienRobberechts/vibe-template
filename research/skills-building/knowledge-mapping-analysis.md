# Knowledge Mapping Analysis

## Current Knowledge Distribution

### Commands → Knowledge Extraction Map

#### imp.md (97 lines)

**Extractable Knowledge:**
```
Lines 18-23: Agent Selection Logic → workflow-agent-delegation
  "Identify appropriate specialized agent: dev-core, dev-api,
   dev-adapter-db, dev-i-cli, react-dev, react-native-dev"

Lines 35-42: TDD Cycle Pattern → workflow-tdd
  "Test → Implement → Refactor → Verify → Continue"
  Run tests, manual tests, responsive testing

Lines 64-82: Autonomy Rules → workflow-autonomy
  "Work continuously without stopping unless..."
  "Do NOT stop or ask questions for..."
  "When blocked..."

Lines 31-33: Baseline Verification → workflow-tdd
  "Run existing tests, Run dev"

Lines 50-57: Integration Testing → workflow-tdd
  "Run full test suite, dev server, manual tests, responsive"

Lines 7-12: State Resume Pattern → project-structure
  "Read docs/implementation.md, check git status, read todos"

Lines 92-94: Output Format → project-structure
  "What was built, Files modified, Manual test results, Next iteration"
```

**Remaining Procedural Logic:** ~30 lines
- Workflow sequence
- When to read what
- Commit step

---

#### feat-imp.md (81 lines)

**Extractable Knowledge:**
```
Lines 49-65: Autonomy Rules → workflow-autonomy (DUPLICATE of imp.md)
Lines 24-29: TDD Cycle → workflow-tdd (DUPLICATE of imp.md)
Lines 5-9: Resume State → project-structure (backlog tracking)
Lines 37-43: Complete Feature → project-structure (feature status tracking)
```

**Duplicate with imp.md:** ~70%
**Unique:** Feature-specific file paths (backlog vs docs)

---

#### plan.md (58 lines)

**Extractable Knowledge:**
```
Lines 10-12: Agent Selection → workflow-agent-delegation
  "Architectural planning → backend-architect"
  "Feature planning → Plan directly"

Lines 17-46: Implementation Plan Structure → workflow-spec-management
  Template format, iteration structure, TDD phases

Lines 48-54: Planning Principles → workflow-spec-management
  "Incremental delivery, TDD mandatory, Dependency order,
   Simple first, Assume best practices"
```

**Remaining:** ~20 lines (workflow steps)

---

#### feat-refine.md (64 lines)

**Extractable Knowledge:**
```
Lines 18-24: Research Solutions Pattern → workflow-spec-management
  "Analyze codebase, research best practices, consider constraints"

Lines 25-28: Propose Approaches → workflow-spec-management
  "List 3+ approaches with pros/cons/effort"

Lines 36-42: Completeness Criteria → workflow-spec-management
  "All sections filled, no ambiguity, ready for implementation"

Lines 47-58: Autonomy Rules → workflow-autonomy
  Similar patterns to imp.md
```

---

#### fix.md (60 lines)

**Extractable Knowledge:**
```
Lines 27-35: TDD Debugging Cycle → workflow-tdd
  "Write test reproducing bug → Fix → Verify"

Lines 40-56: Autonomy Rules → workflow-autonomy (DUPLICATE)

Lines 17-25: Baseline + Integration → workflow-tdd (DUPLICATE)
```

**Duplicate with imp.md:** ~80%

---

### Skills → Reorganization Map

#### dev-core (1394 lines) → Split

**Reason:** Covers two distinct layers (domain + application)

```
SPLIT:
├── dev-core-domain (700 lines)
│   ├── Entities (lines 400-514)
│   ├── Value Objects (lines 516-637)
│   ├── Domain Services (lines 639-678)
│   ├── Domain Exceptions (lines 680-702)
│   ├── Functional IDs (lines 54-276)
│   └── Ubiquitous Language (lines 283-399)
│
├── dev-core-application (600 lines)
│   ├── Use Cases (lines 724-817)
│   ├── Application Services (lines 1115-1154)
│   ├── DTOs (lines 819-847)
│   ├── Ports Design (lines 849-1113)
│   └── Application Exceptions (lines 1156-1167)
│
└── dev-core (100 lines) - Coordinator
    ├── Overview of hexagonal core
    ├── When to use domain vs application
    └── References to sub-skills
```

**Agent Implications:**
- Create `dev-core-domain` agent (domain-only work)
- Create `dev-core-application` agent (use cases only)
- Keep `dev-core` agent (uses both)

---

#### dev-adapter-db (1637 lines) → Split by Technology

**Reason:** Multi-technology, users only need subset

```
SPLIT:
├── dev-adapter-db-core (600 lines)
│   ├── Repository Pattern (lines 1-200)
│   ├── Entity Mapping (lines 201-350)
│   ├── Transactions (lines 351-450)
│   ├── Testing (lines 451-600)
│   └── Common Patterns
│
├── dev-adapter-db-prisma (400 lines)
│   ├── Prisma Client usage
│   ├── Schema definition
│   ├── Migrations
│   ├── Type generation
│   └── Prisma-specific testing
│
├── dev-adapter-db-drizzle (400 lines)
│   ├── Drizzle ORM patterns
│   ├── Schema builders
│   ├── Query building
│   └── Type-safe queries
│
└── dev-adapter-db-raw (237 lines)
    ├── Raw SQL approaches
    ├── Knex query builder
    ├── Connection pooling
    └── Manual mapping
```

**Loading Strategy:**
- Always load: dev-adapter-db-core
- Load on-demand: Technology-specific skills based on project detection

---

#### react-dev (859 lines) → Split by Concern

**Reason:** Testing vs development are different workflows

```
SPLIT:
├── react-dev-core (400 lines)
│   ├── Component Patterns
│   ├── Hooks Usage
│   ├── State Management
│   └── Props Design
│
├── react-dev-testing (300 lines)
│   ├── Testing Library
│   ├── Component Tests
│   ├── Hook Tests
│   └── Integration Tests
│
├── react-dev-performance (159 lines)
│   ├── Memoization
│   ├── Code Splitting
│   ├── Virtualization
│   └── Profiling
│
└── react-shared (300 lines) - NEW
    ├── Common with react-native
    ├── Hooks fundamentals
    ├── State patterns
    └── Component composition
```

**Usage:**
- Development: Load react-dev-core + react-shared
- Testing: Load react-dev-testing + react-shared
- Optimization: Load react-dev-performance + react-shared

---

## Knowledge Duplication Matrix

### Cross-Command Duplication

| Knowledge Domain | imp.md | feat-imp.md | fix.md | plan.md | feat-refine.md |
|-----------------|--------|-------------|--------|---------|----------------|
| TDD Cycle | ✓ | ✓ | ✓ | ✓ | - |
| Autonomy Rules | ✓ | ✓ | ✓ | - | ✓ |
| Baseline Verify | ✓ | ✓ | ✓ | - | - |
| Integration Test | ✓ | ✓ | ✓ | - | - |
| Agent Selection | ✓ | - | - | ✓ | - |
| Spec Templates | - | - | - | ✓ | ✓ |
| State Tracking | ✓ | ✓ | - | - | - |

**Duplication Score:**
- TDD Cycle: 80% overlap across 4 commands
- Autonomy Rules: 95% overlap across 4 commands
- Baseline/Integration: 100% overlap across 3 commands

**Impact:** ~250 lines of duplicated knowledge across commands

---

## New Skills Specification

### 1. workflow-tdd (~300 lines)

**Structure:**
```markdown
# TDD Workflow Skill

## Baseline Verification
- npm test (all tests pass)
- npm run dev (server starts)
- No compilation errors

## Red-Green-Refactor Cycle
1. Write failing test
2. Minimal code to pass
3. Refactor
4. Verify all tests pass
5. Continue to next feature

## Test Types
- Unit tests (domain/application)
- Integration tests (adapters)
- E2E tests (full system)

## Integration Testing Checklist
- [ ] Run full test suite
- [ ] Run dev server
- [ ] Complete manual tests from spec
- [ ] Test responsive (if UI)
- [ ] Verify all edge cases

## TDD for Debugging
1. Write test reproducing bug
2. Verify test fails
3. Fix minimal code
4. Verify test passes
5. Run full suite

## Test Organization
- Mirror source structure
- One test file per source file
- Descriptive test names
- AAA pattern (Arrange, Act, Assert)
```

---

### 2. workflow-autonomy (~250 lines)

**Structure:**
```markdown
# Autonomous Workflow Skill

## When to Work Autonomously

Work continuously WITHOUT stopping for:
- Implementation details
- Code style choices
- Minor architectural decisions
- Test naming or structure
- Variable/function naming
- File organization
- Common patterns
- Tool selection (if obvious)

## When to Stop and Ask

ONLY stop for:
- Missing critical information
  - Unclear spec
  - Ambiguous requirement
  - Conflicting goals
- External dependencies
  - API keys
  - Missing packages
  - Environment issues
- Critical errors blocking progress
  - Cannot resolve independently
  - System-level issues

## Decision-Making Framework

### Ask Yourself:
1. "Can I make a reasonable choice based on best practices?"
   → YES: Proceed autonomously
   → NO: Ask user

2. "Is this blocking all progress?"
   → YES: Ask user immediately
   → NO: Continue with unblocked work

3. "Would the user care about this detail?"
   → YES: Ask user
   → NO: Decide and proceed

### Resolution Attempts Before Asking
1. Read existing code for patterns
2. Check documentation
3. Search codebase for similar cases
4. Apply industry best practices
5. If STILL blocked → Ask ONE specific question

## Question Quality

### Good Questions:
✓ "Should payments use Stripe or PayPal?"
✓ "Unclear if users can delete their own posts. Which?"
✓ "API key missing. Where do I find it?"

### Bad Questions (Don't Ask):
✗ "Should I use const or let?"
✗ "What should I name this variable?"
✗ "Which test library?"
✗ "How should I organize imports?"

## Parallel Work While Blocked

If blocked on X:
1. List all unblocked tasks
2. Work on highest-priority unblocked task
3. Return to X when unblocked
4. Never sit idle waiting
```

---

### 3. workflow-spec-management (~300 lines)

**Structure:**
```markdown
# Specification Management Skill

## Spec Types

### Business Spec
- Location: /specs/business-specs.md
- Purpose: Business goals, objectives, scope
- Template: /specs/business-specs.template.md

### Feature Spec
- Location: /backlog/*.md
- Purpose: Individual feature details
- Template: /backlog/_feature-template.md

### Implementation Plan
- Location: /docs/implementation.md
- Purpose: Iteration tracking
- Format: Numbered iterations with status

## Spec Template Structure

### Feature Template Sections
1. Objective (What & Why)
2. User Stories (As a... I want... So that...)
3. Requirements (Functional + Non-functional)
4. Acceptance Criteria (Testable conditions)
5. Technical Implementation (Approach chosen)
6. Tasks (Checklist for implementation)
7. Manual Tests (UI/UX verification)
8. Deliverables (What gets shipped)

## Approach Research Methodology

### Step 1: Analyze Codebase
- Read existing patterns
- Identify libraries in use
- Check architectural decisions
- Note constraints

### Step 2: Research Options
- Industry best practices
- Library documentation
- Similar problems solved
- Performance implications

### Step 3: Propose Approaches (3+ options)

For each approach:
- **Method:** How it works
- **Pros:** Benefits
- **Cons:** Drawbacks
- **Effort:** Time estimate
- **Complexity:** Simple/Medium/Complex

### Step 4: Rank & Recommend
- Simplicity first
- Maintainability second
- Performance third
- Consistency with codebase

## Iteration Status Tracking

### Status Values
- 🔄 IN PROGRESS - Currently being implemented
- ✅ COMPLETED - Finished and tested
- ⏸️ BLOCKED - Cannot proceed
- ❌ SKIPPED - Decided not to implement

### Completion Criteria
- [ ] All tasks completed
- [ ] All tests passing
- [ ] Manual tests verified
- [ ] Documentation updated
- [ ] Code committed

## Spec Completeness Checklist

Before marking spec "Ready for Implementation":
- [ ] All sections filled (no TBD)
- [ ] No ambiguous requirements
- [ ] Technical approach chosen
- [ ] Files to modify listed
- [ ] Edge cases identified
- [ ] Testing strategy defined
- [ ] Success criteria clear
```

---

### 4. workflow-agent-delegation (~200 lines)

**Structure:**
```markdown
# Agent Delegation Skill

## Agent Responsibility Matrix

| Task Type | Agent | Scope |
|-----------|-------|-------|
| Domain entities, value objects, use cases | dev-core | Domain + Application layers |
| REST API endpoints | dev-api | HTTP controllers, routes, validation |
| Database repositories | dev-adapter-db | ORM, queries, migrations |
| External API clients | dev-adapter-api-client | HTTP clients, retry logic |
| CLI commands | dev-i-cli | Argument parsing, prompts |
| React web components | react-dev | Components, hooks, state |
| React Native mobile | react-native-dev | Cross-platform UI |
| Architecture review | backend-architect | System design, structure |
| Code quality review | review-clean-code | SOLID, refactoring |

## Task → Agent Mapping

### Domain Logic
**Indicators:**
- Creating entities
- Defining business rules
- Value objects
- Use cases
- Domain services

**Agent:** dev-core

### API Development
**Indicators:**
- REST endpoints
- HTTP methods
- Request validation
- API documentation
- Route handlers

**Agent:** dev-api

### Database Work
**Indicators:**
- Repository implementations
- Database queries
- Migrations
- ORM configuration
- Transaction management

**Agent:** dev-adapter-db

### Architectural Planning
**Indicators:**
- System design
- Folder structure
- Module boundaries
- Interface definitions
- Workflow mapping

**Agent:** backend-architect

## Delegation Decision Tree

```
Is this pure business logic?
├─ YES → dev-core
└─ NO → Is it HTTP API related?
    ├─ YES → dev-api
    └─ NO → Is it database related?
        ├─ YES → dev-adapter-db
        └─ NO → Is it external API?
            ├─ YES → dev-adapter-api-client
            └─ NO → Is it CLI?
                ├─ YES → dev-i-cli
                └─ NO → Is it UI?
                    ├─ Web → react-dev
                    ├─ Mobile → react-native-dev
                    └─ Unknown → Ask user
```

## Task Tool Invocation Pattern

### Template
```
I'll delegate this to the [AGENT_NAME] agent to [WHAT_IT_WILL_DO].

<Task tool>
  subagent_type: [AGENT_NAME]
  description: [3-5 word summary]
  prompt: |
    [Detailed instructions]

    Context:
    - [Relevant context 1]
    - [Relevant context 2]

    Requirements:
    - [Requirement 1]
    - [Requirement 2]

    Return:
    - [What agent should report back]
</Task>
```

### Example
```
I'll delegate this to the dev-core agent to create the User domain entity.

<Task tool>
  subagent_type: dev-core
  description: Create User entity
  prompt: |
    Create a User entity with the following:

    Context:
    - This is for a user management system
    - Using hexagonal architecture

    Requirements:
    - Email value object
    - User status (active/suspended)
    - Registration timestamp
    - Suspend/activate methods

    Return:
    - Files created
    - Tests written
    - Any ports defined
</Task>
```

## Parallel vs Sequential Delegation

### Parallel (Multiple agents at once)
Use when tasks are independent:
- Different modules
- No shared dependencies
- Can be developed simultaneously

**Example:**
- Agent 1: Create domain entities
- Agent 2: Design API routes
- Agent 3: Set up database schema

### Sequential (One after another)
Use when tasks depend on each other:
- Later task needs output from earlier
- Shared interfaces being defined
- Integration points

**Example:**
1. dev-core: Define ports (interfaces)
2. dev-adapter-db: Implement repository port
3. dev-api: Use repository in controllers
```

---

## Visual Refactoring Overview

### Before: Knowledge Embedded in Commands

```
┌─────────────────────────────────────┐
│         imp.md (97 lines)           │
├─────────────────────────────────────┤
│ • Workflow steps (30 lines)         │
│ • TDD knowledge (20 lines)          │
│ • Autonomy rules (25 lines)         │
│ • Agent selection (15 lines)        │
│ • Testing patterns (7 lines)        │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│      feat-imp.md (81 lines)         │
├─────────────────────────────────────┤
│ • Workflow steps (25 lines)         │
│ • TDD knowledge (18 lines)          │  ← DUPLICATE
│ • Autonomy rules (22 lines)         │  ← DUPLICATE
│ • Feature tracking (16 lines)       │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│         fix.md (60 lines)           │
├─────────────────────────────────────┤
│ • Workflow steps (20 lines)         │
│ • TDD debugging (15 lines)          │  ← Similar
│ • Autonomy rules (25 lines)         │  ← DUPLICATE
└─────────────────────────────────────┘
```

### After: Knowledge Extracted to Skills

```
Commands (Thin)                     Skills (Knowledge)
┌─────────────────┐                ┌──────────────────────┐
│  imp.md         │───references──→│ workflow-tdd         │
│  (40 lines)     │                │ (300 lines)          │
│                 │                └──────────────────────┘
│ Just workflow   │                ┌──────────────────────┐
│ steps           │───references──→│ workflow-autonomy    │
└─────────────────┘                │ (250 lines)          │
                                   └──────────────────────┘
┌─────────────────┐                ┌──────────────────────┐
│  feat-imp.md    │───references──→│ workflow-agent-      │
│  (35 lines)     │                │ delegation           │
└─────────────────┘                │ (200 lines)          │
                                   └──────────────────────┘
┌─────────────────┐                ┌──────────────────────┐
│  fix.md         │───references──→│ project-structure    │
│  (35 lines)     │                │ (150 lines)          │
└─────────────────┘                └──────────────────────┘
```

**Result:**
- Commands: 70% smaller (procedural only)
- Skills: Single source of truth
- No duplication
- Composable knowledge

---

## Migration Priority

### High Priority (Do First)
1. **workflow-autonomy** - Affects all commands, highest duplication
2. **workflow-tdd** - Core to all implementation commands
3. **Simplify imp.md** - Most used command, biggest impact

### Medium Priority (Do Second)
4. **workflow-agent-delegation** - Improves delegation clarity
5. **Split dev-core** - Large skill, high usage
6. **Simplify feat-imp.md** - Second most used command

### Low Priority (Do Later)
7. **Split dev-adapter-db** - Optimization, lower urgency
8. **workflow-spec-management** - Nice to have
9. **Split react-dev** - Project-specific

---

## Questions for Team Review

1. **Skill Granularity:** Are workflow skills too small (~200-300 lines) or just right? Target size 500 lines
2. **Command Simplicity:** Is 30-40 lines per command too thin or ideal? Good
3. **Split Criteria:** Should we split all skills >800 lines or only problematic ones?  problematic ones
4. **Naming:** Do workflow-* vs dev-* naming conventions make sense? Yes
5. **Agent Impact:** Should we create more specialized agents (dev-core-domain, dev-core-application)? No
6. **Migration Path:** Big bang or gradual rollout? Big bang
7. **Backwards Compat:** Keep old commands during transition or hard cutover?hard cutover

---

**Analysis Date:** 2025-11-16
**Analyst:** Claude Code Assistant
**Status:** Ready for Review
