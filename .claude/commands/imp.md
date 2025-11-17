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
