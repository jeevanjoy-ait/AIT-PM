# Jira/Confluence Sync Rules

## Overview
When enabled, this extension does two things, driven by the PM Orchestrator persona ([common/pm-orchestrator-persona.md](../../../common/pm-orchestrator-persona.md)):
1. **Developer-tagged Jira tickets**: every requirement (or user story, if that stage runs) is assigned to a specific developer from the team roster, and a Jira ticket is created and assigned accordingly.
2. **Confluence publishing**: every markdown artifact AIDLC produces is mirrored to Confluence, matching the `aidlc-docs/` folder structure, with a confirm-before-publish step every time.

`aidlc-docs/` markdown remains the canonical source of truth in both cases — Jira and Confluence reflect it, they never originate content.

---

## Part 1: Developer Tagging and Jira Tickets

### 1.1 Roster
Read `aidlc-docs/team-roster.md`. If it doesn't exist, create it from [common/team-roster-template.md](../../../common/team-roster-template.md) and stop — tell the human it must be filled in with real developers before tagging can happen. Never invent placeholder people.

### 1.2 Tagging point: Clarifying Questions (informational only)
When generating `requirement-verification-questions.md` (Requirements Analysis Step 6), add a `[Proposed Developer]:` line under each question, proposed by matching the question's subject matter against roster skills. This is informational only — it does not create a ticket yet, since answers aren't finalized. The human may edit it or leave it.

### 1.3 Tagging point: Finalized Requirements (creates tickets)
When generating `requirements.md` (Requirements Analysis Step 7), add an `[Assigned Developer]:` line under each requirement (functional and non-functional), carried forward from the matching question's proposed tag where one exists, or newly proposed. Before the Requirements Analysis approval gate closes, this is the human's last chance to correct a tag.

**Once requirements.md is approved**:
- Create one parent Jira issue (Epic or Story, depending on scope) for the overall unit of work via `createJiraIssue`, summarizing `requirements.md`.
- For each tagged requirement, create a child Jira issue (Task) via `createJiraIssue`, linked to the parent (`createIssueLink`), with:
  - Summary: the requirement text
  - Assignee: resolve the tagged developer's roster email to a Jira `accountId` via `lookupJiraAccountId`, then set it on the issue
  - If the email doesn't resolve to a Jira account, create the ticket unassigned and flag it to the human rather than guessing or skipping the ticket
- Record all created issue keys in `aidlc-docs/aidlc-state.md` under `## Integration Configuration`.

### 1.4 If User Stories executes
Tag and ticket per **user story** instead of per requirement, to avoid duplicate tickets for the same work — requirements.md items are already covered by their linked stories. The same tagging mechanics apply to `stories.md`.

### 1.5 Downstream updates
At Code Generation / Build and Test completion, add a comment (`addCommentToJiraIssue`) to the relevant child ticket(s) summarizing what was built, and transition the ticket (`transitionJiraIssue`) if a matching workflow transition exists (e.g. "In Review").

---

## Part 2: Confluence Publishing

### 2.1 Setup check
Before the first publish attempt, verify `CONFLUENCE_BASE_URL`, `CONFLUENCE_EMAIL`, and `CONFLUENCE_API_TOKEN` are set in the environment (see [common/integration-setup.md](../../../common/integration-setup.md)). If any are missing, tell the human once and skip Confluence publishing for the rest of this session — do not ask for the token value directly, and never write it to any file, log, or `audit.md` entry.

### 2.2 What gets published
Every markdown file created under `aidlc-docs/inception/`, `aidlc-docs/construction/`, and `aidlc-docs/operations/` gets mirrored as a Confluence page, immediately after that artifact is created — not batched at the end. `aidlc-state.md` and `audit.md` are internal tracking files and are NOT published.

### 2.3 Page hierarchy
Mirror the `aidlc-docs/` folder structure as a Confluence page tree under one root page for the project/unit of work:
```
[Project Name] (root page)
├── Inception
│   ├── Requirements
│   ├── User Stories (if applicable)
│   ├── Execution Plan
│   └── Application Design (if applicable)
└── Construction
    └── [Unit Name]
        ├── Functional Design (if applicable)
        ├── NFR Requirements / Design (if applicable)
        └── Build and Test
```
Look up existing pages first (`getPagesInConfluenceSpace`, `searchConfluenceUsingCql`) to find the right parent `ancestors` id rather than creating a duplicate hierarchy.

### 2.4 Markdown → Confluence storage format
Convert the artifact's markdown to Confluence storage format before publishing:
- `#` / `##` / `###` → `<h1>` / `<h2>` / `<h3>`
- Bullet/numbered lists → `<ul><li>` / `<ol><li>`
- Bold/italic/inline code → `<strong>` / `<em>` / `<code>`
- Fenced code blocks → `<ac:structured-macro ac:name="code"><ac:plain-text-body><![CDATA[...]]></ac:plain-text-body></ac:structured-macro>`
- Tables → `<table><tbody><tr><td>...`
- **Mermaid diagrams**: Confluence does not render Mermaid natively. Publish the fenced source as a code block and note on the page that the diagram should be viewed in the repo — don't claim it renders.

This conversion doesn't need to be pixel-perfect — it needs to be readable. Note any content that didn't convert cleanly rather than silently dropping it.

### 2.5 Confirm before every publish
Before calling the Confluence API, show the human:
- The target page title and space
- Whether this creates a new page or updates an existing one
- A brief summary of what changed (for updates)

Wait for explicit confirmation before publishing. Confirm each page individually rather than batching multiple pages into one confirmation — each is a distinct piece of content going somewhere visible to others.

### 2.6 REST mechanics
Use Confluence Cloud's REST API via `curl`, authenticating with Basic auth (email + API token) read from the environment. Never inline the token as a literal value outside normal env-var expansion, and never echo it.

Check if the page exists:
```bash
curl -s -u "${CONFLUENCE_EMAIL}:${CONFLUENCE_API_TOKEN}" \
  "${CONFLUENCE_BASE_URL}/wiki/rest/api/content?spaceKey=${CONFLUENCE_SPACE_KEY}&title=${TITLE_URLENCODED}&expand=version"
```

Create (if it doesn't exist) — write the JSON body to a temp file in the scratchpad directory first and pass it via `-d @file.json` rather than inlining large content on the command line:
```bash
curl -s -u "${CONFLUENCE_EMAIL}:${CONFLUENCE_API_TOKEN}" \
  -X POST -H "Content-Type: application/json" \
  "${CONFLUENCE_BASE_URL}/wiki/rest/api/content" \
  -d @create-page.json
```
where `create-page.json` contains:
```json
{
  "type": "page",
  "title": "TITLE",
  "space": {"key": "SPACE_KEY"},
  "ancestors": [{"id": "PARENT_PAGE_ID"}],
  "body": {"storage": {"value": "STORAGE_FORMAT_CONTENT", "representation": "storage"}}
}
```

Update (if it exists — note the version bump) the same way with `-X PUT` against `${CONFLUENCE_BASE_URL}/wiki/rest/api/content/${PAGE_ID}`, including `"id"` and `"version": {"number": CURRENT_VERSION_PLUS_ONE}` in the JSON body.

---

## Failure Handling
Jira and Confluence calls can fail independently (permissions, missing project/space, connector not authorized, network). Report each failure once with the specific error and continue the AIDLC stage regardless — this extension supports the team's visibility, it does not gate their ability to ship.
