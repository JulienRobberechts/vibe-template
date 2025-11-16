# Claude Code Command & Skill Refactor - Research Documentation

This directory contains comprehensive research and planning for refactoring Claude Code's command and skill system to reduce duplication, improve maintainability, and enhance modularity.

## Documents Overview

### 1. [IMPLEMENTATION-PLAN.md](./IMPLEMENTATION-PLAN.md) ⭐ **START HERE**
**Precise, approved implementation plan ready to execute.**

**Contents:**
- User decisions applied (big bang, hard cutover, 500-line skills)
- Day-by-day implementation schedule (7 days)
- Detailed checklists for each phase
- Exact file operations and commands
- Testing procedures
- Rollback plan
- Success metrics

**Read this** to start implementation immediately.

---

### 2. [QUICK-REFERENCE.md](./QUICK-REFERENCE.md) 📋
**Quick reference guide for implementation.**

**Contents:**
- At-a-glance status and decisions
- Week schedule overview
- Key bash commands
- Daily checklists
- Testing commands
- Common issues & solutions

**Use this** during implementation for quick lookups.

---

### 3. [command-skill-refactor-plan.md](./command-skill-refactor-plan.md)
**Original strategic planning document.**

**Contents:**
- Current structure analysis (agents, commands, skills)
- Knowledge distribution issues
- Phase-by-phase refactoring recommendations
- New skills to create (workflow-tdd, workflow-autonomy, etc.)
- Existing skills to reorganize (split dev-core, dev-adapter-db, react-dev)
- Command simplification approach
- Implementation roadmap with timeline
- Success metrics and risk mitigation

**Read this** for the big picture and strategic direction.

---

### 4. [knowledge-mapping-analysis.md](./knowledge-mapping-analysis.md)
**Detailed knowledge extraction and duplication analysis.**

**Contents:**
- Line-by-line mapping of extractable knowledge from commands
- Knowledge duplication matrix across commands
- Visual diagrams of before/after structure
- Detailed specifications for new skills
- Split recommendations for large skills (dev-core, dev-adapter-db, react-dev)
- Migration priority ranking
- Questions for team review

**Read this** for detailed tactical implementation guidance.

---

### 5. [implementation-examples.md](./implementation-examples.md)
**Concrete before/after examples of refactoring.**

**Contents:**
- Example: imp.md refactoring (97 → 40 lines)
- Example: Creating workflow-tdd skill (full content)
- Command comparison matrix (all commands)
- Agent configuration changes
- Multi-technology skill usage patterns
- Skill composition patterns
- Key takeaways and best practices

**Read this** to see exactly what the refactored code looks like.

---

## Quick Summary

### Current Problems

**Commands (10-97 lines):**
- Embed knowledge that should be in skills
- ~30% duplication across commands
- Mix procedural workflow with domain knowledge
- Hard to maintain (update knowledge in many places)

**Skills (135-1637 lines):**
- Some too large (dev-core 1394 lines, dev-adapter-db 1637 lines)
- Mix multiple concerns (domain + application in dev-core)
- Load unnecessary content (all ORMs when using just Prisma)
- Could be more composable

### Proposed Solution

**Create 6 New "Workflow" Skills:**
1. **workflow-tdd** (~300 lines) - TDD cycle, testing patterns
2. **workflow-autonomy** (~250 lines) - When to ask questions, decision rules
3. **workflow-spec-management** (~300 lines) - Spec templates, iteration tracking
4. **workflow-agent-delegation** (~200 lines) - Agent selection, delegation patterns
5. **git-workflow** (~200 lines) - Git operations, commit messages
6. **project-structure** (~150 lines) - Folder conventions, file locations

**Reorganize Existing Skills:**
- Split **dev-core** → dev-core-domain + dev-core-application
- Split **dev-adapter-db** → core + prisma + drizzle + raw
- Split **react-dev** → core + testing + performance
- Extract **react-shared** from react-dev + react-native-dev

**Simplify All Commands:**
- Extract embedded knowledge → workflow skills
- Keep only procedural steps (~30-50 lines per command)
- Reference skills for "how to do it"
- Reduce duplication from 30% to <5%

### Expected Results

**Commands:**
- 51% size reduction (360 → 175 lines total)
- Zero duplication
- Much easier to read/understand
- Pure workflow logic

