Refine the feature spec. Work autonomously without stopping unless blocked.

## Workflow

1. **Read Feature**
   - Find file matching `{{ARGUMENTS}}` in `/backlog/`
   - Read and understand current state

2. **Format & Clarify**
   - Apply template from `/backlog/_feature-template.md`
   - Fill sections: objectives, user stories, requirements, acceptance criteria, tasks, manual tests, deliverables
   - Remove unnecessary sections
   - Leave "Technical Implementation" empty

3. **Research Solutions**
   - Analyze codebase patterns
   - Research best practices
   - Consider constraints

4. **Propose Approaches**
   - List 3+ technical approaches
   - For each: method, pros, cons, effort
   - Rank by simplicity, maintainability, testability

5. **Ask User**
   - Present approaches clearly
   - Recommend preferred + rationale
   - Ask: "Which approach?"

6. **Document Choice**
   - Update "Technical Implementation":
     - Approach
     - Files to create/modify
     - Technical hints
   - Complete open questions

7. **Verify Completeness**
   - All sections filled
   - No ambiguity
   - Ready for implementation

8. **Commit**
   - Stage feature file
   - Add commit message: "spec: feat [title]"
   - Ask user to commit

## Autonomy Rules

**Work continuously unless:**
- Feature file missing
- Requirements fundamentally unclear
- Need user decision on approach

**Do NOT ask about:**
- Template formatting
- Wording improvements
- Minor clarifications

## Output

- What was refined (1 sentence)
- Proposed approaches (brief)
- Recommendation
- Question: approach selection
