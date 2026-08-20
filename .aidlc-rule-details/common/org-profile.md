# Organization Profile

**Purpose**: Organization-specific context that calibrates adaptive decisions across all AIDLC stages — how eagerly conditional stages trigger, which extensions are recommended by default, and how integrations behave. Load this file alongside common rules at workflow start and consult it wherever a stage makes an EXECUTE/SKIP judgment call or presents an extension opt-in question.

## Organization Context
- **Stage**: Early-stage startup
- **Team size**: Mid-size (multiple engineers, not solo)
- **Approval gates**: Standard — keep every existing "Wait for Explicit Approval" gate as defined in the base workflow. This profile never removes or auto-approves a gate; it only changes which CONDITIONAL stages get proposed as EXECUTE vs SKIP.
- **Risk posture**: Move fast on routine work, but don't skip the checkpoints that catch expensive-to-reverse mistakes (auth, payments, PII, production data, new infrastructure).

## Stage-Trigger Calibration

Conditional stages (User Stories, Application Design, Units Generation, Functional Design, NFR Requirements, NFR Design, Infrastructure Design) should default toward **SKIP** for typical day-to-day feature work at our scale. Raise the bar set by each stage's own "Execute IF" criteria as follows:

- **Application Design / Units Generation**: Only execute if the change introduces a genuinely NEW service/component boundary, touches 3 or more existing components, or the user explicitly asks for an architecture review. A single new endpoint, table, or internal module inside an existing service does not need this stage.
- **NFR Requirements / NFR Design / Infrastructure Design**: Only execute if the change touches auth, payments, PII, or introduces a new piece of infrastructure (new datastore, new external integration, new deployment target). Routine feature work on existing infra skips these.
- **User Stories**: Execute for anything customer-facing or involving more than one persona; skip for internal tooling, bug fixes, and refactors.

This calibration never changes the ALWAYS-execute stages (Workspace Detection, Requirements Analysis, Workflow Planning, Code Generation, Build and Test) and never removes an approval gate — the user can always override a SKIP by requesting the stage in the execution plan approval step.

## Extension Defaults (Recommendations Only)

These are *recommended* answers to surface alongside each extension's opt-in question. They do not change the question itself or skip asking it — the user can always answer differently:

- **Security Baseline**: Recommend **A) Yes**, even at prototype stage. Note that rules SECURITY-10 (SBOM), SECURITY-13 (CI/CD pipeline integrity), and SECURITY-14 (alerting dashboards) are commonly not yet applicable for a pre-production startup — mark those N/A with the rationale "not yet applicable at current stage" rather than treating them as blocking, until there's production traffic.
- **Resiliency Baseline**: Recommend **B) No** by default — revisit once there is production traffic and an on-call rotation.
- **Property-Based Testing**: Recommend **B) No** by default — opt in specifically for modules with complex algorithmic or parsing logic.

## Integration Configuration

- **Source of truth**: `aidlc-docs/` markdown in the GitHub repo remains canonical for every AIDLC artifact, always.
- **Jira/Confluence**: Optional mirror for stakeholder visibility — see the `integrations/tracking` extension. Jira issue creation/updates/comments are supported by the connected tools; Confluence page creation is NOT currently automatable (connected tools are read/search-only for Confluence) — treat that part as a manual copy-paste step until a Confluence-write tool is available.
- **Slack**: Optional gate notifications — see the `integrations/notifications` extension. Requires the Slack connector to be authorized (via claude.ai connector settings or `/mcp`) before it can actually post; until then, treat opt-in answers of "Yes" as pending and tell the user.
