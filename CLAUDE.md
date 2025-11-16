# Architecture
- Hexagonal architecture (domain/application/infrastructure layers)
- Port/adapter pattern for external dependencies
- Use Zod for validation

# File Organization
- Domain: src/domain/
- Application: src/application/
- Infrastructure: src/infrastructure/
- Tests: **/__tests__/
- Feature specs: /backlog/

# Code Style
- Use ES modules (import/export), not CommonJS
- Destructure imports when possible

# Testing (TDD)
- Write tests BEFORE implementation
- Run single test files, not full suite
- Check test output for actual vs expected values
- Command: npm test -- path/to/test.test.ts

# Task Tracking (MANDATORY)
- Any task >2 steps → use TodoWrite immediately
- Mark in_progress BEFORE starting work
- Complete todos immediately after finishing (not batched)

# Error Handling
- Never assume env vars exist - check first
- Read FULL error before suggesting fixes
- Check package.json dependencies before imports

# Code Reading Strategy
- Use Serena MCP tools for code exploration (not Read on entire files)
- Start with symbols overview before reading full files
- Use find_symbol for specific functions/classes

# Agent Usage
- Use agents for exploratory work spanning >3 files
- Don't launch agents for single-file tasks
- Agents for: architecture review, multi-file refactoring, debugging

# Workflow
- Typecheck after code changes
- Feature workflow: spec → test → implement → typecheck → commit

# Git Conventions
- Branch prefix: jrob/
- Commits: "feat:", "spec:", "fix:", "test:"

# Common Commands

# Common Issues
- Check dependencies before imports
- Use absolute paths for file operations