**Skills:**
- +6 new workflow skills (~1400 lines)
- Better organized (split large skills)
- More composable (mix and match)
- Faster context loading (load only what's needed)

**Overall:**
- Single source of truth for all knowledge
- Easier to maintain (change once, affects all)
- Better agent specialization
- Clearer separation: commands = what, skills = how

## Implementation Phases

### Phase 1: Create Workflow Skills (Week 1)
Priority: workflow-autonomy, workflow-tdd, workflow-agent-delegation

### Phase 2: Split Large Skills (Week 2)
Priority: dev-core, dev-adapter-db

### Phase 3: Simplify Commands (Week 3)
Priority: imp.md, feat-imp.md, fix.md

### Phase 4: Testing & Validation (Week 4)
Test all commands, verify no regressions

### Phase 5: Documentation (Week 5)
Update docs, create migration guide

## Key Metrics

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Avg command size | 50 lines | 30 lines | -40% |
| Max skill size | 1637 lines | 800 lines | -51% |
| Knowledge duplication | 30% | <5% | -83% |
| Total workflow skills | 0 | 6 | +6 |
| Commands with embedded knowledge | 8/11 (73%) | 0/11 (0%) | -73% |

## Getting Started

### ✅ Decisions Made - Ready to Implement

All critical decisions have been made:
- Skill size: 500 lines (merged smaller skills)
- Migration: Big bang (1 week)
- Backwards compat: Hard cutover
- Splits: Only dev-core & dev-adapter-db
- Agents: Hybrid (some specialized)

### For Implementers (START HERE):

1. **Read** [IMPLEMENTATION-PLAN.md](./IMPLEMENTATION-PLAN.md) - Complete 7-day plan
2. **Reference** [QUICK-REFERENCE.md](./QUICK-REFERENCE.md) - Daily checklists
3. **Execute** Day 1: Create workflow-implementation skill
4. **Follow** the detailed checklist for each day

### For Reviewers:
1. Read [command-skill-refactor-plan.md](./command-skill-refactor-plan.md) - Strategic overview
2. Review [knowledge-mapping-analysis.md](./knowledge-mapping-analysis.md) - Detailed analysis
3. Check [implementation-examples.md](./implementation-examples.md) - Code examples
4. All decisions documented in README decisions section

### For Decision Makers:
✅ **Plan Approved** - All decisions finalized, ready for implementation

## Critical Decisions Needed

### 1. Skill Granularity
**Question:** Are 200-300 line workflow skills too small?

**Options:**
- A) Keep as proposed (6 skills ~200-300 lines)
- B) Merge some (e.g., workflow-tdd + workflow-autonomy)
- C) Split further (e.g., workflow-tdd-unit vs workflow-tdd-integration)

**Recommendation:** A (keeps concerns separated)
Choice: A

---

### 2. Migration Strategy
**Question:** Big bang or gradual rollout?

**Options:**
- A) Refactor all at once (1 week, risky)
- B) Gradual (1 skill + 1 command per week, 6 weeks)
- C) Parallel (keep old + new, A/B test)

**Recommendation:** B (safer, allows learning)
Choice: A

---

### 3. Backwards Compatibility
**Question:** Keep old commands during transition?

**Options:**
- A) Hard cutover (delete old immediately)
- B) Deprecate (keep old, mark as deprecated)
- C) Alias (old commands call new commands)

**Recommendation:** C (zero disruption)
Choice: A

---

### 4. Large Skill Splits
**Question:** Split all skills >800 lines or only problematic ones?

**Options:**
- A) Split all >800 lines (7 skills affected)
- B) Split only most problematic (dev-core, dev-adapter-db)
- C) Don't split, just optimize

**Recommendation:** B (high impact, low risk)
Choice: B

---

### 5. Agent Proliferation
**Question:** Create specialized agents for split skills?

**Options:**
- A) Yes (e.g., dev-core-domain agent, dev-core-application agent)
- B) No (keep existing agents, they load multiple skills)
- C) Hybrid (some specialized, some composite)

**Recommendation:** B (simpler, agents already composite)
Choice: C

---

## Visual Summary

