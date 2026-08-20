# Startup Engineering Standards

## Overview
Lightweight, MANDATORY cross-cutting engineering standards calibrated for an early-stage, mid-size team (see [common/org-profile.md](../../../common/org-profile.md)). They apply across all AIDLC phases and are always enforced — no opt-in — and are intentionally small in number so they don't slow down a small team, while still covering the mistakes that are expensive to walk back later.

**Enforcement**: At each applicable stage, verify compliance before presenting the stage completion message. Non-compliance is a blocking finding: list it under a "Standards Findings" section, withhold the "Continue to Next Stage" option until resolved, and log it in `aidlc-docs/audit.md` — the same blocking-finding behavior used by other extensions.

If a rule doesn't apply to the current change (e.g. ENG-01 when the change is Low risk), mark it **N/A** in the compliance summary — this is not a blocking finding.

---

## Rule ENG-01: Reversible Rollout for Risky Changes
**Rule**: Any change classified Medium risk or higher in Workflow Planning's Risk Assessment MUST ship behind a feature flag, a config toggle, or another trivially revertible mechanism (e.g. a single reversible migration).

**Verification**: The code generation plan names the specific rollback mechanism for any Medium+ risk change.

---

## Rule ENG-02: Test Coverage for New Business Logic
**Rule**: New or changed business logic MUST include automated tests covering the primary path plus at least one edge case. Pure config, copy, or style-only changes are exempt.

**Verification**: Code generation output includes tests alongside any new/changed logic; the completion message names what's covered.

---

## Rule ENG-03: No Secrets or Hardcoded Config in Source
**Rule**: No API keys, credentials, or environment-specific config MUST be committed to source. Use environment variables or a secrets manager.

**Verification**: No literal secret-shaped strings appear in generated code or IaC; config is read from environment/secrets manager.

---

## Rule ENG-04: Breaking Changes Called Out Explicitly
**Rule**: Any API, schema, or contract change that breaks backward compatibility MUST be explicitly flagged to the user — in the requirements doc or design doc — before code generation begins, not discovered after the fact.

**Verification**: The requirements or design doc contains an explicit "Breaking Changes" note when applicable, or states "None" when not.

---

## Enforcement Integration
Include an "Engineering Standards" compliance line (compliant/non-compliant/N/A per rule) in stage completion summaries for Requirements Analysis, Application Design, Units Generation, and Code Generation.
