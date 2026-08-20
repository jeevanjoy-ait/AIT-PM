# Jira/Confluence Sync Rules

## Overview
When enabled, this extension does three things, driven by the PM Orchestrator persona ([common/pm-orchestrator-persona.md](../../../common/pm-orchestrator-persona.md)):
1. **Developer-tagged Jira tickets**: every requirement, question, or user story (if that stage runs) can be assigned to a specific developer from the team roster, with a Jira ticket created and assigned accordingly.
2. **Answers flow back automatically**: when a delegated ticket is resolved, its answer is pulled back into the local docs and Confluence, and the workflow advances past the gate that was waiting on it — without anyone needing to come back and manually re-trigger it.
3. **Confluence publishing**: every markdown artifact AIDLC produces is mirrored to Confluence, matching the `aidlc-docs/` folder structure, with a confirm-before-publish step every time (one narrow exception — see Part 2).

`aidlc-docs/` markdown remains the canonical source of truth throughout — Jira and Confluence reflect it, they never originate content on their own.

---

## Part 1: Developer Tagging and Jira Tickets

### 1.1 Roster
Read `aidlc-docs/team-roster.md`. If it doesn't exist, create it from [common/team-roster-template.md](../../../common/team-roster-template.md) and stop — tell the human it must be filled in with real developers before tagging can happen. Never invent placeholder people.

### 1.2 Tagging point: Clarifying Questions (can delegate the answer itself)
When generating `requirement-verification-questions.md` (Requirements Analysis Step 6), add a `[Proposed Developer]:` line under each question alongside the existing `[Answer]:` tag, proposed by matching the question's subject matter against roster skills. The human now has two ways to close out a question, per question:
- **Answer it directly** — fill in `[Answer]:` themselves, the way this has always worked. No Jira/Confluence involvement for that item.
- **Delegate it** — leave `[Answer]:` blank and confirm (or edit) `[Proposed Developer]:`. This is a signal, not just a label: for every question left this way once the question doc is submitted, publish the question to its Confluence page (per Part 3) and create a Jira ticket assigned to that developer via `lookupJiraAccountId` + `createJiraIssue`, with the question (and its options, if multiple-choice) as the description, and an explicit instruction in the ticket: *"Reply with your answer as a comment, then move this ticket to Done."*

Record each delegated question's Jira ticket key and Confluence page/anchor against it in `aidlc-docs/aidlc-state.md` under `## Integration Configuration`, and see Part 2 for what happens once the developer resolves it.

**The `⛔ GATE: Await User Answers` in Requirements Analysis Step 6 is now satisfied when every question has EITHER a filled-in `[Answer]:` OR a resolved delegated ticket** — not only by direct answers as before.

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

## Part 2: Closing the Loop — Answers Flow Back from Jira

This is what makes delegation (1.2) actually work end to end: once a developer resolves their ticket, the answer needs to get back into the docs, into Confluence, and the workflow needs to keep moving — without the human who kicked this off having to sit there watching Jira.

**Scope, explicitly**: this closes the loop on the *answer-gate* only — the "is this question answered yet" wait. It does not touch the PM's own `Wait for Explicit Approval` gates (approving the finished requirements doc, the execution plan, a stage's completion, etc.). Those still require a human, in the conversation, every time — see [common/org-profile.md](../../../common/org-profile.md) and the base CLAUDE.md workflow. A Jira ticket moving to Done can unblock a *question*; it can never stand in for a PM's approval of a *deliverable*.

### 2.1 Set up the recurring check
The moment the first question or requirement gets delegated (1.2 or 1.3), set up a recurring check if one isn't already running for this project — use the `schedule` skill (or `CronCreate` directly). Default interval: **every 15 minutes**. Ask once, at this point, whether the human wants a different cadence — don't ask again after that. Cancel the recurring check once `aidlc-state.md` shows no open delegated tickets left; don't leave an idle poller running.

