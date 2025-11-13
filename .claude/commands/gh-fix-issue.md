Analyze and fix GitHub issue: $ARGUMENTS

Use TDD workflow (Plan → Test → Implement → Refactor → Review):

**Phase 1: Plan**
- `gh issue view $ARGUMENTS` - get issue details
- Search codebase for affected code
- Propose 2-3 solutions, ask developer to choose
- List unresolved questions if any

**Phase 2: Test**
- Write failing test demonstrating bug
- Run test suite to confirm failure

**Phase 3: Implement**
- Create branch: `jrob/issue-$ARGUMENTS-short-desc`
- Implement chosen solution
- Verify tests pass

**Phase 4: Refactor**
- Run linting/type checking: `npm run lint && npm run typecheck`
- Fix any issues

**Phase 5: Review**
- Add changeset in `.changeset/0000-short-desc.md`
- Commit with concise message
- Push: `git push -u origin HEAD`
- Create PR: `gh pr create`
- Link issue in PR description

Use `gh` for all GitHub ops.