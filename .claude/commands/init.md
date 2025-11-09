# Initialize the project

Check if git is already initialized. If not:
- Run `git init`
- Create `.gitignore` if missing (common patterns for this project type)
- Stage all files: `git add .`
- Create initial commit: `git commit -m "Initial commit"`
- Create private GitHub repo using `gh repo create` (interactive or specify name)
- Push to main: `git push -u origin main`

If already initialized:
- Check git status
- Ask user if they want to create GitHub remote if none exists
- Push any uncommitted changes if requested

Always verify:
- Remote is properly configured
- Initial push succeeds
- Repo is private (confirm with user)
