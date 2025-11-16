# Quick Reference - Refactor Implementation

## At a Glance

**Status:** Ready to implement
**Timeline:** 7 days
**Approach:** Big bang + hard cutover
**Risk Level:** High (mitigated by testing)

---

## Your Decisions

| Decision | Choice |
|----------|--------|
| Skill Size | 500 lines (merged smaller skills) |
| Command Size | 30-40 lines ✓ |
| Split Strategy | Only problematic (dev-core, dev-adapter-db) |
| Naming | workflow-* / dev-* ✓ |
| Agents | Hybrid (some specialized) |
| Migration | Big bang |
| Backwards Compat | Hard cutover (delete old) |

---

## New Structure

### Workflow Skills (3 total, ~500 lines each)

1. **workflow-implementation** (500 lines)
   - Merged: TDD + autonomy
   - Content: Red-green-refactor, baseline, integration tests, autonomy rules

2. **workflow-project** (500 lines)
   - Merged: Spec mgmt + project structure + git
   - Content: Spec templates, iteration tracking, project conventions, git workflow

3. **workflow-agent-delegation** (500 lines)
   - Content: Agent matrix, task mapping, delegation patterns

### Split Skills

**dev-core** → **dev-core-domain** (700 lines) + **dev-core-application** (600 lines)
- Domain: Entities, VOs, domain services, functional IDs, ubiquitous language
- Application: Use cases, ports, DTOs, application services

**dev-adapter-db** → 4 skills
- **dev-adapter-db-core** (600 lines)
- **dev-adapter-db-prisma** (400 lines)
- **dev-adapter-db-drizzle** (400 lines)
- **dev-adapter-db-raw** (237 lines)

### New Agents (Hybrid Strategy)

✓ dev-core-domain (domain-only work)
✓ dev-core-application (use cases only)
✓ dev-adapter-db-prisma (Prisma-specific, optional)

---

## Week Schedule

```
┌─────────┬──────────────────────────────────────────┐
│ Day 1-2 │ Create 3 workflow skills (~1500 lines)   │
├─────────┼──────────────────────────────────────────┤
│ Day 3   │ Split dev-core & dev-adapter-db          │
├─────────┼──────────────────────────────────────────┤
│ Day 4   │ Refactor 5 commands to 30-40 lines       │
├─────────┼──────────────────────────────────────────┤
│ Day 5   │ Update agents, create specialized agents │
├─────────┼──────────────────────────────────────────┤
│ Day 6   │ Comprehensive testing & validation       │
├─────────┼──────────────────────────────────────────┤
│ Day 7   │ Hard cutover, final commit               │
└─────────┴──────────────────────────────────────────┘
```

---

## Key Commands

### Backup Before Starting
```bash
cp -r .claude .claude.backup
git add .claude.backup
git commit -m "backup: pre-refactor claude config"
```

### Create New Skills
```bash
mkdir -p .claude/skills/workflow-implementation
mkdir -p .claude/skills/workflow-project
mkdir -p .claude/skills/workflow-agent-delegation
mkdir -p .claude/skills/dev-core-domain
mkdir -p .claude/skills/dev-core-application
mkdir -p .claude/skills/dev-adapter-db-core
mkdir -p .claude/skills/dev-adapter-db-prisma
mkdir -p .claude/skills/dev-adapter-db-drizzle
mkdir -p .claude/skills/dev-adapter-db-raw
```

### Test Commands
```bash
# Test each command after refactoring
/imp
/feat-imp
/fix
/plan
/feat-refine
```

### Cutover (Day 7)
```bash
# Delete old monolithic skills
rm .claude/skills/dev-core/SKILL.md
rm .claude/skills/dev-adapter-db/SKILL.md

# Rename new commands
for cmd in imp feat-imp fix plan feat-refine; do
  rm .claude/commands/$cmd.md
  mv .claude/commands/$cmd.md.new .claude/commands/$cmd.md
done

# Final commit
git add .claude/
git commit -m "refactor: command/skill structure - zero duplication"
```

---

## Quick Skill Reference

### workflow-implementation
**When:** Any implementation work
**Provides:** TDD cycle, autonomy rules, integration testing
**Used by:** imp, feat-imp, fix

### workflow-project
**When:** Managing specs, tracking, git
**Provides:** Spec templates, iteration tracking, git workflow
**Used by:** imp, feat-imp, plan, feat-refine

### workflow-agent-delegation
**When:** Deciding which agent to use
**Provides:** Agent selection matrix, delegation patterns
**Used by:** imp, plan

