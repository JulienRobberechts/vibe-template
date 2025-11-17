# Workflow: Implementation with TDD

**Purpose:** Master TDD cycle, autonomy rules, baseline verification, integration testing, and test organization for all implementation work.

**Version:** 1.0
**Lines:** ~500

---

## When to Use This Skill

Load this skill when:
- Implementing features or iterations
- Fixing bugs with failing tests
- Refactoring code with test coverage
- Any task requiring test-first development

---

## Core Principles

1. **Test First, Always:** Write failing test before implementation
2. **Verify Baseline:** Ensure existing tests pass before starting
3. **Small Steps:** Minimal code to pass each test
4. **Continuous Progress:** Work autonomously, ask only when blocked
5. **Integration Verify:** Full suite + manual tests before completion

---

## Section 1: TDD Cycle (Red-Green-Refactor)

### The Three Phases

**1. RED: Write Failing Test**

Before writing ANY production code:

```typescript
// Example: Testing a new function
describe('calculateTotal', () => {
  it('should sum item prices with tax', () => {
    const items = [
      { price: 10, quantity: 2 },
      { price: 5, quantity: 1 }
    ];
    const result = calculateTotal(items, 0.1); // 10% tax
    expect(result).toBe(27.5); // (20 + 5) * 1.1
  });
});
```

**Run the test and verify it FAILS:**
```bash
npm test -- calculateTotal.test.ts
```

Expected output: Test fails because `calculateTotal` doesn't exist yet.

**Why this matters:**
- Confirms test actually validates the behavior
- Prevents false positives (test that always passes)
- Ensures you understand the requirement

---

**2. GREEN: Minimal Implementation**

Write the SIMPLEST code that makes the test pass:

```typescript
// src/utils/calculateTotal.ts
export function calculateTotal(items: Item[], taxRate: number): number {
  const subtotal = items.reduce((sum, item) =>
    sum + (item.price * item.quantity), 0
  );
  return subtotal * (1 + taxRate);
}
```

**Run test again:**
```bash
npm test -- calculateTotal.test.ts
```

Expected output: Test passes (green).

**Guidelines:**
- Don't over-engineer (no "what if" code)
- Don't add features not tested
- Don't optimize prematurely
- Just make it work

---

**3. REFACTOR: Clean Up**

Now improve code quality WITHOUT changing behavior:

```typescript
// Refactored for clarity
export function calculateTotal(items: Item[], taxRate: number): number {
  const subtotal = calculateSubtotal(items);
  return applyTax(subtotal, taxRate);
}

function calculateSubtotal(items: Item[]): number {
  return items.reduce((sum, item) =>
    sum + (item.price * item.quantity), 0
  );
}

function applyTax(amount: number, rate: number): number {
  return amount * (1 + rate);
}
```

**Re-run tests to ensure refactor didn't break anything:**
```bash
npm test -- calculateTotal.test.ts
```

**Refactoring opportunities:**
- Extract functions for clarity
- Rename variables for readability
- Remove duplication
- Improve structure
- Add type safety

**Do NOT:**
- Change test expectations
- Add new functionality
- Skip re-running tests

---

### TDD Workflow Checklist

For EVERY feature/fix:

- [ ] Write failing test (RED)
- [ ] Run test, confirm it fails
- [ ] Write minimal implementation (GREEN)
- [ ] Run test, confirm it passes
- [ ] Refactor code (REFACTOR)
- [ ] Re-run test, confirm still passes
- [ ] Run full test suite, confirm no regressions
- [ ] Repeat for next test case

---

### Test Types by Layer

**1. Unit Tests (Domain & Application)**

Test single function/class in isolation:

```typescript
// Domain entity test
describe('Order', () => {
  it('should calculate total from items', () => {
    const order = new Order([
      new OrderItem('product1', 10, 2),
      new OrderItem('product2', 5, 1)
    ]);
    expect(order.getTotal()).toBe(25);
  });
});
```

**Characteristics:**
- No external dependencies (no DB, no API)
- Fast execution (< 100ms)
- Deterministic (same input = same output)
- Run on every change

