# Detection Specification

- **Detection ID:** DET-<DOMAIN>-NNN
- **Requirement ID:** REQ-DE-YYYY-NNN
- **Name:**
- **Author / Owner:**
- **Status / Maturity:** DEVELOPMENT
- **Priority:**
- **Created:** YYYY-MM-DD
- **Last updated:** YYYY-MM-DD

## 1. Detection objective
What exact behavior, indicator, sequence, or statistical condition is being identified?

## 2. Threat context
- Threat / campaign / tool:
- ATT&CK tactic(s):
- ATT&CK technique(s) and sub-techniques:
- Indicator type: STATIC / BEHAVIORAL / STATISTICAL / HYBRID
- Research references and dates:
- Confidence / assumptions:

## 3. Scope
- Assets / data domains:
- Environment:
- In-scope systems:
- Out-of-scope systems:

## 4. Telemetry requirements
| Source / dataset | Event type / ID | Required fields | Actual field names | Normalization | Available? |
|---|---|---|---|---|---|
| | | | | | |

- Ingestion path:
- Expected latency:
- Retention requirement:
- Known gaps / data engineering tasks:

## 5. Detection logic design
- Query / correlation approach:
- Sequence or aggregation logic:
- Required joins / enrichment / lookup data:
- Threshold / time window:
- Exclusions and rationale:
- Expected output fields:
- Performance considerations:

## 6. False positives and edge cases
| Scenario | Expected outcome | Rationale / owner |
|---|---|---|
| | | |

## 7. Validation plan
| Test ID | Positive / Negative / Edge | Procedure / dataset | Expected result | Evidence location |
|---|---|---|---|---|
| | | | | |

## 8. Alert and SOC guidance
- Severity:
- Confidence:
- Why this alert fires:
- Initial triage steps:
- Evidence to collect:
- Related detections:
- Escalation criteria:
- Recommended response:
- Required approval:

## 9. Release plan
- Experimental scope:
- Test-stage reviewer(s):
- Stable-release approver:
- Rollback / disable procedure:
- Monitoring owner:
- Review cadence / next review date:

## 10. Limitations
Document procedures, systems, fields, or variants not covered by this detection.