### dev-core-domain
**When:** Domain layer work (entities, VOs)
**Provides:** Entity patterns, value objects, functional IDs
**Used by:** dev-core agent, dev-core-domain agent

### dev-core-application
**When:** Application layer work (use cases, ports)
**Provides:** Use case patterns, port design, DTOs
**Used by:** dev-core agent, dev-core-application agent

---

## Daily Checklist

### Day 1-2
- [ ] workflow-implementation skill created (500 lines)
- [ ] workflow-project skill created (500 lines)
- [ ] workflow-agent-delegation skill created (500 lines)
- [ ] All skills have proper headers
- [ ] Content extracted from commands correctly

### Day 3
- [ ] dev-core-domain created (700 lines)
- [ ] dev-core-application created (600 lines)
- [ ] dev-adapter-db-core created (600 lines)
- [ ] dev-adapter-db-prisma created (400 lines)
- [ ] dev-adapter-db-drizzle created (400 lines)
- [ ] dev-adapter-db-raw created (237 lines)

### Day 4
- [ ] imp.md.new created (35 lines)
- [ ] feat-imp.md.new created (35 lines)
- [ ] fix.md.new created (35 lines)
- [ ] plan.md.new created (30 lines)
- [ ] feat-refine.md.new created (35 lines)
- [ ] All commands reference correct skills

### Day 5
- [ ] dev-core agent updated
- [ ] dev-adapter-db agent updated
- [ ] dev-core-domain agent created
- [ ] dev-core-application agent created
- [ ] dev-adapter-db-prisma agent created (optional)

### Day 6
- [ ] All commands tested
- [ ] All agents tested
- [ ] Full workflow tested (plan → imp → fix)
- [ ] No regressions found

### Day 7
- [ ] Backup verified
- [ ] Old files deleted
- [ ] New files renamed
- [ ] Final commit made
- [ ] Documentation updated

---

## Metrics to Track

**Before:**
- Total command lines: 360
- Max skill size: 1637 lines
- Knowledge duplication: ~30%
- Workflow skills: 0

**Target After:**
- Total command lines: 175 (-51%)
- Max skill size: 700 lines (-57%)
- Knowledge duplication: 0%
- Workflow skills: 3

---

## Rollback If Needed

```bash
# If something goes wrong
rm -rf .claude
mv .claude.backup .claude
git checkout .claude
git reset --hard HEAD~1  # If already committed
```

---

## Testing Commands

### Test imp.md
```bash
# Should:
# - Load workflow-implementation skill
# - Load workflow-project skill
# - Load workflow-agent-delegation skill
# - Show TDD patterns
# - Show autonomy rules
# - Delegate to agent if appropriate
```

### Test plan.md
```bash
# Should:
# - Load workflow-project skill
# - Load workflow-agent-delegation skill
# - Create implementation plan
# - Use backend-architect if architectural
```

### Test dev-core agent
```bash
# Should:
# - Load dev-core-domain skill
# - Load dev-core-application skill
# - Load workflow-implementation skill
# - Provide both domain and application guidance
```

### Test dev-core-domain agent (specialized)
```bash
# Should:
# - Load ONLY dev-core-domain skill
# - Load workflow-implementation skill
# - Focus on entities, VOs, domain logic
```

---

## Common Issues & Solutions

### Issue: Skill not loading
**Solution:** Check agent config, verify skill exists in .claude/skills/

### Issue: Command shows "missing reference"
**Solution:** Check skill name matches exactly, check SKILL.md exists

### Issue: Too much/little content in command
**Solution:** Move knowledge to skill, keep only workflow steps in command

### Issue: Duplication found
**Solution:** Extract to appropriate workflow skill

### Issue: Agent confused about which skill to use
**Solution:** Check agent config skills list, ensure clear descriptions

---

## Success Indicators

✓ Commands are 30-40 lines
✓ No knowledge duplication across commands
✓ All skills ≤700 lines
✓ Clear separation: commands = workflow, skills = knowledge
✓ All tests pass
✓ Workflows complete successfully
✓ Easier to read and maintain

---

## Files to Review

1. **IMPLEMENTATION-PLAN.md** - Detailed day-by-day plan
2. **command-skill-refactor-plan.md** - Strategic overview
3. **knowledge-mapping-analysis.md** - Knowledge extraction details
4. **implementation-examples.md** - Before/after code examples
5. **README.md** - Documentation overview
6. **QUICK-REFERENCE.md** - This file

---

**Version:** 1.0
**Updated:** 2025-11-16
**Status:** Ready for Day 1