---

**2. Integration Tests (Adapters)**

Test adapter + external system interaction:

```typescript
// Database repository test
describe('UserRepository', () => {
  let repo: UserRepository;
  let db: TestDatabase;

  beforeEach(async () => {
    db = await createTestDatabase();
    repo = new UserRepository(db);
  });

  it('should save and retrieve user', async () => {
    const user = new User('user@example.com', 'John');
    await repo.save(user);

    const retrieved = await repo.findByEmail('user@example.com');
    expect(retrieved?.name).toBe('John');
  });

  afterEach(async () => {
    await db.close();
  });
});
```

**Characteristics:**
- Uses real/test databases, APIs, file systems
- Slower execution (100ms - 1s)
- Requires setup/teardown
- Run before commits

---

**3. End-to-End Tests (Full System)**

Test complete workflows through public interface:

```typescript
// API endpoint test
describe('POST /api/orders', () => {
  it('should create order and return 201', async () => {
    const response = await request(app)
      .post('/api/orders')
      .send({
        items: [{ productId: 'p1', quantity: 2 }],
        customerId: 'c1'
      });

    expect(response.status).toBe(201);
    expect(response.body.orderId).toBeDefined();

    // Verify in database
    const order = await db.orders.findById(response.body.orderId);
    expect(order.status).toBe('pending');
  });
});
```

**Characteristics:**
- Tests full stack (API → use case → repository → DB)
- Slowest execution (1s - 10s)
- Most realistic
- Run before releases

---

## Section 2: Baseline Verification

### Before Starting ANY Work

**Always verify the baseline is GREEN:**

```bash
# 1. Run existing test suite
npm test

# 2. Check for failures
# If ANY test fails: STOP and fix before proceeding
```

**Why this matters:**
- Ensures you start from known-good state
- Prevents confusion about who broke what
- Maintains team trust in test suite

---

### If Baseline Tests Fail

**DO NOT PROCEED with your feature. Instead:**

1. **Identify the failure:**
   ```bash
   npm test 2>&1 | tee test-output.txt
   ```

2. **Determine cause:**
   - Did I break it? (uncommitted changes)
   - Was it already broken? (git stash and re-run)
   - Environment issue? (check node_modules, .env)

3. **Fix or escalate:**
   - If you broke it: fix your changes
   - If already broken: notify team, fix if quick, or skip broken tests temporarily
   - If environment: resolve setup issues

4. **Re-verify baseline:**
   ```bash
   npm test
   ```

Only proceed when baseline is GREEN.

---

### Baseline for Dev Server

**Verify dev server starts successfully:**

```bash
npm run dev
```

