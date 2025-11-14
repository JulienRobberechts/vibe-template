# Implementation

Implement iteration from `docs/implementation.md` using TDD. Work autonomously without stopping unless blocked.

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

Search for sub-agent to delegate the work.

3. **Plan** (if no active todos)
   - Use TodoWrite: create tasks from checklist
   - Break down into subtasks
   - Only list blocking questions that prevent starting

4. **Verify Baseline**
   - Run existing tests: `npm test`
   - If fails: fix before proceeding
   - Run dev: `npm run dev`

5. **TDD Cycle** (repeat until all todos completed)
   - **Test**: Write failing test
   - **Implement**: Minimal code to pass
   - **Refactor**: Clean up
   - **Verify**: Run all tests (automated + manual from spec)
   - Mark todo completed
   - **Continue**: Move to next todo without stopping

6. **Integration Test**
   - Run full test suite: `npm test`
   - Run dev server: `npm run dev`
   - Complete ALL manual tests from iteration checklist
   - Test responsive (mobile + desktop) if UI
   - If any fail: fix and repeat

7. **Document**
   - Update `docs/implementation.md`:
     - Mark iteration ✅ COMPLETED
     - Add timestamp: "Completed: YYYY-MM-DD"
     - Note deviations from plan
     - List unresolved issues
   - List questions for next iteration (if any)
   - Update the `README.md` file to explain the current way to use and test the project

8. **Commit**
   - Stage changes
   - Commit: "feat: iteration $1 - [brief desc]"
   - Add changeset if needed

## Autonomy Rules

**Work continuously without stopping unless:**
- Blocked by missing information (unclear spec, ambiguous requirement)
- Blocked by external dependency (API key, missing package, environment issue)
- Critical error that prevents progress

**Do NOT stop or ask questions for:**
- Implementation details (make reasonable decisions)
- Code style choices (follow existing patterns)
- Minor architectural decisions (use best practices)
- Test naming or structure (use conventions)
- Variable/function naming (choose clear names)

**When blocked:**
- Try to resolve independently first (read docs, check existing code)
- If still blocked: ask ONE specific question
- Continue with unblocked tasks while waiting

## General Rules

- Each iteration = working, testable app
- Follow architecture in docs/implementation.md
- Never commit failing tests
- Always run full test suite before commit
- Make forward progress on every execution

## Output

Provide:
- What was built (1-2 sentences)
- Files created/modified
- Manual test results
- Next iteration recommendation
