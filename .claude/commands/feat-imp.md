Implement the feature using TDD. Work autonomously without stopping unless blocked.

## Workflow

1. **Resume State**
   - Read `/backlog/{{ARGUMENTS}}`: check status
   - Check git status + existing todos
   - If COMPLETED: skip to next feature
   - If in progress: resume from last todo

2. **Verify Spec**
   - Extract objectives, tasks, deliverables, manual tests
   - If incomplete: complete spec, ask validation

3. **Plan Tasks** (if no active todos)
   - Use TodoWrite: create tasks from spec checklist
   - Only ask blocking questions

4. **Baseline Check**
   - Run `npm test` (fix if fails)
   - Run `npm run dev` (verify working)

5. **TDD Cycle** (repeat per todo)
   - Write failing test
   - Minimal implementation to pass
   - Refactor + verify (automated + manual)
   - Mark todo completed
   - Continue to next without stopping

6. **Integration Verify**
   - Run full suite: `npm test`
   - Run dev: `npm run dev`
   - Complete ALL manual tests from spec
   - Test responsive if UI
   - Fix failures, repeat if needed

7. **Complete Feature**
   - Update `/backlog/{{ARGUMENTS}}`:
     - Status: ✅ COMPLETED
     - Completed: YYYY-MM-DD
     - Deviations + unresolved issues

8. **Commit**
   - Stage changes
   - Add commit message: "feat: [brief]"
   - Add changeset if needed
   - Ask user to commit

## Autonomy Rules

**Work continuously unless:**
- Missing critical info (unclear spec, ambiguous requirement)
- External blocker (API key, missing package, environment)
- Critical error blocking progress

**Do NOT ask about:**
- Implementation details
- Code style
- Test/variable naming
- Minor architecture choices

**When blocked:**
- Resolve independently first (docs, existing code)
- Ask ONE specific question
- Continue unblocked work while waiting

## Rules

- Feature = working, testable app
- Follow architecture (docs/implementation.md)
- Never commit failing tests
- Run full suite before commit
- Make forward progress every execution

## Output

- What was built (1 sentence)
- Files changed
- Manual test results
- Next feature recommendation