```
┌─────────────────────────────────────────────────────────────┐
│                    CURRENT STATE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Commands (11 files)          Skills (9 skills)             │
│  ┌─────────────────┐         ┌──────────────────┐          │
│  │ imp.md          │         │ dev-core         │          │
│  │ (97 lines)      │         │ (1394 lines)     │          │
│  │                 │         │                  │          │
│  │ ▪ Workflow      │         │ ▪ Domain layer   │          │
│  │ ▪ TDD knowledge │ ◄────┐  │ ▪ App layer      │          │
│  │ ▪ Autonomy rules│      │  │ ▪ Entities       │          │
│  │ ▪ Agent logic   │      │  │ ▪ Use cases      │          │
│  │ ▪ Testing steps │      │  │ ▪ Ports          │          │
│  └─────────────────┘      │  └──────────────────┘          │
│                           │                                 │
│  ┌─────────────────┐     │  ┌──────────────────┐          │
│  │ feat-imp.md     │     │  │ dev-adapter-db   │          │
│  │ (81 lines)      │     │  │ (1637 lines)     │          │
│  │                 │     │  │                  │          │
│  │ ▪ Similar       │ ◄───┤  │ ▪ Prisma         │          │
│  │ ▪ Duplicate TDD │     │  │ ▪ Drizzle        │          │
│  │ ▪ Duplicate     │     │  │ ▪ Knex           │          │
│  │   autonomy      │     │  │ ▪ Raw SQL        │          │
│  └─────────────────┘     │  └──────────────────┘          │
│                          │                                 │
│         ... (9 more commands with duplication)             │
│                                                             │
│  PROBLEMS:                                                  │
│  ✗ 30% knowledge duplication                               │
│  ✗ Commands mix workflow + knowledge                       │
│  ✗ Skills too large (1394-1637 lines)                      │
│  ✗ Hard to maintain                                        │
└─────────────────────────────────────────────────────────────┘

                            ▼ REFACTOR ▼

┌─────────────────────────────────────────────────────────────┐
│                    FUTURE STATE                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Commands (11 files)          Skills (21 skills)            │
│  ┌─────────────────┐         ┌──────────────────┐          │
│  │ imp.md          │    ┌───►│ workflow-tdd     │          │
│  │ (40 lines)      │    │    │ (300 lines)      │          │
│  │                 │    │    └──────────────────┘          │
│  │ ▪ Workflow only │────┤                                  │
│  │ ▪ References:   │    │    ┌──────────────────┐          │
│  │   - workflow-tdd│    ├───►│ workflow-autonomy│          │
│  │   - workflow-   │    │    │ (250 lines)      │          │
│  │     autonomy    │    │    └──────────────────┘          │
│  │   - workflow-   │    │                                  │
│  │     agent-      │    │    ┌──────────────────┐          │
│  │     delegation  │    └───►│ workflow-agent-  │          │
│  └─────────────────┘         │ delegation       │          │
│                               │ (200 lines)      │          │
│  ┌─────────────────┐         └──────────────────┘          │
│  │ feat-imp.md     │                                       │
│  │ (35 lines)      │    ┌────► (Same skills)               │
│  │                 │    │                                  │
│  │ ▪ Workflow only │────┤    ┌──────────────────┐          │
│  │ ▪ References    │    └───►│ dev-core-domain  │          │
│  │   same skills   │         │ (700 lines)      │          │
│  └─────────────────┘         └──────────────────┘          │
│                                                             │
│         ... (9 more commands, all thin)                     │
│                               ┌──────────────────┐          │
│                               │ dev-core-app     │          │
│  BENEFITS:                    │ (600 lines)      │          │
│  ✓ Zero duplication           └──────────────────┘          │
│  ✓ Commands are pure workflow                              │
│  ✓ Skills focused & reusable  ┌──────────────────┐          │
│  ✓ Easy to maintain           │ dev-adapter-db-  │          │
│                               │ prisma           │          │
│                               │ (400 lines)      │          │
│                               └──────────────────┘          │
│                                                             │
│                               ... (15 more focused skills)  │
└─────────────────────────────────────────────────────────────┘
```

## Next Actions

**Immediate (This Week):**
- [ ] Team reviews these documents
- [ ] Decisions made on 5 critical questions above
- [ ] Approve/modify/reject overall plan

**Short Term (Next 2 Weeks):**
- [ ] Create first workflow skill (workflow-tdd recommended)
- [ ] Refactor first command (imp.md recommended)
- [ ] Validate approach with real usage

**Medium Term (Weeks 3-6):**
- [ ] Create remaining workflow skills
- [ ] Split large skills (dev-core, dev-adapter-db)
- [ ] Refactor all commands
- [ ] Update agents

**Long Term (Weeks 7-8):**
- [ ] Full testing across all workflows
- [ ] Documentation updates
- [ ] Team training on new structure
- [ ] Deprecate old patterns

## Questions?

Contact project maintainer or open issue with:
- Tag: `refactor-planning`
- Reference: This research directory
- Specific questions from documents

## Document Index

Priority order for implementation:

1. **[IMPLEMENTATION-PLAN.md](./IMPLEMENTATION-PLAN.md)** ⭐ - Precise 7-day implementation plan
2. **[QUICK-REFERENCE.md](./QUICK-REFERENCE.md)** 📋 - Quick reference during implementation
3. [command-skill-refactor-plan.md](./command-skill-refactor-plan.md) - Strategic overview
4. [knowledge-mapping-analysis.md](./knowledge-mapping-analysis.md) - Knowledge extraction details
5. [implementation-examples.md](./implementation-examples.md) - Before/after code examples
6. [README.md](./README.md) - This file

---

**Research Version:** 1.1
**Date:** 2025-11-16
**Status:** ✅ Approved - Ready for Implementation
**Next Step:** Start Day 1 - Create workflow-implementation skill
