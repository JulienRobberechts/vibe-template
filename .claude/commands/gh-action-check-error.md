Analyze GitHub Actions workflow run. Args: [run-id|workflow-name] (optional, defaults to latest)

Steps:
1. Get run details: `gh run view [run-id] --log-failed` or `gh run list --limit 1`
2. If success: report status, done
3. If failed: identify failing job/step, extract error
4. Determine root cause:
   - **Repo code issue**: Create issue with `gh issue create` (title: brief error, body: logs + context), then suggest `/gh-fix-issue {issue-number}`
   - **Infra/config issue**: Explain fix, don't create issue
   - **Transient**: Suggest `gh run rerun [run-id]`

Use `gh` for all GitHub ops.
