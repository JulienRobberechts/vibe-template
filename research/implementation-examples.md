# Implementation Examples: Command & Skill Refactor

## Example 1: Refactoring imp.md

### Before (97 lines)

```markdown
# Implementation

Implement iteration from `docs/implementation.md` using TDD. Work autonomously.

## Workflow

1. **Check State & Resume**
   - Read `docs/implementation.md`: identify current iteration status
   - Check git status: what's uncommitted/in-progress
   - Read existing todos (if any): resume from last in_progress or pending
   - If iteration marked COMPLETED: move to next iteration
   - If mid-iteration: continue from where stopped

2. **Read Spec** (if starting new iteration)
   - Read `docs/implementation.md` for current iteration
   - Extract objectives, tasks, deliverables
   - Note manual test checkboxes
   - Identify appropriate specialized agent: dev-core (domain/use-cases),
     dev-api (REST endpoints), dev-adapter-db (repositories), dev-i-cli (CLI),
     react-dev (web UI), react-native-dev (mobile)

3. **Delegate** (if suitable specialized agent exists)
   - Use Task tool with matching agent type from above
   - Provide iteration details and let agent work autonomously
   - If no match: proceed to Plan step

4. **Plan** (if no active todos)
   - Use TodoWrite: create tasks from checklist
   - Break down into subtasks
   - Only list blocking questions

5. **Verify Baseline**
   - Run existing tests: `npm test`
   - If fails: fix before proceeding
   - Run dev: `npm run dev`

6. **TDD Cycle** (repeat until all todos completed)
   - **Test**: Write failing test
   - **Implement**: Minimal code to pass
   - **Refactor**: Clean up
   - **Verify**: Run all tests (automated + manual from spec)
   - Mark todo completed
   - **Continue**: Move to next todo

7. **Integration Test**
   - Run full test suite: `npm test`
   - Run dev server: `npm run dev`
   - Complete ALL manual tests from iteration checklist
   - Test responsive (mobile + desktop) if UI
   - If any fail: fix and repeat

8. **Document**
   - Update `docs/implementation.md`:
     - Mark iteration ✅ COMPLETED
     - Add timestamp: "Completed: YYYY-MM-DD"
     - Note deviations from plan
   - Update README.md

9. **Commit**
   - Stage changes
   - Commit: "feat: iteration $1 - [brief desc]"
   - Add changeset if needed

## Autonomy Rules

**Work continuously without stopping unless:**
- Blocked by missing information
- Blocked by external dependency
- Critical error that prevents progress

**Do NOT stop or ask questions for:**
- Implementation details
- Code style choices
- Minor architectural decisions
- Test naming or structure
- Variable/function naming

**When blocked:**
- Try to resolve independently first
- If still blocked: ask ONE specific question
- Continue with unblocked tasks while waiting

## Output

- What was built (1-2 sentences)
- Files created/modified
- Manual test results
- Next iteration recommendation
```

---

### After (40 lines)

```markdown
# Implementation

Implement iteration from `docs/implementation.md` using TDD. Work autonomously.

**Skills:**
- workflow-tdd: TDD cycle, baseline verification, integration testing
- workflow-autonomy: When to ask questions, when to work independently
- workflow-agent-delegation: Agent selection and task delegation
- project-structure: File locations, status tracking
- git-workflow: Commit patterns, changeset creation

## Workflow

1. **Check State**
   - Read `docs/implementation.md` current iteration status
   - Check git status + existing todos
   - Resume from last in_progress or move to next iteration if completed

2. **Delegate to Agent** (if iteration matches specialized agent)
   - Use workflow-agent-delegation to identify matching agent
   - Invoke Task tool with iteration details
   - Let agent work autonomously
   - Skip to Document step when done

3. **Plan** (if no matching agent or no todos exist)
   - TodoWrite: create tasks from iteration checklist
   - List only blocking questions (use workflow-autonomy criteria)

4. **TDD Implementation**
   - Follow workflow-tdd baseline verification
   - Execute TDD cycle for each todo
   - Apply workflow-autonomy rules continuously
   - Mark todos completed immediately after each

5. **Document & Commit**
   - Update docs/implementation.md per project-structure
   - Commit using git-workflow patterns
   - Add changeset if needed

## Output

Per project-structure: what built, files modified, manual tests, next iteration
```

**Reduction:** 97 → 40 lines (59% smaller)

