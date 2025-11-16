# Claude Code Command & Skill Refactor Plan

## Analysis Summary

### Current Structure

```
.claude/
├── agents/          # 9 files (agent configs referencing skills)
├── commands/        # 11 files (10-97 lines each, workflow procedures)
├── skills/          # 9 directories (135-1637 lines each, deep knowledge)
└── settings.json
```

**Key Findings:**
- **Agents** are lightweight configs (metadata + skill references)
- **Skills** contain extensive domain knowledge and best practices
- **Commands** mix workflow procedures with embedded knowledge
- Commands contain knowledge that could live in skills

### Knowledge Distribution Issues

#### Commands with Embedded Knowledge

1. **imp.md** (97 lines)
   - Workflow: TDD cycle, autonomy rules, integration testing
   - Embedded knowledge: When to delegate to specialized agents, agent selection criteria
   - Issue: Duplicates agent responsibility knowledge

2. **plan.md** (58 lines)
   - Workflow: Planning process, architectural vs feature planning
   - Embedded knowledge: When to use backend-architect agent
   - Issue: Agent selection logic embedded in command

3. **feat-imp.md** (81 lines)
   - Similar to imp.md but for backlog features
   - Issue: Duplicates much of imp.md workflow

4. **feat-refine.md** (64 lines)
   - Workflow: Spec refinement, approach selection
   - Embedded knowledge: How to research, propose alternatives
   - Issue: Could benefit from "spec-refinement" skill

5. **fix.md** (60 lines)
   - Workflow: Bug fixing with TDD
   - Issue: Similar structure to imp.md, could share knowledge

## Refactoring Recommendations

### Phase 1: Extract Command Knowledge into Skills

#### 1.1 Create "workflow-tdd" Skill

**Purpose:** Centralize TDD workflow knowledge shared across commands

**Extract from:**
- imp.md → TDD cycle, baseline verification, integration testing
- feat-imp.md → Similar TDD patterns
- fix.md → TDD debugging approach

**Content:**
```
workflow-tdd/
  SKILL.md
  - TDD red-green-refactor cycle
  - Baseline verification (npm test, npm run dev)
  - Integration testing checklist
  - Manual testing guidelines
  - Test-first debugging approaches
```

**Benefits:**
- Single source of truth for TDD practices
- Commands become thinner (procedural only)
- Consistency across all TDD workflows

#### 1.2 Create "workflow-autonomy" Skill

**Purpose:** Define autonomy rules and decision-making criteria

**Extract from:**
- imp.md → Autonomy rules (when to stop, when NOT to ask)
- feat-imp.md → Same autonomy patterns
- All commands → Decision-making criteria

**Content:**
```
workflow-autonomy/
  SKILL.md
  - When to work autonomously vs ask questions
  - When to stop (blocked scenarios)
  - What NOT to ask about (implementation details, style)
  - How to resolve blocks independently
  - Prioritization heuristics
```

**Benefits:**
- Consistent autonomy behavior across all commands
- Easy to update rules globally
- Clear guidelines for question-asking

#### 1.3 Create "workflow-spec-management" Skill

**Purpose:** Spec creation, refinement, and iteration tracking

**Extract from:**
- feat-refine.md → Approach selection, research patterns
- spec-create.md → Template application
- imp.md → Implementation.md status tracking

**Content:**
```
workflow-spec-management/
  SKILL.md
  - Spec template structures
  - Approach research methodology
  - Presenting multiple technical approaches
  - Iteration status tracking
  - Spec completeness criteria
```

#### 1.4 Create "workflow-agent-delegation" Skill

**Purpose:** When and how to delegate to specialized agents

**Extract from:**
- imp.md → Agent identification logic (dev-core, dev-api, etc.)
- plan.md → Architectural vs feature planning agent choice

**Content:**
```
workflow-agent-delegation/
  SKILL.md
  - Agent responsibility matrix
  - Task → Agent mapping criteria
  - When to use backend-architect
  - When to use dev-* specialized agents
  - Parallel vs sequential delegation
  - How to formulate agent prompts
```

### Phase 2: Reorganize Existing Skills

#### 2.1 Split "dev-adapter-db" (1637 lines)

**Current:** Too large, covers many DB technologies

**Split into:**

A. **dev-adapter-db-core** (600 lines)
   - Repository pattern fundamentals
   - Entity mapping principles
   - Transaction management patterns
   - Testing database adapters
   - Common across all ORMs

B. **dev-adapter-db-prisma** (400 lines)
   - Prisma-specific patterns
   - Schema design
   - Migrations
   - Type generation

C. **dev-adapter-db-drizzle** (400 lines)
   - Drizzle-specific patterns
   - Query building
   - Type-safe queries

D. **dev-adapter-db-raw** (237 lines)
   - Raw SQL approaches
   - Query builders (Knex)
   - Connection pooling

