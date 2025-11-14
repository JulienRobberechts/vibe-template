# Plan Implementation

1. Find and read business specifications (`/specs/version-X.md`) and technical specifications (`/specs/tech-specs.md`)

2. Create multi-iteration implementation plan in `/docs/implementation.md`:

If a sub-agent seems able to do this job, delegate the job to this sub-agent (example: backend-architect)

**Structure:**
- Architecture decisions (file structure, module boundaries)
- Iterations (sequential, one per feature/feature group):
  - Iteration N: Feature name
    - Scope (what gets built)
    - TDD phases: Plan → Test → Implement → Refactor → Review
    - Deliverables (working code, tests, docs)
    - Dependencies (what must be done first)
    - Unresolved questions (if any)

**Planning Principles:**
- Design for incremental delivery (each iteration = working feature)
- Follow TDD strictly (write tests before implementation)
- Start simple: core features first, skip optional unless requested
- Only ask questions if genuinely ambiguous (prefer best-practice defaults)
- Order iterations by dependency (foundation → features → polish)

3. Show plan summary and ask user to review