**Benefits:**
- Workflow steps clearer (procedural only)
- Knowledge lives in reusable skills
- Easier to scan and understand
- No duplication with other commands

---

## Example 2: Creating workflow-tdd Skill

### New File: .claude/skills/workflow-tdd/SKILL.md

```markdown
---
name: workflow-tdd
description: Test-Driven Development workflow patterns for implementing features with confidence
version: 1.0.0
---

# Test-Driven Development Workflow

You are using TDD to implement features with high quality and confidence. This skill provides the patterns and practices for effective TDD workflows.

## Core Principles

**Test-First Development:**
- Write test BEFORE implementation
- Red → Green → Refactor cycle
- Never write production code without a failing test
- Keep cycles small (minutes, not hours)

**Quality Gates:**
- All tests pass before moving forward
- Baseline must be clean before starting
- Integration tests validate full system
- Manual tests verify user experience

## Baseline Verification

Before starting ANY implementation:

### 1. Run Automated Tests
```bash
npm test
```

**Expected:** All tests pass
**If fails:**
- Read failure output
- Fix broken tests first
- Do NOT proceed until green

### 2. Start Development Server
```bash
npm run dev
```

**Expected:** Server starts without errors
**If fails:**
- Check compilation errors
- Resolve dependency issues
- Ensure clean startup

### 3. Verify Working State
- Application loads in browser
- No console errors
- Basic functionality works
- Database connections healthy

**If ANY baseline check fails:** Fix immediately. Never build on broken foundation.

---

## Red-Green-Refactor Cycle

### Phase 1: Red (Write Failing Test)

**Steps:**
1. Identify next smallest behavior to implement
2. Write test that describes expected behavior
3. Run test → verify it FAILS
4. Confirm failure is for right reason

**Example:**
```typescript
// 1. Write test for behavior
describe('User.suspend()', () => {
  it('should suspend an active user', () => {
    const user = User.create(Email.create('test@example.com'));

    user.suspend();

    expect(user.status.isSuspended()).toBe(true);
  });
});

// 2. Run test → RED (fails because suspend() doesn't exist)
```

**Red Phase Checklist:**
- [ ] Test is clear and focused
- [ ] Test fails when run
- [ ] Failure message makes sense
- [ ] Only ONE new behavior tested

### Phase 2: Green (Make Test Pass)

**Steps:**
1. Write MINIMAL code to pass test
2. Take shortcuts if needed (refactor later)
3. Run test → verify it PASSES
4. Run ALL tests → verify nothing broke

**Example:**
```typescript
// Minimal implementation
suspend(): void {
  this._status = UserStatus.suspended();
}
```

**Green Phase Rules:**
- Write minimal code (no gold-plating)
- Don't handle edge cases yet (unless tested)
- Duplication is OK (refactor next)
- Only make THIS test pass

**Green Phase Checklist:**
- [ ] New test passes
- [ ] All other tests still pass
- [ ] No compilation errors
- [ ] Code is "good enough" for now

### Phase 3: Refactor (Improve Code)

**Steps:**
1. Look for duplication
2. Extract methods/classes
3. Improve naming
4. Run tests after EACH change
5. Keep tests green throughout

**Example:**
```typescript
// Before refactor
suspend(): void {
  this._status = UserStatus.suspended();
}

activate(): void {
  this._status = UserStatus.active();
}

// After refactor (remove duplication)
private transitionTo(status: UserStatus): void {
  this._status = status;
}

suspend(): void {
  this.transitionTo(UserStatus.suspended());
}

activate(): void {
  this.transitionTo(UserStatus.active());
}
```

**Refactor Phase Rules:**
- Only refactor when tests are GREEN
- Run tests after each refactor
- Never change behavior (only structure)
- Improve design incrementally

**Refactor Phase Checklist:**
- [ ] Code is cleaner than before
- [ ] No duplication
- [ ] Good naming
- [ ] All tests still pass
- [ ] No new functionality added

---

## Test Types & Levels

### Unit Tests (Domain & Application)

**What:** Pure business logic in isolation
**Location:** Same directory as source
**Speed:** Milliseconds

**Example:**
```typescript
// domain/entities/user.entity.test.ts

import { User } from './user.entity';
import { Email } from '../value-objects/email.vo';

