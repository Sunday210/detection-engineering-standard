# Detection Documentation Record

> Complete one copy for every detection. Store the completed copy beside the rule, tests, and validation evidence in the detection repository. This record is intended for both SOC analysts reviewing alerts and detection engineers maintaining or reusing the rule.
>
> The fields marked **[Book — Chapter 8]** reproduce the categories and property names of the documentation template in *Practical Threat Detection Engineering*, Table 8.1 (printed pages 171–173). Fields marked **[Project extension]** are additional controls proposed for this SIEM project.

## 1. Maintenance information

| Property | Value |
|---|---|
| Author **[Book]** | `{{author}}` |
| Creation Date **[Book]** | `{{YYYY-MM-DD}}` |
| Revision History **[Book]** | `{{date — what changed}}` |
| Maturity **[Book]** | `Experimental / Test / Stable` |
| License **[Book]** | `{{license or internal-only}}` |
| Ticket **[Book]** | `{{issue / requirement ID}}` |
| References **[Book]** | `{{threat reports, research, incident reports, vendor analysis}}` |
| Stable Detection ID **[Project extension]** | `{{DE-XXXX}}` |
| Owner / Team **[Project extension]** | `{{accountable owner}}` |
| Requirement ID and Priority **[Project extension]** | `{{REQ-ID / priority / rationale}}` |
| Created / Last Reviewed **[Project extension]** | `{{dates}}` |
| Review Due / Expiry **[Project extension]** | `{{date; especially important for volatile IoCs}}` |

## 2. What the detection alerts on

| Property | Value |
|---|---|
| Name **[Book]** | `{{descriptive activity name}}` |
| Description **[Book]** | `{{what activity is being detected and why it matters}}` |
| MITRE TTPs **[Book]** | `{{ATT&CK tactic / technique IDs and names; distinguish direct from indirect mapping}}` |
| Tags **[Book]** | `{{searchable keywords, platform, environment, campaign, etc.}}` |
| The Detection **[Book]** | `{{rule file path and any runtime/configuration information needed to execute it}}` |
| Rule **[Book]** | `{{actual query / analytic, or an explicit link to the version-controlled rule}}` |
| Rule Format **[Book]** | `{{KQL / EQL / ES|QL / Sigma / other}}` |
| Data Sources / Indexes **[Book]** | `{{event types, log sources, datasets, indexes/data streams that feed the rule}}` |
| Indicator Type **[Project extension]** | `Static / Behavioral / Statistical / Hybrid` |
| Required Fields **[Project extension]** | `{{field names and expected meanings/types}}` |
| Schedule / Look-back **[Project extension]** | `{{interval, look-back window, suppression/throttle if relevant}}` |
| Related Detections **[Book]** | `{{rules that provide context, prerequisites, or duplicate/overlapping coverage}}` |

## 3. Information used to analyze the alert

### Related Detections **[Book]**
`{{Other alerts that could confirm or add context to this activity.}}`

### Investigation Suggestions **[Book]**
`{{What should the SOC analyst inspect next? Include relevant host, user, process, network, identity, cloud, or application pivots. Explain how to corroborate or disprove the alert.}}`

### Responses or Remediations **[Book]**
`{{Recommended actions for a confirmed true positive. Distinguish investigation advice from actions requiring authorization. Link the relevant incident-response or SOAR playbook.}}`

### False Positives **[Book]**
`{{Legitimate activity that may trigger this detection, how to recognize it, and any approved exclusions. Do not use broad exclusions without justification and an owner.}}`

### Severity **[Book]**
`{{Impact if this alert is a true positive: Low / Medium / High / Critical, with rationale.}}`

### Confidence **[Book]**
`{{How likely an alert from this rule is to represent the intended activity: Low / Medium / High, with rationale. Confidence is not the same as severity.}}`

## 4. Validation and operational notes [Project extension]

| Property | Value |
|---|---|
| Positive test cases | `{{known-bad samples / Atomic tests / simulation IDs}}` |
| Negative test cases | `{{known-good dataset and benign scenarios}}` |
| Edge cases / variations | `{{procedure variants, boundary values, missing or malformed fields}}` |
| Validation report | `{{path or link to report}}` |
| Last validated | `{{YYYY-MM-DD}}` |
| Known limitations | `{{visibility limits, ATT&CK procedure coverage limits, assumptions}}` |
| Performance considerations | `{{query cost, high-cardinality fields, expected event/alert volume}}` |
| Alert routing | `{{SOC queue / case category / severity mapping}}` |
| SOAR eligibility | `Not eligible / Enrichment only / Case automation / Approved response` |
| Approval constraints | `{{human approval, asset exclusions, conditions, rollback}}` |
| Deployment / Rollback | `{{release reference and previous known-good version}}` |

## 5. Revision history

| Date | Version | Author | Change | Reviewer |
|---|---|---|---|---|
| `{{date}}` | `0.1` | `{{name}}` | Initial draft | `{{reviewer or pending}}` |

## 6. Completion checklist

- [ ] Maintenance metadata is complete and the rule has a stable ID and owner.
- [ ] Name and description explain the activity in language SOC analysts can understand.
- [ ] ATT&CK mapping describes what the rule actually detects; indirect mappings are identified as such.
- [ ] Rule format, rule content/path, data sources, indexes, and required fields are specified.
- [ ] Related detections and investigation suggestions are documented.
- [ ] False positives and exclusions are described with rationale.
- [ ] Severity and confidence are separately justified.
- [ ] Positive, negative, and relevant edge-case tests are linked.
- [ ] Known limitations, validation date, and review/expiry date are recorded.
- [ ] Response guidance and SOAR eligibility/approval constraints are clear.
- [ ] Documentation was reviewed together with the rule before Stable promotion.

---

**Source note:** The book groups detection documentation into information for maintaining the detection, information about what the detection alerts on, and information used to analyze the alert. The template above retains the Table 8.1 property set and adds project-specific traceability, testing, performance, and SOAR fields. See *Practical Threat Detection Engineering*, Chapter 8, “Documenting a detection,” printed pages 169–174.
