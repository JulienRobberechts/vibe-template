# Precise Implementation Plan - Claude Code Refactor

## Executive Summary

**Approach:** Big bang refactor with hard cutover
**Timeline:** 1 week intensive implementation
**Target:** Zero duplication, maintainable command/skill structure
**Risk:** High (big bang) - mitigated by comprehensive testing

---

## User Decisions Applied

✅ **Skill Granularity:** Target 500 lines (merge smaller workflow skills)
✅ **Command Simplicity:** 30-40 lines per command
✅ **Split Criteria:** Only problematic skills (dev-core, dev-adapter-db)
✅ **Naming:** workflow-* vs dev-* conventions
✅ **Agent Strategy:** Hybrid (some specialized, some composite)
✅ **Migration:** Big bang (all at once)
✅ **Backwards Compat:** Hard cutover (delete old immediately)

---

## Phase 1: Create Workflow Skills (Day 1-2)

### Adjusted Skill Sizes (Target: ~500 lines each)

#### 1.1 workflow-implementation (500 lines)
**Merged:** workflow-tdd + workflow-autonomy

**Content:**
- TDD cycle (red-green-refactor) - 200 lines
- Baseline verification - 50 lines
- Integration testing - 100 lines
- Test organization - 50 lines
- Autonomy rules (when to ask vs proceed) - 100 lines

**Files to create:**
```bash
.claude/skills/workflow-implementation/SKILL.md
```

**Extract from:**
- imp.md lines 31-42 (TDD cycle)
- imp.md lines 64-82 (Autonomy rules)
- feat-imp.md lines 24-29, 49-65 (duplicates)
- fix.md lines 17-35, 40-56 (duplicates)

---

#### 1.2 workflow-project (500 lines)
**Merged:** workflow-spec-management + project-structure + git-workflow

**Content:**
- Spec templates & management - 200 lines
- Iteration tracking - 100 lines
- Project structure conventions - 100 lines
- Git workflow (commit, changeset) - 100 lines

**Files to create:**
```bash
.claude/skills/workflow-project/SKILL.md
```

**Extract from:**
- plan.md lines 17-54 (spec templates)
- feat-refine.md lines 18-42 (approach research)
- imp.md lines 7-12, 59-69 (state tracking, documentation)
- All commands (git patterns)

---

#### 1.3 workflow-agent-delegation (500 lines)

**Content:**
- Agent responsibility matrix - 100 lines
- Task → Agent mapping - 150 lines
- Delegation patterns - 100 lines
- Parallel vs sequential - 100 lines
- Task tool invocation templates - 50 lines

**Files to create:**
```bash
.claude/skills/workflow-agent-delegation/SKILL.md
```

**Extract from:**
- imp.md lines 18-23 (agent selection)
- plan.md lines 10-12 (architectural planning)

---

### Total Workflow Skills: 3 (not 6)
- workflow-implementation (~500 lines)
- workflow-project (~500 lines)
- workflow-agent-delegation (~500 lines)

**Total new lines:** ~1500 (vs original 1400 for 6 skills)

---

## Phase 2: Split Large Skills (Day 3)

### 2.1 Split dev-core (1394 → 700 + 600 = 1300 lines)

**Action:** Split into domain + application, NO coordinator

**Create:**
```bash
.claude/skills/dev-core-domain/SKILL.md       (700 lines)
.claude/skills/dev-core-application/SKILL.md  (600 lines)
```

**Content Distribution:**

**dev-core-domain (700 lines):**
- Lines 1-53: Header & Scope
- Lines 54-276: Functional IDs
- Lines 283-399: Ubiquitous Language
- Lines 400-514: Entities
- Lines 516-637: Value Objects
- Lines 639-678: Domain Services
- Lines 680-702: Domain Exceptions

**dev-core-application (600 lines):**
- Lines 1-53: Header & Scope (adapted)
- Lines 704-722: Repository Interfaces
- Lines 724-817: Use Cases
- Lines 819-847: DTOs
- Lines 849-1113: Ports Design (CRITICAL SECTION)
- Lines 1115-1154: Application Services
- Lines 1156-1167: Application Exceptions
- Lines 1169-1300: Testing Strategy (use cases)
- Lines 1302-1347: Folder Structure