describe('User Entity', () => {
  describe('suspend()', () => {
    it('suspends active user', () => {
      const user = User.create(Email.create('test@example.com'));
      user.suspend();
      expect(user.status.isSuspended()).toBe(true);
    });

    it('throws when already suspended', () => {
      const user = User.create(Email.create('test@example.com'));
      user.suspend();
      expect(() => user.suspend()).toThrow();
    });
  });
});
```

**When:** Every domain entity, value object, use case

### Integration Tests (Adapters)

**What:** External interactions (database, APIs, files)
**Location:** Adapter directories
**Speed:** Seconds

**Example:**
```typescript
// infrastructure/adapters/repositories/prisma-user.repository.test.ts

describe('PrismaUserRepository', () => {
  let repository: PrismaUserRepository;

  beforeEach(async () => {
    await cleanDatabase();
    repository = new PrismaUserRepository(prisma);
  });

  it('saves and retrieves user', async () => {
    const user = User.create(Email.create('test@example.com'));

    await repository.save(user);
    const retrieved = await repository.findById(user.id);

    expect(retrieved?.email.getValue()).toBe('test@example.com');
  });
});
```

**When:** Every adapter implementation

### E2E Tests (Full System)

**What:** User workflows through real system
**Location:** e2e/ or tests/ directory
**Speed:** Seconds to minutes

**Example:**
```typescript
describe('User Registration Flow', () => {
  it('registers new user end-to-end', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', password: 'SecurePass123!' });

    expect(response.status).toBe(201);
    expect(response.body.data.email).toBe('test@example.com');

    // Verify in database
    const user = await db.users.findByEmail('test@example.com');
    expect(user).toBeDefined();
  });
});
```

**When:** Critical user workflows

---

## Integration Testing Checklist

After implementing ALL features in an iteration:

### Automated Tests
```bash
npm test
```

**Verify:**
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] All E2E tests pass
- [ ] No warnings or errors
- [ ] Test coverage maintained/improved

### Development Server
```bash
npm run dev
```

**Verify:**
- [ ] Server starts cleanly
- [ ] No compilation errors
- [ ] No runtime warnings
- [ ] Database migrations applied
- [ ] All routes accessible

### Manual Tests (from Spec)

Execute EVERY manual test in iteration spec:

**For UI Features:**
- [ ] Complete all checkbox items in spec
- [ ] Test on desktop viewport
- [ ] Test on mobile viewport
- [ ] Test on tablet (if applicable)
- [ ] Verify responsive behavior
- [ ] Check accessibility (keyboard, screen reader)

**For API Features:**
- [ ] Test all endpoints with valid data
- [ ] Test error cases (400, 401, 404, 500)
- [ ] Verify response formats
- [ ] Check status codes
- [ ] Test edge cases from spec

**For Domain Features:**
- [ ] Exercise all business rules
- [ ] Test edge cases
- [ ] Verify exceptions thrown correctly
- [ ] Check invariants maintained

### If ANY Test Fails

**Do NOT proceed to commit:**
1. Investigate failure
2. Write automated test to catch it (if not caught)
3. Fix issue
4. Re-run full suite
5. Repeat until ALL green

**Only commit when:**
- ✅ All automated tests pass
- ✅ All manual tests pass
- ✅ No errors in console
- ✅ Code compiles cleanly

---

## TDD for Debugging (Fixing Bugs)

When fixing bugs, TDD process adapts:

### 1. Reproduce Bug with Test

Write test that FAILS due to bug:

```typescript
it('should handle empty email gracefully', () => {
  // This test will fail if bug exists
  expect(() => Email.create('')).toThrow('Email cannot be empty');
});
```

**Run test → Verify it FAILS**

### 2. Fix Minimal Code

Add minimal code to pass test:

```typescript
static create(email: string): Email {
  if (!email.trim()) {
    throw new DomainException('Email cannot be empty');
  }
  return new Email(email.toLowerCase().trim());
}
```

**Run test → Verify it PASSES**

### 3. Add Edge Case Tests

Expand test coverage:

```typescript
describe('Email.create() edge cases', () => {
  it('throws on empty string', () => {
    expect(() => Email.create('')).toThrow();
  });

  it('throws on whitespace only', () => {
    expect(() => Email.create('   ')).toThrow();
  });

  it('trims whitespace', () => {
    const email = Email.create('  test@example.com  ');
    expect(email.getValue()).toBe('test@example.com');
  });
});
```

### 4. Verify Bug Fixed System-Wide

- Run full test suite
- Test manually in UI
- Check related functionality
- Verify no regressions

---

## Test Organization

### File Structure
```
src/
├── domain/
│   ├── entities/
│   │   ├── user.entity.ts
│   │   └── user.entity.test.ts       ← Same directory
│   └── value-objects/
│       ├── email.vo.ts
│       └── email.vo.test.ts
└── application/
    └── use-cases/
        ├── register-user.use-case.ts
        └── register-user.use-case.test.ts
