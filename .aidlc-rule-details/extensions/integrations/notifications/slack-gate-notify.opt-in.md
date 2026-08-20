# Slack Gate Notifications — Opt-In

**Extension**: Slack Gate Notifications

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: Slack Notifications
Should stage approval gates post a notification to Slack?

A) Yes — post a message to a Slack channel each time a stage is waiting for approval (recommended for teams reviewing async)

B) No — rely on this conversation only

X) Other (please describe after [Answer]: tag below)

[Answer]: 
```

If the answer is A, ask one follow-up question for the target Slack channel, and record it in `aidlc-docs/aidlc-state.md` under `## Integration Configuration`. This requires the Slack connector to be authorized first (via claude.ai connector settings or `/mcp`) — if it isn't authorized yet, tell the user and treat the answer as pending rather than posting anything.