**Delete:**
```bash
.claude/skills/dev-core/SKILL.md  # Original monolith
```

**Update agents:**
- Keep `dev-core` agent, update to load both skills
- Create `dev-core-domain` agent (domain-only work) - HYBRID STRATEGY
- Create `dev-core-application` agent (use cases only) - HYBRID STRATEGY

---

### 2.2 Split dev-adapter-db (1637 → 600 + 400 + 400 + 237 = 1637 lines)

**Action:** Split by technology

**Create:**
```bash
.claude/skills/dev-adapter-db-core/SKILL.md     (600 lines)
.claude/skills/dev-adapter-db-prisma/SKILL.md   (400 lines)
.claude/skills/dev-adapter-db-drizzle/SKILL.md  (400 lines)
.claude/skills/dev-adapter-db-raw/SKILL.md      (237 lines)
```

**Content Distribution:**
- **dev-adapter-db-core:** Repository pattern, entity mapping, transactions, testing
- **dev-adapter-db-prisma:** Prisma client, schema, migrations
- **dev-adapter-db-drizzle:** Drizzle ORM, query building
- **dev-adapter-db-raw:** Raw SQL, Knex, connection pooling

**Delete:**
```bash
.claude/skills/dev-adapter-db/SKILL.md  # Original monolith
```

**Update agents:**
- Keep `dev-adapter-db` agent, load core + detect tech
- Optionally create `dev-adapter-db-prisma` agent - HYBRID STRATEGY

---

### 2.3 Do NOT Split Other Skills

**Rationale:** User chose "only problematic ones"

**Keep as-is:**
- react-dev (859 lines) - Below problematic threshold
- react-native-dev (359 lines)
- dev-api (701 lines)
- dev-i-cli (1155 lines) - Consider future split if problematic
- dev-adapter-api-client (903 lines) - Consider future split if problematic
- review-clean-code (600 lines)
- backend-architect (135 lines)

---

## Phase 3: Refactor Commands (Day 4)

### 3.1 imp.md (97 → 35 lines)

**Create:** `.claude/commands/imp.md.new`

**Content:**
```markdown
# Implementation

Implement iteration from `docs/implementation.md` using TDD. Work autonomously.

**Skills Required:**
- workflow-implementation: TDD cycle, autonomy rules
- workflow-project: Spec tracking, git workflow
- workflow-agent-delegation: Agent selection

## Workflow

1. **Resume State**
   - Read `docs/implementation.md` iteration status
   - Check git status + todos
   - Resume from last in_progress or next iteration

2. **Delegate** (if iteration matches specialized agent)
   - Use workflow-agent-delegation skill
   - Invoke Task tool with iteration context
   - Skip to step 5 when done

3. **Plan** (if no delegation or no todos)
   - TodoWrite from iteration checklist
   - Apply workflow-implementation autonomy rules

4. **Implement**
   - Follow workflow-implementation TDD patterns
   - Mark todos completed immediately

5. **Complete**
   - Update docs/implementation.md (workflow-project)
   - Commit using workflow-project git patterns
   - Add changeset if needed

## Output
What built, files changed, manual tests, next iteration.
```

**Replace:** Delete old `imp.md`, rename `imp.md.new` → `imp.md`

---

### 3.2 feat-imp.md (81 → 35 lines)

Similar structure to imp.md but reads from `/backlog/*.md`

---

### 3.3 fix.md (60 → 35 lines)

Similar structure focusing on TDD debugging workflow

---

### 3.4 plan.md (58 → 30 lines)

**Content:**
```markdown
# Plan Implementation

Create implementation plan in `/docs/implementation.md`.

**Skills Required:**
- workflow-project: Spec management, iteration format
- workflow-agent-delegation: When to use backend-architect

## Workflow

1. **Read Context**
   - Read `/specs/business-specs.md`
   - Read `/specs/tech-specs.md`
   - Explore codebase structure (Task tool w/ Explore agent)

2. **Choose Approach**
   - If architectural (system design, folder structure) → Use backend-architect agent
   - If feature-specific → Plan directly

3. **Create Plan**
   - Follow workflow-project iteration format
   - Use ExitPlanMode tool to present

## Output
Implementation plan written to /docs/implementation.md
```