```

### Naming Convention
- Source: `user.entity.ts`
- Test: `user.entity.test.ts`
- Pattern: `{filename}.test.ts`

### Test Structure (AAA Pattern)

```typescript
describe('Feature or Class', () => {
  describe('method()', () => {
    it('should do expected behavior', () => {
      // Arrange - Set up test data
      const user = User.create(Email.create('test@example.com'));

      // Act - Execute behavior
      user.suspend();

      // Assert - Verify outcome
      expect(user.status.isSuspended()).toBe(true);
    });
  });
});
```

---

## Performance Considerations

### Run Single Tests During Development

```bash
# Run single file
npm test user.entity.test.ts

# Run single test
npm test user.entity.test.ts -t "should suspend active user"
```

**Why:** Faster feedback loop during TDD cycles

### Run Full Suite Before Commit

```bash
npm test
```

**Why:** Catch regressions, ensure system health

---

## TDD Workflow Summary

**Every Feature:**
1. Verify baseline (tests + dev server)
2. Red: Write failing test
3. Green: Minimal code to pass
4. Refactor: Improve design
5. Repeat until feature complete
6. Integration test (all tests + manual)
7. Commit only when ALL green

**Every Bug Fix:**
1. Write test reproducing bug
2. Verify test fails
3. Fix with minimal code
4. Verify test passes
5. Add edge case tests
6. Run full suite

**Key Principle:** Tests guide development. Never write code without a test demanding it.

---

**Skill Version:** 1.0.0
**Last Updated:** 2025-11-16
```

**Size:** ~300 lines of reusable knowledge

**Used By:**
- imp.md
- feat-imp.md
- fix.md
- Any future implementation command

---

## Example 3: Before/After Command Comparison Matrix

| Command | Before | After | Reduction | Skills Referenced |
|---------|--------|-------|-----------|-------------------|
| imp.md | 97 lines | 40 lines | 59% | workflow-tdd, workflow-autonomy, workflow-agent-delegation, project-structure, git-workflow |
| feat-imp.md | 81 lines | 35 lines | 57% | workflow-tdd, workflow-autonomy, project-structure, git-workflow |
| fix.md | 60 lines | 35 lines | 42% | workflow-tdd, workflow-autonomy, git-workflow |
| plan.md | 58 lines | 30 lines | 48% | workflow-spec-management, workflow-agent-delegation, backend-architect |
| feat-refine.md | 64 lines | 35 lines | 45% | workflow-spec-management, workflow-autonomy |
| **Total** | **360 lines** | **175 lines** | **51%** | **6 new skills (~1400 lines)** |

**Net Change:**
- Commands: 360 → 175 lines (-185 lines, -51%)
- New Skills: 0 → 1400 lines (+1400 lines)
- Total: +1215 lines BUT knowledge is centralized and reusable

**Value:**
- Zero duplication in commands
- Skills reusable across projects
- Easier to update (change once, affects all)
- Commands much easier to read/understand

---

## Example 4: Agent Configuration Changes

### Before

```yaml
# .claude/agents/dev-core.md
---
name: dev-core
description: Use this agent to develop domain and application layers...
model: sonnet
color: yellow
skills:
  - dev-core
---

You are a specialized core domain developer. Use the dev-core skill to
implement pure business logic in domain and application layers.
```

**Issues:**
- Single monolithic skill (1394 lines)
- No workflow guidance
- No TDD patterns

---

### After

```yaml
# .claude/agents/dev-core.md
---
name: dev-core
description: Use this agent to develop domain and application layers...
model: sonnet
color: yellow
skills:
  - dev-core-domain          # Domain layer expertise
  - dev-core-application     # Application layer expertise
  - workflow-tdd             # TDD practices
  - workflow-autonomy        # Decision making
---

You are a specialized core domain developer implementing hexagonal
architecture core layers.

**Domain Work:** Use dev-core-domain skill
**Application Work:** Use dev-core-application skill
**Development Process:** Follow workflow-tdd patterns
**Autonomy:** Apply workflow-autonomy rules

Always write tests first. Define ports for all external dependencies.
```

