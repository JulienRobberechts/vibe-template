# Personal tools or tips to use Claude

## Command

```
claude --dangerously-skip-permissions
```

## Tools

### [Serena MCP for Claude Code](https://github.com/oraios/serena)

Semantic code retrieval and intelligent code editing => Will boost your Claude using less token, and going faster.

My Serena Setup to do on each new project (See the doc for yours)

```
claude mcp add serena -- uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant --project "$(pwd)"
```

INFO: I had some trouble to make it work. I had to start it manually before for Claude to make it works. But now it's up and running it's great.

### [Claude Conversation Extractor tool](https://github.com/ZeroSumQuant/claude-conversation-extractor)

Tool to extract and save your Claude history. It could be useful sometimes.

### Advices


  ✅ What Works:
- Concise, imperative requests ("implement phase 5")
- Use Numbered task lists
- References to existing docs
- Running single tests
- Add commands documentation in claude.MD
- Specify the scope of your request
- Mixing multiple concerns in one request
- Do not mix multiple concerns in one request
- Do not switch between projects mid-conversation

## Notes

https://www.anthropic.com/engineering/claude-code-best-practices

1. Create CLAUDE.md files
2. If using GitHub, install the gh CLI
