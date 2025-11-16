  Immediate Actions (High Impact)

  1. Enhance Project CLAUDE.md Files

  Add to /home/dev1/proj/vibe-template/CLAUDE.md:
  # Testing
  - npm test: Run all tests
  - npm test -- path/to/test.spec.ts: Run specific test
  - npm test -- --watch: Watch mode

  # Project Structure
  - src/: Source code
  - __tests__/ or *.test.ts: Test files
  - dist/: Build output

  2. Create TROUBLESHOOTING.md

  Documents recurring issues:
  - ES module import errors in extensions
  - Test module resolution failures
  - Build output verification

  3. Strengthen TDD Instructions

  Update ~/.claude/CLAUDE.md:
  ## TDD Workflow (MANDATORY)
  1. Write failing test FIRST
  2. Verify it fails
  3. Write minimal code to pass
  4. Verify it passes
  5. Refactor
  6. Commit

  Never implement before test exists.

  4. Add Quick Reference Docs

  For complex projects (ts-pure-analyser), create /docs/QUICK-REFERENCE.md with:
  - Valid enum/type values
  - Common patterns
  - API syntax examples

  Patterns Identified

  ✅ What Works:
  - Concise, imperative requests ("implement phase 5")
  - Numbered task lists
  - References to existing docs
  - Running single tests (you already do this)

  ❌ Common Issues:
  - Missing test commands documentation
  - Ambiguous scope ("add more debugging")
  - Mixing multiple concerns in one request
  - Context switching between projects mid-conversation

  Strategic Recommendations

  1. Per-Project CLAUDE.md - Each project should have architecture overview, key commands, debugging
  tips
  2. Decision Log - Document why you chose specific libraries/patterns (prevents re-explaining)
  3. Build Verification - Script to verify dist/ structure
  4. Shorter Sessions - Commit more frequently for better resume points

  Most Frequent Pain Points

  1. Repetitive questions about test commands → Fix with #1
  2. Same errors recurring → Fix with #2
  3. TDD not consistently followed → Fix with #3
  4. Type lookups slow iteration → Fix with #4