**Benefits:**
- Focused skills (700 + 600 lines vs 1394)
- Workflow guidance included
- Composable skills
- Clear separation of concerns

---

## Example 5: Multi-Technology Skill Usage

### Before: dev-adapter-db (1637 lines)

User working with Prisma loads entire skill:
- Prisma patterns (400 lines) ✓ Needed
- Drizzle patterns (400 lines) ✗ Unused
- Raw SQL patterns (237 lines) ✗ Unused
- Knex patterns (200 lines) ✗ Unused
- Core repository (400 lines) ✓ Needed

**Result:** 800 lines needed, 837 lines wasted (51% overhead)

---

### After: Split by Technology

```yaml
# .claude/agents/dev-adapter-db-prisma.md
---
name: dev-adapter-db-prisma
description: Database adapter using Prisma ORM
model: sonnet
color: blue
skills:
  - dev-adapter-db-core      # 600 lines - Repository patterns
  - dev-adapter-db-prisma    # 400 lines - Prisma-specific
  - workflow-tdd             # 300 lines - Testing patterns
---

You implement database repositories using Prisma ORM in hexagonal architecture.

**Always reference:** dev-adapter-db-core for repository patterns
**Prisma-specific:** Use dev-adapter-db-prisma for ORM details
**Testing:** Follow workflow-tdd for database tests
```

**Result:**
- Core patterns (600 lines) ✓ Loaded
- Prisma patterns (400 lines) ✓ Loaded
- Drizzle patterns (400 lines) ✗ Not loaded
- Other ORMs (437 lines) ✗ Not loaded

**Loaded:** 1000 lines (vs 1637 lines)
**Overhead:** 0% (all relevant)
**Savings:** 39% reduction in loaded context

---

## Example 6: Skill Composition Pattern

### Scenario: Implementing User Registration API

**Needed Knowledge:**
1. Domain entity (User)
2. Use case (RegisterUser)
3. REST API endpoint
4. Request validation
5. TDD workflow

---

### Before: Multiple Large Skills

```
Load: dev-core (1394 lines) - Entire domain+application knowledge
Load: dev-api (701 lines) - Entire API knowledge
Total: 2095 lines

Actual usage:
- Domain entities: 200 lines used
- Use cases: 150 lines used
- REST endpoints: 100 lines used
- Validation: 50 lines used
Waste: 2095 - 500 = 1595 lines (76% overhead)
```

---

### After: Focused Skills

```yaml
# Agent for this specific task
---
name: user-registration-developer
skills:
  - dev-core-domain          # 700 lines - Just domain patterns
  - dev-core-application     # 600 lines - Just use case patterns
  - dev-api                  # 701 lines - API patterns
  - workflow-tdd             # 300 lines - TDD workflow
---

Total: 2301 lines (slightly more)
BUT: All 2301 lines are relevant to user registration
Overhead: ~15% (vs 76%)
```

**Better:** Create task-specific agent on demand:

```yaml
---
name: registration-flow-dev
description: Implements user registration flow (domain → use case → API)
skills:
  - dev-core-domain          # Entities & value objects
  - dev-core-application     # Use cases
  - dev-api                  # REST endpoints
  - workflow-tdd             # Testing
---
```

---

## Key Takeaways

### Commands Should Be:
✓ Procedural (steps to follow)
✓ Workflow-focused (what to do, in what order)
✓ Thin (30-50 lines)
✓ Reference skills for knowledge

✗ Not knowledge repositories
✗ Not duplicating patterns
✗ Not embedding best practices

### Skills Should Be:
✓ Knowledge-focused (how to do things)
✓ Reusable across commands
✓ Single responsibility
✓ Right-sized (200-800 lines)
✓ Technology or domain specific

✗ Not workflow instructions
✗ Not overly broad
✗ Not mixing unrelated concerns

### Agents Should:
✓ Reference multiple focused skills
✓ Combine skills for specific roles
✓ Keep skills composable
✓ Load only what's needed

✗ Not load monolithic skills
✗ Not duplicate skill content
✗ Not hardcode knowledge

---

**Document Version:** 1.0
**Date:** 2025-11-16
**Purpose:** Implementation Guide for Refactoring
