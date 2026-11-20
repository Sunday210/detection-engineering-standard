# Detection Review Checklist

## Requirement and investigation
- [ ] Requirement ID, requester, reason, scope, exceptions, and evidence exist.
- [ ] Priority and triage rationale are recorded.
- [ ] Existing rules and coverage were reviewed.
- [ ] ATT&CK mapping describes the activity actually detected.
- [ ] Data source, dataset, required fields, and field semantics were verified from actual events.
- [ ] Telemetry completeness, quality, timeliness, and coverage were assessed.
- [ ] Threat research and assumptions are referenced.

## Rule logic and implementation
- [ ] Rule ID is stable and unique.
- [ ] Query is readable and scoped to the intended behavior.
- [ ] Query does not rely on volatile fields without justification.
- [ ] Exclusions are documented, limited, and owned.
- [ ] Scheduling, look-back, suppression, and alert grouping are appropriate.
- [ ] Performance and data-volume impact have been considered.
- [ ] No secrets or unnecessary sensitive data are embedded.

## Tests and validation
- [ ] Positive test exists and passes.
- [ ] Negative / known-good test exists and passes.
- [ ] Relevant boundary and variation tests exist.
- [ ] Missing or malformed fields are handled intentionally.
- [ ] Validation report records simulation execution and SIEM alert correlation.
- [ ] False positives, false negatives, and limitations are documented.

## Documentation and SOC usability
- [ ] Description explains why the rule fires.
- [ ] Data sources, rule format, references, maturity, severity, and confidence are recorded.
- [ ] Investigation suggestions and evidence requirements are actionable.
- [ ] Remediation guidance and approval requirements are clear.
- [ ] Related detections and dependencies are linked.

## Release and operations
- [ ] CI checks pass.
- [ ] Independent peer review is complete.
- [ ] Experimental / test / stable maturity is correctly assigned.
- [ ] Rollback / disable procedure exists.
- [ ] Deployment record is complete.
- [ ] Owner, monitoring, and review date are assigned.
- [ ] SOAR automation tier and required approvals are explicit, if applicable.
