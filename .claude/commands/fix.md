# Fix Failing Tests

Systematically fix failing tests in the project using a structured approach.

## Workflow

1. **Discover Failing Tests**
   - Run test suite to identify failures
   - Group failures by type/pattern
   - Prioritize based on:
     - Root cause tests (others may be cascading failures)
     - Critical functionality
     - Test complexity

2. **For Each Failing Test**

   a) **Analyze Failure**
      - Read test code and understand intent
      - Examine error message/stack trace
      - Identify root cause:
        - Implementation bug
        - Test expectation mismatch
        - Missing functionality
        - Configuration issue

   b) **Plan Fix**
      - Implementation fix: Modify source code to pass test
      - Test fix: Adjust test expectations if requirements changed
      - Explain approach clearly
      - Consider side effects on other tests

   c) **Confirm & Execute**
      - Present plan to user
      - Wait for confirmation
      - Apply changes
      - Verify fix passes
      - Check for regressions

3. **Batch Processing Options**
   - Fix similar failures together
   - Skip tests temporarily if blocked
   - Mark tests for follow-up

## Usage

```bash
/fix                          # Fix all failing tests
/fix --file <test-file>       # Fix specific test file
/fix --pattern <name>         # Fix tests matching pattern
/fix --batch                  # Auto-fix similar issues
```

## Best Practices

- Run full test suite before starting
- Fix one test at a time unless batching similar issues
- Verify each fix doesn't break other tests
- Document non-obvious fixes
- Consider adding regression tests