**Check:**
- [ ] Server starts without errors
- [ ] Port is accessible (e.g., http://localhost:3000)
- [ ] Hot reload works (edit file, see change)
- [ ] No console errors in browser (if UI)

If dev server fails, fix before coding.

---

## Section 3: Integration Testing

### Automated Integration Tests

After implementing a feature, run ALL automated tests:

```bash
# Full test suite (unit + integration + e2e)
npm test

# Watch for specific patterns if needed
npm test -- --watch --testPathPattern=order
```

**Success criteria:**
- [ ] All tests pass (0 failures)
- [ ] No skipped tests (unless intentional)
- [ ] Coverage meets threshold (if configured)
- [ ] No warnings in output

---

### Manual Integration Tests

For features with UI or complex workflows, complete ALL manual test checkboxes from the spec/iteration:

**Example from iteration checklist:**

```markdown
## Manual Tests

- [ ] Login with valid credentials → redirects to dashboard
- [ ] Login with invalid credentials → shows error message
- [ ] Logout → redirects to login page
- [ ] Session expires after 30min → shows timeout message
```

**Process:**

1. **Start fresh environment:**
   ```bash
   npm run dev
   # Clear browser cache, use incognito if needed
   ```

2. **Execute each test systematically:**
   - Follow steps exactly
   - Note actual vs expected behavior
   - Screenshot failures

3. **Document results:**
   ```markdown
   - [x] Login with valid credentials → ✅ PASS
   - [x] Login with invalid credentials → ✅ PASS
   - [ ] Logout → ❌ FAIL (redirects to /home instead of /login)
   - [ ] Session expires → ⚠️ SKIP (requires 30min wait)
   ```

4. **Fix failures before marking iteration complete**

---

### Responsive Testing (UI Features)

If feature includes UI, test on multiple viewports:

**Required viewport tests:**

- [ ] **Mobile (375x667):** Touch interactions, small screen layout
- [ ] **Tablet (768x1024):** Medium screen, portrait/landscape
- [ ] **Desktop (1920x1080):** Full layout, hover states

**Tools:**
- Browser DevTools (F12 → Device Toolbar)
- BrowserStack (for real devices)
- Responsive design mode

**Check for:**
- Text readability (font size, line height)
- Button tap targets (min 44x44px)
- Image scaling (no pixelation)
- Scroll behavior (no horizontal scroll)
- Layout shifts (stable CLS)

---

### Failure Handling

**If ANY test fails (automated or manual):**

1. **STOP deployment/commit**
2. **Investigate failure:**
   - Read error message carefully
   - Check logs (browser console, server logs)
   - Reproduce failure manually
3. **Fix the issue:**
   - Debug with TDD (write test, fix, verify)
4. **Re-run FULL test suite:**
   - Don't assume "only that test" is affected
5. **Repeat until all green**

**Never:**
- Skip failing tests to move forward
- Commit with known test failures
- Assume "it will be fixed later"

---

## Section 4: Test Organization

### File Structure

**Convention: Tests live next to source code**

```
src/
  domain/
    entities/
      Order.ts
      __tests__/
        Order.test.ts
    value-objects/
      Money.ts
      __tests__/
        Money.test.ts
  application/
    use-cases/
      CreateOrder.ts
      __tests__/
        CreateOrder.test.ts
  infrastructure/
    repositories/
      OrderRepository.ts
      __tests__/
        OrderRepository.integration.test.ts
```

**Naming:**
- Unit tests: `*.test.ts`
- Integration tests: `*.integration.test.ts`
- E2E tests: `*.e2e.test.ts`

---

### Test Structure: AAA Pattern

**Arrange-Act-Assert** for clarity:

```typescript
describe('Order', () => {
  it('should apply discount to total', () => {
    // ARRANGE: Set up test data
    const order = new Order([
      new OrderItem('product1', 100, 1)
    ]);
    const discount = new Discount(0.1); // 10% off

    // ACT: Perform the operation
    order.applyDiscount(discount);

    // ASSERT: Verify the outcome
    expect(order.getTotal()).toBe(90);
  });
});
```

**Benefits:**
- Easy to read and understand
- Clear separation of setup, action, verification
- Helps identify test logic issues

---

### Test Naming Conventions

**Pattern:** `should [expected behavior] when [condition]`

```typescript
// Good
it('should return null when user not found')
it('should throw error when email is invalid')
it('should calculate discount when coupon is valid')

// Bad
it('test user retrieval')
it('checks email')
it('discount')
```

**Describe blocks for context:**

```typescript
describe('UserService', () => {
  describe('register', () => {
    it('should create user when email is unique')
    it('should throw error when email exists')
  });

  describe('authenticate', () => {
    it('should return token when credentials valid')
    it('should throw error when credentials invalid')
  });
});
```

---

### Performance: Running Single Tests

**Don't run the full suite on every change:**

```bash
# Run single test file
npm test -- Order.test.ts

# Run tests matching pattern
npm test -- --testNamePattern="should apply discount"

# Run in watch mode for TDD
npm test -- --watch Order.test.ts
```

**When to run full suite:**
- Before committing
- After refactoring shared code
- Before marking iteration complete
- Before creating PR

---

## Section 5: Autonomy Rules

### Work Continuously Without Stopping

**You should work autonomously through:**
- Writing tests
- Implementing features
- Refactoring code
- Running tests
- Fixing minor issues
- Making reasonable decisions

**Make forward progress on every execution.**

---

### When to Ask Questions

**ONLY stop and ask when:**

1. **Blocked by missing information:**
   - Spec is fundamentally unclear
   - Ambiguous requirement (can't determine correct behavior)
   - Conflicting requirements

2. **Blocked by external dependency:**
   - Missing API key or credentials
   - Package not available
   - Environment configuration issue

3. **Critical error preventing progress:**
   - Build system broken
   - Test framework not working
   - Database connection failing

---

### When NOT to Ask

**Do NOT stop or ask questions about:**

1. **Implementation details:**
   - How to structure a class
   - Which function to use
   - Code organization choices

   **Instead:** Use best practices and existing patterns

2. **Code style choices:**
   - Variable naming
   - Function placement
   - Comment style

   **Instead:** Follow existing codebase conventions

3. **Minor architectural decisions:**
   - Where to put a utility function
   - Whether to create a new file
   - How to organize imports

   **Instead:** Apply hexagonal architecture principles

4. **Test naming or structure:**
   - How to name test cases
   - Where to put test files
   - AAA pattern usage

   **Instead:** Follow test organization section above

5. **Variable/function naming:**
   - What to call a variable
   - How to name a function
   - Class naming

   **Instead:** Choose clear, descriptive names

---

### Decision Framework

**When uncertain, use this framework:**

1. **Check existing code:**
   - How is similar functionality handled?
   - What patterns are already used?
   - Is there a convention to follow?

2. **Apply best practices:**
   - SOLID principles
   - Hexagonal architecture
   - Test-driven development
   - Clean code principles

3. **Choose the simpler option:**
   - Fewer dependencies
   - Less abstraction
   - Easier to test
   - Clearer intent

4. **Proceed with confidence:**
   - Make the decision
   - Implement it
   - Move forward
   - Can refactor later if needed

---

### Question Quality Guidelines

**If you must ask, ask ONE specific question:**

**Good questions:**
```
"The spec says 'notify user' but doesn't specify email vs SMS.
Which notification method should I implement?"

"The API key for Stripe isn't in .env.example.
Where can I get the test API key?"
```

**Bad questions:**
```
"How should I implement this?" (too vague)

"Should I use class or function?
And what should I name it?
And where should it go?" (multiple questions)

"Is this the right approach?" (decision you should make)
```

---

### While Blocked: Continue Unblocked Work

**If you're waiting for an answer:**

1. **Identify unblocked tasks:**
   - Other todos in the list
   - Related tests you can write
   - Documentation you can update

2. **Work on those instead:**
   - Keep making progress
   - Don't sit idle
   - Maximize productivity

3. **Return to blocked task when unblocked:**
   - User provides answer
   - You find solution yourself
   - Workaround becomes apparent

---

## Workflow Integration

### How Commands Use This Skill

Commands load this skill and reference specific sections:

```markdown
# Example: imp.md command

**Skills Required:**
- workflow-implementation: TDD cycle, autonomy rules

## Workflow

1. **Plan** (if no active todos)
   - Use TodoWrite: create tasks from checklist

2. **Implement** (apply workflow-implementation)
   - Follow TDD cycle (Section 1)
   - Work autonomously (Section 5)
   - Verify integration (Section 3)
```

This keeps command files SHORT (30-40 lines), while this skill provides depth.

---

## Quick Reference

**TDD Cycle:**
1. Write failing test (RED)
2. Minimal implementation (GREEN)
3. Refactor (REFACTOR)
4. Repeat

**Before starting:**
- [ ] Baseline tests pass (`npm test`)
- [ ] Dev server works (`npm run dev`)

**Before completing:**
- [ ] All automated tests pass
- [ ] All manual tests completed
- [ ] Responsive tested (if UI)

**Autonomy:**
- Work continuously
- Ask only when blocked
- Make reasonable decisions
- Follow existing patterns

---

**End of Skill: workflow-implementation**
