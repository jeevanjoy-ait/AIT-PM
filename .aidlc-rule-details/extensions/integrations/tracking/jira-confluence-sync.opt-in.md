# Jira/Confluence Sync — Opt-In

**Extension**: Jira/Confluence Sync (PM Orchestrator Integration)

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: Jira/Confluence Sync
Should this work be tracked in Jira (with developer-assigned tickets) and mirrored to Confluence?

A) Yes — tag each requirement or question with a developer from the team roster; a tagged question can be delegated to Jira for that developer to answer directly (their answer flows back into the docs and Confluence automatically, and the workflow advances once it's in — no separate re-check needed); publish every AIDLC artifact to Confluence (confirmed before each publish, except a delegated answer flowing back, which publishes on its own since the developer's resolution is itself the approval)

B) No — keep everything as local markdown in aidlc-docs/ only

X) Other (please describe after [Answer]: tag below)

[Answer]: 
```

If the answer is A, and any of the following are missing, ask a single consolidated follow-up question for them before proceeding, then record all in `aidlc-docs/aidlc-state.md` under `## Integration Configuration`:
- Jira project key
- Confluence space key
- Whether `aidlc-docs/team-roster.md` exists and is filled in — create it from the template and pause if not (see [common/pm-orchestrator-persona.md](../../../common/pm-orchestrator-persona.md))
- **Confluence write access** — run the Write Method Detection in [jira-confluence-sync.md](jira-confluence-sync.md) Section 3.1: if the person's Atlassian connector already exposes Confluence write tools, use those (no further setup needed). If it doesn't, ask them to choose between fixing the connector's scopes and using the REST API fallback (which needs the env vars in [common/integration-setup.md](../../../common/integration-setup.md)) — don't default to skipping Confluence publishing without asking. Jira ticketing can proceed independently either way.

Note for the human: delegating a question's answer to Jira (rather than answering it directly) is only offered once you've confirmed A here — it's covered in full in [jira-confluence-sync.md](jira-confluence-sync.md) Parts 1.2 and 2, including the polling cadence question that comes up the first time you actually delegate one.
