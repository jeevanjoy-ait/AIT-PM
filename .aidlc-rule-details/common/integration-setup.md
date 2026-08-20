# Integration Setup

**Purpose**: What a developer needs configured locally before the `integrations/tracking` extension can actually create Jira tickets and publish to Confluence. Referenced from that extension's rule files — read this once per machine, not once per project.

## Jira
Jira access goes through the already-connected Atlassian MCP tools — no separate credentials needed beyond having that connector authorized for your account (via claude.ai connector settings or `/mcp`). If `createJiraIssue`/`lookupJiraAccountId` calls fail with an auth error, the connector isn't authorized yet.

## Confluence
Whether Confluence publishing goes through your MCP connector or needs the REST fallback below depends on how your Atlassian connector is scoped — some connectors expose Confluence write tools (`createConfluencePage`/`updateConfluencePage`), others only expose read/search. The `integrations/tracking` extension detects which one you have automatically (see `jira-confluence-sync.md` Section 2.1) and asks you to choose a path if write tools aren't available:
- **If your connector has write access**: nothing further to configure — it's used directly, no env vars needed.
- **If it doesn't, and you'd rather fix the connector**: reauthorize with broader scopes in your connector settings, or connect directly to Atlassian's official remote MCP server, then let the extension re-detect.
- **If you'd rather use the REST fallback instead**: it publishes via Confluence's REST API directly via `curl`, authenticated with an API token. Set these environment variables (e.g. in your shell profile, or a local `.env.aidlc` file that is git-ignored — never commit them):

| Variable | Value |
|---|---|
| `CONFLUENCE_BASE_URL` | Your Atlassian site, e.g. `https://yourcompany.atlassian.net` |
| `CONFLUENCE_EMAIL` | The Atlassian account email the token belongs to |
| `CONFLUENCE_API_TOKEN` | An API token from https://id.atlassian.com/manage-profile/security/api-tokens |
| `CONFLUENCE_SPACE_KEY` | The target space key (e.g. `ENG`) |

**Recommended**: generate the token against a dedicated bot/service Atlassian account scoped to only the space(s) AIDLC should publish to, rather than a personal account — this bounds the blast radius if the token ever leaks.

Never paste the token value into a chat with the AI — it only needs to exist in your environment for the `curl` commands to pick it up.