---

### 3.5 feat-refine.md (64 → 35 lines)

Focus on spec refinement using workflow-project patterns

---

### 3.6 spec-create.md (17 → 15 lines)

Minimal change, reference workflow-project

---

### 3.7 Other Commands

Update as needed to reference new skills

---

## Phase 4: Update Agents (Day 5)

### 4.1 Update Existing Agents

**dev-core.md:**
```yaml
---
name: dev-core
description: Develop domain and application layers of hexagonal architecture
model: sonnet
color: yellow
skills:
  - dev-core-domain
  - dev-core-application
  - workflow-implementation
---

You develop pure business logic. Use dev-core-domain for entities/value objects,
dev-core-application for use cases/ports. Follow workflow-implementation TDD.
```

**dev-adapter-db.md:**
```yaml
---
name: dev-adapter-db
description: Database output adapters in hexagonal architecture
model: sonnet
color: blue
skills:
  - dev-adapter-db-core
  - dev-adapter-db-prisma  # Default, detect from package.json
  - workflow-implementation
---

You implement database repositories. Use dev-adapter-db-core for patterns,
tech-specific skill for ORM details. Follow workflow-implementation TDD.
```

---

### 4.2 Create New Specialized Agents (Hybrid Strategy)

**dev-core-domain.md:** (NEW)
```yaml
---
name: dev-core-domain
description: Domain layer development only - entities, value objects, domain services
model: sonnet
color: yellow
skills:
  - dev-core-domain
  - workflow-implementation
---

You develop ONLY domain layer. Focus on pure business logic with zero
infrastructure dependencies. Follow workflow-implementation TDD.
```

**dev-core-application.md:** (NEW)
```yaml
---
name: dev-core-application
description: Application layer development only - use cases, ports, DTOs
model: sonnet
color: yellow
skills:
  - dev-core-application
  - workflow-implementation
---

You develop ONLY application layer. Create use cases that orchestrate domain,
define ports for adapters. Follow workflow-implementation TDD.
```

**dev-adapter-db-prisma.md:** (NEW - optional)
```yaml
---
name: dev-adapter-db-prisma
description: Prisma database adapters specifically
model: sonnet
color: blue
skills:
  - dev-adapter-db-core
  - dev-adapter-db-prisma
  - workflow-implementation
---

You implement Prisma repositories. Use dev-adapter-db-core patterns with
Prisma-specific implementation. Follow workflow-implementation TDD.
```

---

## Phase 5: Testing & Validation (Day 6)

### 5.1 Test Each Command

**Test Matrix:**

| Command | Test Case | Expected Outcome |
|---------|-----------|------------------|
| /imp | Run implementation workflow | Loads workflow-implementation, workflow-project, delegates correctly |
| /feat-imp | Run feature implementation | Same as /imp but with backlog/ paths |
| /fix | Debug workflow | TDD debugging pattern works |
| /plan | Create plan | Uses workflow-project, delegates to backend-architect if needed |
| /feat-refine | Refine spec | Spec management patterns work |

**Validation Checklist:**
- [ ] All commands load correct skills
- [ ] No missing skill references
- [ ] TDD workflow works end-to-end
- [ ] Autonomy rules applied correctly
- [ ] Agent delegation works
- [ ] Git workflow (commit, changeset) works
- [ ] No duplication in command content

---

### 5.2 Test Agent Specialization

**Test Cases:**

1. **General dev-core agent:**
   - Task: "Implement User entity and RegisterUser use case"
   - Expected: Loads both domain + application skills

2. **Specialized dev-core-domain agent:**
   - Task: "Create Product entity with value objects"
   - Expected: Loads only domain skill, TDD workflow

3. **Specialized dev-core-application agent:**
   - Task: "Create CreateOrder use case"
   - Expected: Loads only application skill, TDD workflow

4. **Database agent:**
   - Task: "Implement UserRepository with Prisma"
   - Expected: Loads core + prisma skills

