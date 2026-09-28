# Report templates

Loaded by `om-judge-agent-session` workflow step 7. The judge report is a machine-consumed artifact (the `om-evolve-harness` skill reads it), so its headings are stable.

## Judge report

Use these headings in this order:

```markdown
# Agent Session Judge Report

## Verdict
pass | fail | inconclusive — one-sentence rationale

## Evidence
- Input: kind, identifier, schema/bundle version, hashes
- Rules: project/framework version, criteria file, and review skills
- Termination: completed | provider-limit | provider-error | user-abort | unknown — sanitized last-entry error summary or “none”
- Fixed attestations: <each required attestation> — pass | fail | stale | unavailable
- Privacy: pass | fail | unavailable

## 🔍 Artifact Findings
1. [severity] category — file:line or evidence path
   - Rule:
   - Evidence:
   - Fix:
   - Confidence:

## Design-Contract Review
Contract/references used, findings, or “not applicable — no design contract declared”.

## Harness-Owner Findings
1. Artifact finding reference
   - Smallest owner: root|guide|skill|facts|hook|case|oracle — path or ID
   - Escape reason:
   - Smallest harness fix:
   - Rerun:

## Missing or Unverifiable Evidence
- Exact missing/stale item and its effect on the verdict.

## Recommended Next Actions
1. Fix blocking artifact defects.
2. Improve the named harness owners.
3. Rerun the listed cases and fixed gates.
```

Omit empty numbered findings but retain every heading — consumers key on them. Do not include raw transcripts, secrets, absolute home paths, or claims that unavailable validation passed.