### 2.2 What each check does
For every ticket recorded in `aidlc-state.md` as an open delegated item:
1. Check its current status (`getJiraIssue`).
2. Still open → do nothing this round.
3. Moved to Done (or the project's equivalent resolved status) → read its most recent comment as the answer.
   - **No comment found**: don't guess. Leave it recorded as unresolved, flag it once (and post to Slack if that extension is enabled), and check again next round rather than treating "Done" alone as an answer.
   - **Comment found**: write it into the matching question's `[Answer]:` tag (or against the matching requirement, if this was a delegated requirement clarification) in the local file, then sync that specific update to its mirrored Confluence page.
4. Record the resolution (ticket key, who resolved it, timestamp) in `audit.md`, and mark the item resolved in `aidlc-state.md`.

**Exception to Part 3's confirm-before-publish rule**: the one Confluence write in step 3 above does *not* need a live confirmation. Every other Confluence publish in this extension waits for a human to confirm it, because the content is AI-assembled and someone should look at it before it goes out. This one is different — the content is the assignee's own words, and their act of resolving the ticket is already their approval of it going out. Don't generalize this exception beyond this exact case.

### 2.3 Advancing the gate
After each check, re-evaluate whether the stage's answer-gate is now fully satisfied — every question closed out, by direct answer or resolved delegation. If it is:
- Generate the next artifact the same way the stage normally would (e.g. `requirements.md`, once every clarifying question is answered) — this runs unattended, since it's mechanical assembly of already-collected answers, not a new judgment call.
- **Stop there.** Do not auto-approve the new artifact and do not skip its `Wait for Explicit Approval` gate. A human still reviews and approves it in the conversation, whenever they're next in it. If Slack notifications are enabled, this is exactly the moment worth pinging: *"requirements.md is ready for your review."*

### 2.4 Failure handling
If a resolved ticket gets reopened, reassigned, or its answer comment edited afterward, treat it as unresolved again on the next check — don't trust a stale answer. If Jira is unreachable for a given check, skip that round silently and try again next interval; a transient failure is not the same as "still unanswered."

---

## Part 3: Confluence Publishing

### 3.1 Write Method Detection (run once per project, before the first publish)

Confluence write access can come from two places, and which one is available depends on how the person running this workflow has their Atlassian connector configured — detect it, don't assume it:

1. **Probe for MCP write tools.** Search for Confluence write tools exposed by the connected Atlassian MCP server (e.g. `createConfluencePage`, `updateConfluencePage`, comment-write tools). If they're available, this connector already has the right Confluence access.
2. **Branch on what you find**:
   - **Found MCP write tools → Write Method = MCP (default).** Use them directly for every publish in this Part — skip the REST/curl path entirely. This is the preferred path whenever it's available: no local credentials to manage, and it's the same connector already handling Jira.
   - **Not found → tell the human plainly** that their current Atlassian connector only exposes read/search access to Confluence, and ask them to choose:
     - **A) Fix the connector** — reauthorize with broader scopes in their connector settings, or connect directly to Atlassian's official remote MCP server, then re-run detection. Pause here until they've done this or chosen the other route.
     - **B) Use the REST API fallback** — configure `CONFLUENCE_BASE_URL`, `CONFLUENCE_EMAIL`, `CONFLUENCE_API_TOKEN`, `CONFLUENCE_SPACE_KEY` per [common/integration-setup.md](../../../common/integration-setup.md), and proceed via the REST mechanics in 3.7. Verify the env vars are actually set before the first publish attempt — do not ask for the token value directly, and never write it to any file, log, or `audit.md` entry.
3. **Record the outcome** in `aidlc-docs/aidlc-state.md` under `## Integration Configuration` as `Confluence Write Method: MCP` or `Confluence Write Method: REST-API`, so this detection runs once per project, not once per publish. If a publish later fails because the recorded method stopped working (token expired, connector re-scoped), re-run detection and re-ask rather than silently giving up.

### 3.2 What gets published
Every markdown file created under `aidlc-docs/inception/`, `aidlc-docs/construction/`, and `aidlc-docs/operations/` gets mirrored as a Confluence page, immediately after that artifact is created — not batched at the end. `aidlc-state.md` and `audit.md` are internal tracking files and are NOT published.

### 3.3 Page hierarchy
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

### 3.4 Markdown → Confluence storage format
Convert the artifact's markdown to Confluence storage format before publishing:
- `#` / `##` / `###` → `<h1>` / `<h2>` / `<h3>`
- Bullet/numbered lists → `<ul><li>` / `<ol><li>`
- Bold/italic/inline code → `<strong>` / `<em>` / `<code>`
- Fenced code blocks → `<ac:structured-macro ac:name="code"><ac:plain-text-body><![CDATA[...]]></ac:plain-text-body></ac:structured-macro>`
- Tables → `<table><tbody><tr><td>...`
- **Mermaid diagrams**: Confluence does not render Mermaid natively. Publish the fenced source as a code block and note on the page that the diagram should be viewed in the repo — don't claim it renders.

This conversion doesn't need to be pixel-perfect — it needs to be readable. Note any content that didn't convert cleanly rather than silently dropping it.

### 3.5 Confirm before every publish
Before calling the Confluence API, show the human:
- The target page title and space
- Whether this creates a new page or updates an existing one
- A brief summary of what changed (for updates)

Wait for explicit confirmation before publishing. Confirm each page individually rather than batching multiple pages into one confirmation — each is a distinct piece of content going somewhere visible to others. **The one exception is the answer-sync write described in Part 2.2** — a resolved delegated ticket's answer publishes without this confirmation, since the assignee's own resolution of the ticket is the approval for that specific content.

### 3.6 Publish mechanics — MCP path (Write Method = MCP)
Call the discovered `createConfluencePage` / `updateConfluencePage` tool directly with the space key, title, parent page id (from 3.3), and body content. Check that tool's own declared parameters for the expected body format before calling — it may accept Markdown directly rather than requiring the storage-format conversion in 3.4; only run that conversion if the tool's schema calls for storage format. Everything else in this Part (hierarchy lookup, confirm-before-publish) applies the same way regardless of which path is active.

### 3.7 Publish mechanics — REST fallback (Write Method = REST-API)
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