---

### 5.3 Verify No Regressions

**Run through typical workflow:**
1. `/plan` - Create implementation plan
2. `/imp` - Implement iteration 1
3. `/imp` - Implement iteration 2
4. `/fix` - Fix a bug

**Verify:**
- [ ] All steps complete successfully
- [ ] Knowledge accessible when needed
- [ ] No confusion from missing content
- [ ] Commands are easier to read
- [ ] Skills provide needed depth

---

## Phase 6: Cutover (Day 7)

### 6.1 Backup Current State

```bash
# Backup entire .claude directory
cp -r .claude .claude.backup
git add .claude.backup
git commit -m "backup: pre-refactor claude config"
```

---

### 6.2 Delete Old Files (Hard Cutover)

```bash
# Delete old monolithic skills
rm .claude/skills/dev-core/SKILL.md
rm .claude/skills/dev-adapter-db/SKILL.md

# Delete old commands (replaced by .new versions)
rm .claude/commands/imp.md
rm .claude/commands/feat-imp.md
rm .claude/commands/fix.md
rm .claude/commands/plan.md
rm .claude/commands/feat-refine.md

# Rename new commands
mv .claude/commands/imp.md.new .claude/commands/imp.md
mv .claude/commands/feat-imp.md.new .claude/commands/feat-imp.md
mv .claude/commands/fix.md.new .claude/commands/fix.md
mv .claude/commands/plan.md.new .claude/commands/plan.md
mv .claude/commands/feat-refine.md.new .claude/commands/feat-refine.md
```

---

### 6.3 Final Commit

