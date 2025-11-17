# Claude Code Configuration (Refactored 2025-11-17)

## Workflow Skills
- **workflow-implementation** (751 lines): TDD cycle, autonomy rules, integration testing
- **workflow-project** (824 lines): Spec management, iteration tracking, git workflow
- **workflow-agent-delegation** (723 lines): Agent selection, task mapping, delegation patterns

## Development Skills
- **dev-core-domain** (823 lines): Domain layer (entities, value objects, domain services)
- **dev-core-application** (737 lines): Application layer (use cases, ports, DTOs)
- **dev-adapter-db-core** (652 lines): Repository patterns, entity mapping, transactions
- **dev-adapter-db-prisma** (350 lines): Prisma ORM implementation
- **dev-adapter-db-drizzle** (111 lines): Drizzle ORM patterns
- **dev-adapter-db-raw** (374 lines): Raw SQL/Knex, query optimization

## Agents
- **dev-core**: Loads domain + application + workflow-implementation
- **dev-core-domain**: Domain layer only
- **dev-core-application**: Application layer only
- **dev-adapter-db**: Loads core + prisma + workflow-implementation
- **dev-adapter-db-prisma**: Prisma-specific

## Commands
All commands 25-36 lines, reference skills:
- **/imp** (36 lines): Implementation workflow
- **/feat-imp** (36 lines): Feature implementation
- **/fix** (35 lines): Bug fix workflow
- **/plan** (25 lines): Planning workflow
- **/feat-refine** (29 lines): Spec refinement

---

# Todo
- ask to load serena MCP server asap in the project developpement
- ask to update the readme of the project after each iteration
- find solution for permissions to not have to answer multiple time to the same permissions questions
- Add step to auto adjust errors and improve skills and commands
- the sub-agent do not stop and take all the iteration without going back to the parent.
- update claude.md at all level of the project
- Make a decision Log - Document why you chose specific libraries/patterns (prevents re-explaining)
- Build Verification - Script to verify dist/ structure
- Shorter Sessions - Commit more frequently for better resume points

# Done

- explicitly ask to use agent in developpement
- Refactored commands to 25-36 lines (51% reduction)
- Created 3 workflow skills (workflow-implementation, workflow-project, workflow-agent-delegation)
- Split dev-core into domain + application
- Split dev-adapter-db into core + prisma + drizzle + raw
- Created specialized agents (dev-core-domain, dev-core-application, dev-adapter-db-prisma)
