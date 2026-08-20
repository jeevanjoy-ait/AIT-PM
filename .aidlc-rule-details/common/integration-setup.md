# Integration Setup

**Purpose**: What a developer needs configured locally before the `integrations/tracking` extension can actually create Jira tickets and publish to Confluence. Referenced from that extension's rule files — read this once per machine, not once per project.

## Jira
Jira access goes through the already-connected Atlassian MCP tools — no separate credentials needed beyond having that connector authorized for your account (via claude.ai connector settings or `/mcp`). If `createJiraIssue`/`lookupJiraAccountId` calls fail with an auth error, the connector isn't authorized yet.

## Confluence
Confluence publishing does NOT go through the MCP connector — it's read/search-only for Confluence. Publishing uses Confluence's REST API directly via `curl`, authenticated with an API token. Set these environment variables (e.g. in your shell profile, or a local `.env.aidlc` file that is git-ignored — never commit them):

| Variable | Value |
|---|---|
| `CONFLUENCE_BASE_URL` | Your Atlassian site, e.g. `https://yourcompany.atlassian.net` |
| `CONFLUENCE_EMAIL` | The Atlassian account email the token belongs to |
| `CONFLUENCE_API_TOKEN` | An API token from https://id.atlassian.com/manage-profile/security/api-tokens |
| `CONFLUENCE_SPACE_KEY` | The target space key (e.g. `ENG`) |

**Recommended**: generate the token against a dedicated bot/service Atlassian account scoped to only the space(s) AIDLC should publish to, rather than a personal account — this bounds the blast radius if the token ever leaks.

Never paste the token value into a chat with the AI — it only needs to exist in your environment for the `curl` commands to pick it up.
