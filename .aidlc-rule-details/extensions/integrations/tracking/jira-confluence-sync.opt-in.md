# Jira/Confluence Sync — Opt-In

**Extension**: Jira/Confluence Sync (PM Orchestrator Integration)

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: Jira/Confluence Sync
Should this work be tracked in Jira (with developer-assigned tickets) and mirrored to Confluence?

A) Yes — tag each requirement with a developer from the team roster, create/assign Jira tickets accordingly, and publish every AIDLC artifact to Confluence (confirmed before each publish)

B) No — keep everything as local markdown in aidlc-docs/ only

X) Other (please describe after [Answer]: tag below)

[Answer]: 
```

If the answer is A, and any of the following are missing, ask a single consolidated follow-up question for them before proceeding, then record all in `aidlc-docs/aidlc-state.md` under `## Integration Configuration`:
- Jira project key
- Confluence space key
- Whether `aidlc-docs/team-roster.md` exists and is filled in — create it from the template and pause if not (see [common/pm-orchestrator-persona.md](../../../common/pm-orchestrator-persona.md))
- Whether `CONFLUENCE_BASE_URL`, `CONFLUENCE_EMAIL`, and `CONFLUENCE_API_TOKEN` are set in the environment (see [common/integration-setup.md](../../../common/integration-setup.md)). If not set, tell the user Confluence publishing will be skipped until they are — Jira ticketing can proceed independently.
