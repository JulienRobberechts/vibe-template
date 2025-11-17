# Claude Code Configuration (Refactored 2025-11-17)

## Workflow Skills
- **workflow-implementation** (751 lines): TDD cycle, autonomy rules, integration testing
- **workflow-project** (824 lines): Spec management, iteration tracking, git workflow
- **workflow-agent-delegation** (723 lines): Agent selection, task mapping, delegation patterns

## Development Skills
- **dev-core-domain** (823 lines): Domain layer (entities, value objects, domain services)
- **dev-core-application** (737 lines): Application layer (use cases, ports, DTOs)
- **dev-adapter-db-core** (empty): Repository patterns - INCOMPLETE
- **dev-adapter-db-prisma** (empty): Prisma-specific - INCOMPLETE
- **dev-adapter-db-drizzle** (empty): Drizzle ORM - INCOMPLETE
- **dev-adapter-db-raw** (empty): Raw SQL/Knex - INCOMPLETE

## Agents
- **dev-core**: Loads domain + application + workflow-implementation
- **dev-core-domain**: Domain layer only
- **dev-core-application**: Application layer only
- **dev-adapter-db**: Loads core + prisma + workflow-implementation (BROKEN - skills missing)
- **dev-adapter-db-prisma**: Prisma-specific (BROKEN - skills missing)

## Commands
All commands 25-36 lines, reference skills:
- **/imp** (36 lines): Implementation workflow
- **/feat-imp** (36 lines): Feature implementation
- **/fix** (35 lines): Bug fix workflow
- **/plan** (25 lines): Planning workflow
- **/feat-refine** (29 lines): Spec refinement

## Known Issues
⚠️ dev-adapter-db split incomplete - directories exist but SKILL.md files missing
⚠️ Original dev-adapter-db/SKILL.md deleted in f5a5852

---

# Todo

- Complete dev-adapter-db skill split (create SKILL.md files)
- ask to load serena MCP server at one point in the project
- ask to update the readme of the project as iteration 0 and after each iteration
- find solution for permissions
- Add step to auto adjust errors and improve skills and commands
- the sub-agent do not stop and take all the iteration without going back to the parent.
- update claude.md at all level of the project
- Make a Decision Log - Document why you chose specific libraries/patterns (prevents re-explaining)
- Build Verification - Script to verify dist/ structure
- Shorter Sessions - Commit more frequently for better resume points

# Done

- explicitly ask to use agent in developpement
- Refactored commands to 25-36 lines (51% reduction)
- Created 3 workflow skills
- Split dev-core into domain + application
- Created specialized agents