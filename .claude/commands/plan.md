# Plan Implementation

**Workflow:**

1. **Explore context** (in parallel if possible):
   - Read specs: `/specs/business-specs.md`, `/specs/tech-specs.md`
   - Explore codebase structure (use Task tool with Explore agent for quick overview)
   - Read relevant existing code

2. **Choose planning approach:**
   - **Architectural planning** (system design, folder structure, module boundaries) → Use `backend-architect` agent
   - **Feature implementation planning** (specific features, components, use cases) → Plan directly

3. **Create plan** in `/docs/implementation.md`:

**Structure:**
```markdown
# Implementation Plan: [Feature/System Name]

## Architecture
- File structure
- Module boundaries
- Key design decisions

## Iterations

**Agent:** Specify if parallelizable via sub-agent (e.g., dev-core, react-dev, dev-api)

### Iteration 1: [Feature Name]
**Scope:** What gets built (1-2 sentences)

**TDD Phases:**
- Plan: Design approach
- Test: Write failing tests
- Implement: Make tests pass
- Refactor: Clean up
- Review: Check quality

**Deliverables:** Working code, tests, docs

**Dependencies:** What must exist first

**Questions:** Unresolved items (if any)

[Repeat for each iteration...]
```

**Planning Principles:**
- Incremental delivery (each iteration = working feature)
- TDD mandatory (tests before code)
- Dependency order (foundation → features → polish)
- Simple first (skip optional unless requested)
- Assume best practices (only ask if genuinely ambiguous)

4. **Present plan:**
   - Show concise summary
   - List unresolved questions
   - Use ExitPlanMode tool to get user approval
