# Personal tools or tips to use Claude

## Tools

### [Serena MCP for Claude Code](https://github.com/oraios/serena)

Semantic code retrieval and intelligent code editing => Will boost your Claude using less token, and going faster.

My Serena Setup to do on each new project (See the doc for yours)

```
claude mcp add serena -- uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant --project "$(pwd)"
```

INFO: I had some trouble to make it work. I had to start it manually before for Claude to make it works. But now it's up and running it's great.

### [Claude Code Usage Monitor](https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor)

Nice terminal for Claude Code usage. But the weekly limit is not diplayed.

### [Claude Conversation Extractor tool](https://github.com/ZeroSumQuant/claude-conversation-extractor)

Tool to extract and save your Claude history. It could be useful sometimes.

## Notes

https://www.anthropic.com/engineering/claude-code-best-practices

1. Create CLAUDE.md files
2. If using GitHub, install the gh CLI