```bash
git add .claude/
git commit -m "refactor: command/skill structure - zero duplication

- Extract workflow knowledge to 3 skills (~500 lines each)
- Split dev-core → domain + application
- Split dev-adapter-db → core + tech-specific
- Simplify commands to 30-40 lines (pure workflow)
- Add specialized agents (hybrid strategy)
- Hard cutover, no backwards compatibility

Commands reduced 51% (360 → 175 lines)
Skills better organized, max 700 lines (was 1637)
Zero knowledge duplication

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

## Detailed Implementation Checklist

### Day 1-2: Workflow Skills

#### workflow-implementation
- [ ] Create `.claude/skills/workflow-implementation/SKILL.md`
- [ ] Write header (name, description, version)
- [ ] Section: TDD Cycle (200 lines)
  - [ ] Red-Green-Refactor pattern
  - [ ] Baseline verification
  - [ ] Test types (unit, integration, e2e)
- [ ] Section: Autonomy Rules (100 lines)
  - [ ] When to ask vs proceed
  - [ ] Decision framework
  - [ ] Question quality guidelines
- [ ] Section: Integration Testing (100 lines)
  - [ ] Automated tests checklist
  - [ ] Manual tests checklist
  - [ ] Failure handling
- [ ] Section: Test Organization (50 lines)
  - [ ] File structure
  - [ ] Naming conventions
  - [ ] AAA pattern
- [ ] Section: Performance (50 lines)
  - [ ] Running single tests
  - [ ] When to run full suite

#### workflow-project
- [ ] Create `.claude/skills/workflow-project/SKILL.md`
- [ ] Write header
- [ ] Section: Spec Management (200 lines)
  - [ ] Business specs
  - [ ] Feature specs
  - [ ] Implementation plans
  - [ ] Template structures
- [ ] Section: Iteration Tracking (100 lines)
  - [ ] Status values
  - [ ] Completion criteria
  - [ ] Approach research
- [ ] Section: Project Structure (100 lines)
  - [ ] File locations
  - [ ] Folder conventions
  - [ ] Build commands
- [ ] Section: Git Workflow (100 lines)
  - [ ] Branch naming
  - [ ] Commit messages
  - [ ] Changeset creation

#### workflow-agent-delegation
- [ ] Create `.claude/skills/workflow-agent-delegation/SKILL.md`
- [ ] Write header
- [ ] Section: Agent Matrix (100 lines)
  - [ ] Responsibility table
  - [ ] Task type indicators
- [ ] Section: Task Mapping (150 lines)
  - [ ] Decision tree
  - [ ] Delegation patterns
- [ ] Section: Task Tool Usage (100 lines)
  - [ ] Invocation templates
  - [ ] Prompt patterns
- [ ] Section: Parallel vs Sequential (100 lines)
  - [ ] When to parallelize
  - [ ] Dependencies
- [ ] Section: Examples (50 lines)

---

### Day 3: Split Skills

#### dev-core-domain
- [ ] Create `.claude/skills/dev-core-domain/SKILL.md`
- [ ] Extract lines 1-53 (header)
- [ ] Extract lines 54-276 (Functional IDs)
- [ ] Extract lines 283-399 (Ubiquitous Language)
- [ ] Extract lines 400-514 (Entities)
- [ ] Extract lines 516-637 (Value Objects)
- [ ] Extract lines 639-678 (Domain Services)
- [ ] Extract lines 680-702 (Domain Exceptions)
- [ ] Extract relevant testing sections
- [ ] Verify total ~700 lines

#### dev-core-application
- [ ] Create `.claude/skills/dev-core-application/SKILL.md`
- [ ] Write adapted header
- [ ] Extract lines 704-722 (Repository Interfaces)
- [ ] Extract lines 724-817 (Use Cases)
- [ ] Extract lines 819-847 (DTOs)
- [ ] Extract lines 849-1113 (Ports Design) ← CRITICAL
- [ ] Extract lines 1115-1154 (Application Services)
- [ ] Extract lines 1156-1167 (Application Exceptions)
- [ ] Extract relevant testing sections
- [ ] Verify total ~600 lines

#### dev-adapter-db-core
- [ ] Create `.claude/skills/dev-adapter-db-core/SKILL.md`
- [ ] Extract repository pattern fundamentals
- [ ] Extract entity mapping principles
- [ ] Extract transaction management
- [ ] Extract testing patterns
- [ ] Verify total ~600 lines

#### dev-adapter-db-prisma
- [ ] Create `.claude/skills/dev-adapter-db-prisma/SKILL.md`
- [ ] Extract Prisma client usage
- [ ] Extract schema definition
- [ ] Extract migrations
- [ ] Extract type generation
- [ ] Verify total ~400 lines

#### dev-adapter-db-drizzle
- [ ] Create `.claude/skills/dev-adapter-db-drizzle/SKILL.md`
- [ ] Extract Drizzle ORM patterns
- [ ] Extract schema builders
- [ ] Extract query building
- [ ] Verify total ~400 lines

#### dev-adapter-db-raw
- [ ] Create `.claude/skills/dev-adapter-db-raw/SKILL.md`
- [ ] Extract raw SQL approaches
- [ ] Extract Knex patterns
- [ ] Extract connection pooling
- [ ] Verify total ~237 lines

---

### Day 4: Refactor Commands

#### imp.md
- [ ] Create `.claude/commands/imp.md.new`
- [ ] Write header with skills list
- [ ] Write 5-step workflow (procedural only)
- [ ] Reference workflow-implementation for TDD
- [ ] Reference workflow-project for tracking
- [ ] Reference workflow-agent-delegation for agent selection
- [ ] Verify 30-40 lines total
- [ ] Test: /imp command works

#### feat-imp.md
- [ ] Create `.claude/commands/feat-imp.md.new`
- [ ] Adapt from imp.md for /backlog/ paths
- [ ] Same skill references
- [ ] Verify 30-40 lines
- [ ] Test: /feat-imp command works

#### fix.md
- [ ] Create `.claude/commands/fix.md.new`
- [ ] Focus on TDD debugging workflow
- [ ] Reference workflow-implementation
- [ ] Verify 30-40 lines
- [ ] Test: /fix command works

#### plan.md
- [ ] Create `.claude/commands/plan.md.new`
- [ ] Focus on plan creation workflow
- [ ] Reference workflow-project
- [ ] Reference workflow-agent-delegation
- [ ] Verify 25-30 lines
- [ ] Test: /plan command works

#### feat-refine.md
- [ ] Create `.claude/commands/feat-refine.md.new`
- [ ] Focus on spec refinement
- [ ] Reference workflow-project
- [ ] Verify 30-40 lines
- [ ] Test: /feat-refine command works

---

### Day 5: Update Agents

#### Update existing
- [ ] Update `.claude/agents/dev-core.md`
  - [ ] Add dev-core-domain to skills
  - [ ] Add dev-core-application to skills
  - [ ] Add workflow-implementation to skills
  - [ ] Update description
- [ ] Update `.claude/agents/dev-adapter-db.md`
  - [ ] Add dev-adapter-db-core to skills
  - [ ] Add dev-adapter-db-prisma to skills
  - [ ] Add workflow-implementation to skills
  - [ ] Update description

#### Create new specialized
- [ ] Create `.claude/agents/dev-core-domain.md`
- [ ] Create `.claude/agents/dev-core-application.md`
- [ ] Create `.claude/agents/dev-adapter-db-prisma.md` (optional)

---

### Day 6: Testing

#### Command Testing
- [ ] Test /imp workflow end-to-end
- [ ] Test /feat-imp workflow
- [ ] Test /fix workflow
- [ ] Test /plan workflow
- [ ] Test /feat-refine workflow
- [ ] Verify all skills load correctly
- [ ] Verify no missing references
- [ ] Verify autonomy rules work

#### Agent Testing
- [ ] Test dev-core agent (general)
- [ ] Test dev-core-domain agent (specialized)
- [ ] Test dev-core-application agent (specialized)
- [ ] Test dev-adapter-db agent
- [ ] Verify skill loading is correct
- [ ] Verify no confusion

#### Integration Testing
- [ ] Run full workflow: plan → imp → fix
- [ ] Verify knowledge accessible
- [ ] Verify TDD patterns work
- [ ] Verify git workflow works
- [ ] Check for any regressions

---

### Day 7: Cutover

- [ ] Backup .claude directory
- [ ] Delete old skills (dev-core, dev-adapter-db)
- [ ] Delete old commands
- [ ] Rename .new commands
- [ ] Final test all commands
- [ ] Commit refactored structure
- [ ] Document changes in vibe-template.md

---

## Success Metrics

**Before → After:**
- Commands: 360 lines → 175 lines (-51%)
- Max skill: 1637 lines → 700 lines (-57%)
- Duplication: 30% → 0%
- Workflow skills: 0 → 3
- Knowledge in skills: ~70% → ~95%

**Qualitative:**
- ✓ Commands easier to read
- ✓ Single source of truth
- ✓ Skills composable
- ✓ Easier to maintain

---

## Rollback Plan

If refactor fails:

```bash
# Restore backup
rm -rf .claude
mv .claude.backup .claude
git checkout .claude
git reset --hard HEAD~1  # If committed
```

---

## Post-Refactor Documentation

Update `vibe-template.md`:
```markdown
## Claude Code Configuration

### Workflow Skills
- workflow-implementation: TDD + autonomy
- workflow-project: Specs + git + project structure
- workflow-agent-delegation: Agent selection

### Development Skills
- dev-core-domain: Domain layer (entities, VOs)
- dev-core-application: Application layer (use cases, ports)
- dev-adapter-db-core: Repository patterns
- dev-adapter-db-prisma: Prisma-specific

### Commands
All commands are 30-40 lines of pure workflow logic.
Knowledge lives in skills, commands reference skills.
```

---

## Timeline Summary

| Day | Phase | Deliverables |
|-----|-------|--------------|
| 1-2 | Workflow Skills | 3 skills created (~1500 lines) |
| 3 | Split Large Skills | dev-core → 2, dev-adapter-db → 4 |
| 4 | Refactor Commands | 5 commands refactored to 30-40 lines |
| 5 | Update Agents | 2 updated, 3 new specialized agents |
| 6 | Testing | All workflows validated |
| 7 | Cutover | Hard cutover, commit, done |

**Total:** 7 days for complete refactor

---

**Plan Version:** 1.0
**Date:** 2025-11-16
**Status:** Ready for Implementation
**Approved By:** User
**Next Step:** Start Day 1 - Create workflow-implementation skill
