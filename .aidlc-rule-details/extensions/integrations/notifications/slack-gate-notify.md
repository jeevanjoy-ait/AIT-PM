# Slack Gate Notification Rules

## Overview
When enabled, every "Wait for Explicit Approval" gate defined across the AIDLC stages ALSO posts a short notification to the configured Slack channel, in addition to — never instead of — presenting the gate in this conversation. This conversation remains authoritative; Slack is a heads-up, not a second approval path.

## Behavior
At each gate (Requirements Analysis, User Stories, Workflow Planning, Application Design, Units Generation, each per-unit Construction stage, Code Generation, Build and Test):
1. Present the gate in-conversation as normal (unchanged).
2. Post one message to the configured Slack channel: stage name, a one-line summary, and the doc path to review. Do not post full document content.
3. Do not wait for a Slack response — approval must still come back through this conversation.

## Failure Handling
If the Slack connector is not authorized or the post fails, note it once to the user and continue without retrying on every subsequent gate.
