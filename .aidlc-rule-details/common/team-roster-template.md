# Team Roster Template

Copy this file to `aidlc-docs/team-roster.md` the first time a PM-orchestrated AIDLC workflow runs in this project (if that file doesn't already exist), then fill in real team members before developer tagging begins.

## Developers

| Name | Jira Account Email | Skills/Tags | Role | Capacity Notes |
|---|---|---|---|---|
| [Full Name] | [email@company.com] | [e.g. backend, api, postgres] | [e.g. Senior Backend Engineer] | [e.g. Available / at capacity / out until DATE] |

**Instructions for the PM**:
- Add one row per developer who can be assigned work from this AIDLC workflow.
- Skills/Tags should be specific enough to match against requirement domains (e.g. "auth", "frontend-react", "infra-terraform", "mobile-ios", "data-pipeline").
- Keep this file up to date — the PM orchestrator persona uses it to propose developer tags on new requirements and questions, and to resolve a Jira `accountId` via email when creating and assigning tickets.
- If a developer's email doesn't resolve to a Jira account, the PM orchestrator will flag it rather than guessing.
