# md2sign Claude Code plugin

This plugin mirrors the md2sign MCP server at `https://md2sign.com/mcp`. The server (in the private md2sign app repo) is the source of truth for tool names, arguments, error codes, prices and limits. The skills and READMEs here must describe exactly what that server does.

When the server changes, update:
- `plugins/md2sign/skills/contracts/SKILL.md`: markdown format, workflow, Pro tools, error table
- `plugins/md2sign/skills/send/SKILL.md`: the guided send flow
- `README.md` and `plugins/md2sign/README.md`: setup, pricing and the free first document

Then bump `version` in `plugins/md2sign/.claude-plugin/plugin.json` (users only get updates when it changes), run `claude plugin validate ./plugins/md2sign --profile directory`, and push. To check the current server surface, call `tools/list` on `https://md2sign.com/mcp`, or read `tests/fixtures/mcp-surface.json` in the app repo.

Never put API keys in this repo. The key comes from the sensitive `api_key` userConfig option.