**Benefits:**
- Faster loading (load only what's needed)
- Easier maintenance (technology-specific)
- Better agent specialization

#### 2.2 Split "dev-core" (1394 lines)

**Current:** Comprehensive but could be more focused

**Split into:**

A. **dev-core-domain** (700 lines)
   - Entities
   - Value Objects
   - Domain Services
   - Domain Events
   - Business Rules
   - Functional IDs
   - Ubiquitous Language

B. **dev-core-application** (600 lines)
   - Use Cases
   - Application Services
   - DTOs
   - Ports (interface design principles)
   - Application Exceptions

C. Keep **dev-core** as coordinator (100 lines)
   - References both domain and application
   - High-level architecture overview
   - When to use which sub-skill

**Benefits:**
- Clearer separation of concerns
- Agents can specialize (domain-only vs application-only)
- Faster context loading

#### 2.3 Split "react-dev" (859 lines)

**Current:** Covers many React concerns

**Split into:**

A. **react-dev-core** (400 lines)
   - Component design
   - Hooks patterns
   - State management basics
   - Performance fundamentals

B. **react-dev-testing** (300 lines)
   - Testing Library patterns
   - Component testing
   - Hook testing
   - Integration testing

C. **react-dev-performance** (159 lines)
   - Memoization
   - Code splitting
   - Virtualization
   - Profiling

**Benefits:**
- Load only relevant knowledge (testing vs implementation)
- Better agent focus (dev vs test agents)

#### 2.4 Merge Similar Skills

**Candidates:**
- **react-dev** + **react-native-dev** → Share common patterns in **react-shared** skill
  - Both share hooks, state, component patterns
  - Extract common knowledge (300 lines)
  - Keep platform-specific parts separate

### Phase 3: Command Simplification

After extracting knowledge to skills, simplify commands:

#### Before (imp.md - 97 lines):
```markdown
## Workflow
1. Check State & Resume
   - Read docs/implementation.md
   - Check git status
   - [detailed instructions...]
2. Read Spec
   - Extract objectives
   - Identify appropriate agent: dev-core, dev-api, dev-adapter-db...
   - [agent selection logic...]
3. Delegate
   - Use Task tool with agent
   - [detailed delegation rules...]
4. Plan
   - Use TodoWrite
   - [planning details...]
5. Verify Baseline
   - Run npm test
   - [testing details...]
6. TDD Cycle
   - Test, Implement, Refactor, Verify
   - [TDD workflow details...]
...
## Autonomy Rules
[detailed rules...]
```

#### After (imp.md - ~40 lines):
```markdown
## Skills
- workflow-tdd: TDD cycle and testing
- workflow-autonomy: When to ask questions
- workflow-agent-delegation: Agent selection

## Workflow
1. Check State & Resume
   - Read docs/implementation.md status
   - Check git + todos

2. Read Spec & Delegate
   - Use workflow-agent-delegation to identify agent
   - Delegate via Task tool if match found

3. Plan (if no todos)
   - TodoWrite from spec checklist

4. TDD Cycle
   - Follow workflow-tdd patterns
   - Apply workflow-autonomy rules

5. Document & Commit
   - Update implementation.md
   - Commit with changeset
```

**Benefits:**
- Commands focus on WHAT to do (procedure)
- Skills provide HOW to do it (knowledge)
- Easier to update procedures vs knowledge
- Less duplication

### Phase 4: New Skills to Create

#### 4.1 "git-workflow" Skill

**Purpose:** Git operations, commit messages, branch naming

**Extract from:**
- Multiple commands reference git operations
- Commit message patterns (feat:, fix:, spec:)
- Changeset creation

**Content:**
- Branch naming conventions (jrob/feature-name)
- Commit message formats
- Changeset creation rules
- When to commit vs amend
- Git safety rules

#### 4.2 "project-structure" Skill

**Purpose:** Project-specific folder conventions

**Content:**
- Where specs live (/specs, /backlog, /docs)
- Implementation tracking (docs/implementation.md)
- Feature tracking (backlog/*.md)
- Test file locations
- Build/run commands

## Implementation Roadmap

### Step 1: Create New Workflow Skills (Week 1)
- [ ] Create workflow-tdd skill
- [ ] Create workflow-autonomy skill
- [ ] Create workflow-spec-management skill
- [ ] Create workflow-agent-delegation skill
- [ ] Create git-workflow skill
- [ ] Create project-structure skill

### Step 2: Split Large Skills (Week 2)
- [ ] Split dev-adapter-db into 4 skills
- [ ] Split dev-core into 3 skills
- [ ] Split react-dev into 3 skills

### Step 3: Extract Shared Knowledge (Week 3)
- [ ] Create react-shared skill
- [ ] Extract common patterns from similar skills

### Step 4: Simplify Commands (Week 4)
- [ ] Refactor imp.md to reference workflow skills
- [ ] Refactor feat-imp.md
- [ ] Refactor fix.md
- [ ] Refactor plan.md
- [ ] Refactor feat-refine.md
- [ ] Update agent configs to reference new skills

### Step 5: Testing & Validation (Week 5)
- [ ] Test each command with new structure
- [ ] Verify knowledge accessibility
- [ ] Ensure no regressions
- [ ] Update documentation

## Success Metrics

**Before:**
- Commands: 10-97 lines (avg ~50 lines)
- Skills: 135-1637 lines (avg ~730 lines)
- Knowledge duplication: ~30% across commands
- Total lines: ~8200 in skills + ~500 in commands = ~8700

**After:**
- Commands: 20-40 lines (avg ~30 lines)
- Skills: 100-800 lines (avg ~400 lines)
- Knowledge duplication: <5%
- Total lines: ~9000 (slight increase but better organized)
- New workflow skills: 6 x ~300 lines = 1800 lines
- Split skills: More granular, easier to maintain

**Benefits:**
- 40% reduction in command complexity
- 45% reduction in max skill size
- 80% reduction in knowledge duplication
- Better skill composability
- Easier agent specialization
- Faster context loading

## Risk Mitigation

**Risk:** Breaking existing workflows
**Mitigation:**
- Keep old commands temporarily
- Gradual migration
- A/B testing

**Risk:** Skills become too granular
**Mitigation:**
- Start with largest/most duplicated skills
- Monitor actual usage patterns
- Merge if too fragmented

**Risk:** Agent confusion with more skills
**Mitigation:**
- Clear skill naming conventions
- Skill discovery/search mechanism
- Agent configs explicitly list required skills

## Next Steps

1. **Validate Approach** - Review this plan with team
2. **Prototype** - Create workflow-tdd skill first (lowest risk)
3. **Measure** - Test command simplification with imp.md
4. **Iterate** - Adjust based on learnings
5. **Scale** - Apply to remaining commands and skills

## Appendix: Skill Organization Principles

### Good Skill Design
- **Single Responsibility:** Each skill covers one domain/concern
- **Right Size:** 200-800 lines (readable in one sitting)
- **Self-Contained:** Minimal cross-references
- **Technology-Focused:** When covering tools (Prisma, Zod, React)
- **Workflow-Focused:** When covering processes (TDD, autonomy)

### Good Command Design
- **Procedural:** Step-by-step what to do
- **Thin:** Reference skills for how to do it
- **Focused:** One clear goal
- **Composable:** Can combine with other commands

### Agent-Skill Relationship
- **Agent = Role:** "I am a core domain developer"
- **Skill = Knowledge:** "Use dev-core-domain for entity design"
- **Command = Task:** "Implement iteration from docs/implementation.md"

## File Structure After Refactor

```
.claude/
├── agents/                          # (unchanged)
│   ├── dev-core.md
│   ├── dev-api.md
│   └── ...
├── commands/                        # (simplified)
│   ├── imp.md                       # 40 lines (was 97)
│   ├── feat-imp.md                  # 35 lines (was 81)
│   ├── plan.md                      # 30 lines (was 58)
│   ├── fix.md                       # 35 lines (was 60)
│   └── ...
└── skills/
    ├── workflow-tdd/                # NEW
    │   └── SKILL.md                 # 300 lines
    ├── workflow-autonomy/           # NEW
    │   └── SKILL.md                 # 250 lines
    ├── workflow-spec-management/    # NEW
    │   └── SKILL.md                 # 300 lines
    ├── workflow-agent-delegation/   # NEW
    │   └── SKILL.md                 # 200 lines
    ├── git-workflow/                # NEW
    │   └── SKILL.md                 # 200 lines
    ├── project-structure/           # NEW
    │   └── SKILL.md                 # 150 lines
    ├── dev-core/                    # REORGANIZED
    │   └── SKILL.md                 # 100 lines (coordinator)
    ├── dev-core-domain/             # NEW (split from dev-core)
    │   └── SKILL.md                 # 700 lines
    ├── dev-core-application/        # NEW (split from dev-core)
    │   └── SKILL.md                 # 600 lines
    ├── dev-adapter-db-core/         # NEW (split from dev-adapter-db)
    │   └── SKILL.md                 # 600 lines
    ├── dev-adapter-db-prisma/       # NEW (split from dev-adapter-db)
    │   └── SKILL.md                 # 400 lines
    ├── dev-adapter-db-drizzle/      # NEW (split from dev-adapter-db)
    │   └── SKILL.md                 # 400 lines
    ├── dev-adapter-db-raw/          # NEW (split from dev-adapter-db)
    │   └── SKILL.md                 # 237 lines
    ├── react-shared/                # NEW (extracted commons)
    │   └── SKILL.md                 # 300 lines
    ├── react-dev-core/              # NEW (split from react-dev)
    │   └── SKILL.md                 # 400 lines
    ├── react-dev-testing/           # NEW (split from react-dev)
    │   └── SKILL.md                 # 300 lines
    ├── react-dev-performance/       # NEW (split from react-dev)
    │   └── SKILL.md                 # 159 lines
    └── [other existing skills...]
```

---

**Document Version:** 1.0
**Date:** 2025-11-16
**Status:** Proposal / Planning Phase
