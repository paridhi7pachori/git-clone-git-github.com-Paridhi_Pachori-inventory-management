# Install GitHub App

Set up GitHub integration for this project: verify the GitHub MCP server is connected, ensure a GITHUB_TOKEN is configured, and confirm the repo is accessible.

## Steps

### 1 — Check MCP status

Run:
```
"/Users/paridhipachori/Library/Application Support/Claude/claude-code/2.1.111/claude.app/Contents/MacOS/claude" mcp list
```

If `github` shows `✓ Connected`, skip to Step 3.

If it shows `✗ Failed`, the token is likely missing — continue to Step 2.

### 2 — Configure GITHUB_TOKEN

The GitHub MCP server requires a personal access token with `repo` scope.

1. Open: https://github.com/settings/tokens/new
   - Note: scopes needed: `repo`, `read:org`
   - Expiration: your preference (90 days recommended)
2. Copy the generated token
3. Update `~/.claude.json` — find the `github` MCP server entry and add the token to its `env`:

```json
"github": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-token-here>"
  }
}
```

Use python3 to patch the file safely:
```bash
python3 -c "
import json
with open('/Users/paridhipachori/.claude.json') as f:
    d = json.load(f)
proj = '/Users/paridhipachori/Desktop/Git Repo/git-clone-git-github.com-Paridhi_Pachori-inventory-management'
d['projects'][proj]['mcpServers']['github']['env'] = {'GITHUB_PERSONAL_ACCESS_TOKEN': '<TOKEN>'}
with open('/Users/paridhipachori/.claude.json', 'w') as f:
    json.dump(d, f, indent=2)
print('Done')
"
```

Then restart Claude Code.

### 3 — Verify repo access

Use the GitHub MCP tool to confirm the repo is accessible:
- Call `mcp__github__get_file_contents` on `README.md` in this repo
- If it returns content, integration is working

Report the result to the user: connected status, repo name, and whether the token has write access.
