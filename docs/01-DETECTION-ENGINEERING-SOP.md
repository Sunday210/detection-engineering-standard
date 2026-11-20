# Detection Engineering Standard Operating Procedure (SOP)

**Document ID:** DE-SOP-001  
**Status:** Draft baseline for project adoption  
**Applies to:** SIEM ingestion, detection content, alert generation, validation, and the hand-off to SOC / case management / SOAR  
**Primary platform:** Elastic Security (implementation adapter; lifecycle is platform-neutral)

---

## 1. Purpose

This SOP defines one repeatable process for turning a security need, threat-intelligence finding, incident lesson, or validation gap into a tested and maintainable detection in the SIEM.

The process must answer the following questions for every detection:

1. **Why** do we need this detection and which risk does it address?
2. **What** behavior, indicator, or statistical pattern should be detected?
3. **Where** will that activity be visible, and which telemetry fields are required?
4. **How** will the logic be implemented and tested?
5. **When** and under what controls will the detection be deployed?
6. **Who** will investigate the alert, and what evidence or response guidance will they receive?
7. **How** will we prove that the detection continues to work and remains useful?

The intended result is not simply a large collection of SIEM rules. It is a managed detection capability with traceable requirements, reliable telemetry, reproducible tests, documented ownership, controlled release, and feedback from operations.

## 2. Scope and operating model

This SOP applies to:

- Endpoint, server, identity, network, application, cloud, and security-product telemetry.
- Static indicator, behavioral, statistical, and hybrid detections.
- New detections, modifications, tuning, migrations, and decommissioning.
- Detection content developed in Git and deployed to Elastic Security or another approved detection platform.
- Alert enrichment, incident/case creation, and SOAR hand-off contracts.

It does not replace incident response procedures, vulnerability management, change management, data classification, or the organization's approval process for containment actions. Those processes must be linked where necessary.

### 2.1 Responsibilities

| Role | Core responsibility |
|---|---|
| Requirement requester | Describes the risk, expected behavior, scope, reason, evidence, and exceptions. |
| Detection engineer | Owns investigation, rule design, implementation, tests, documentation, deployment, and tuning. |
| Data / platform engineer | Provides or modifies collection, parsing, normalization, enrichment, storage, and pipeline reliability. |
| Threat intelligence / DFIR / Threat Hunting | Supplies adversary context, incident artifacts, TTPs, and findings that may become requirements. |
| SOC analyst | Reviews alerts, records dispositions, reports noise and missed activity, and follows investigation guidance. |
| Detection reviewer | Independently reviews correctness, test quality, documentation, risk, and operational impact. |
| SIEM / SOAR owner | Maintains platform integrations, permissions, deployment mechanisms, case routing, and response safeguards. |

For a small project, one person may hold several roles. The author of a detection should still obtain an independent review before the rule becomes stable.

### 2.2 System-level flow

The overall SIEM and response flow is:

```text
Sources
  -> Collection / Forwarding
  -> Parsing and Normalization
  -> Enrichment / Context
  -> Elasticsearch storage and search
  -> Detection rules / Correlation
  -> Alerts
  -> Triage and Investigation
  -> Case Management
  -> SOAR orchestration (where appropriate)
  -> Containment / Eradication / Recovery (authorized actions)
  -> Disposition and lessons learned
  -> New or updated detection requirements
```

**Book baseline:** the book focuses on the detection engineering lifecycle, detection sources, rules, validation, and performance management.  
**Project extension:** the end-to-end SIEM-to-SOAR flow above makes explicit the interfaces needed by this project's larger SIEM/SOAR system. Treat SOAR playbooks and response approvals as separate controlled artifacts, not as hidden side effects of a detection rule.

## 3. Core concepts and design principles

### 3.1 Model adversary activity before writing rules

Use frameworks such as MITRE ATT&CK to describe adversary tactics and techniques, and use the Unified Kill Chain as a high-level model for understanding how attack activity may progress. These models are guides rather than strict linear attack scripts; real intrusions may skip, repeat, or reorder stages.

Use ATT&CK to:

- Describe the intended adversary behavior.
- Find related techniques, software, groups, and public reporting.
- Identify telemetry that can reveal the behavior.
- Map detection and validation coverage.

Do not treat an ATT&CK tag as proof that the technique is covered. Coverage must be demonstrated against the behavior or procedure being claimed.

### 3.2 Select indicator types deliberately

| Indicator type | What it detects | Strength | Main limitation |
|---|---|---|---|
| Static | A fixed attribute such as a hash, IP, domain, URL, filename, or other exact value. | Fast to implement; useful during an active campaign. | Easy for an adversary to change; often short-lived. |
| Behavioral | A pattern of activity, relationships, sequence, process context, or technique. | Usually more durable than a single static value. | Requires better telemetry, research, and tuning. |
| Statistical | An aggregate, threshold, rate, or deviation from a baseline. | Can reveal unusual activity that has no fixed signature. | Requires baselines, time windows, and careful handling of legitimate variance. |
| Hybrid | A combination of static, behavioral, and/or statistical features. | Can provide stronger context and specificity. | More complex to build, test, explain, and maintain. |

The book's Pyramid of Pain discussion supports using static indicators as tactical signals while progressively investing in tools, behaviors, and TTPs where feasible. This is not a reason to discard IoCs: use them when they are relevant, but record their context, source, confidence, and expiry/review needs.

### 3.3 Balance four quality dimensions

A detection must be evaluated against:

- **Coverage:** which attack procedures or variations it can identify.
- **Durability:** how difficult it is for an adversary to bypass by changing an indicator or implementation detail.
- **Precision / noisiness:** how often alerts represent the intended malicious behavior versus benign activity.
- **Operational cost:** development, execution, storage, maintenance, and analyst investigation effort.

A rule that detects everything but floods the SOC is not automatically a good rule. A narrow rule with no false positives but many false negatives is not automatically a good rule either. The acceptable balance depends on severity, confidence, and the organization's risk tolerance.

## 4. Lifecycle overview and mandatory state transitions

The six primary phases are:

1. Requirements Discovery
2. Triage
3. Investigate
4. Develop
5. Test
6. Deploy

The phases are sequential at the workflow level, but testing and investigation can feed back to earlier stages. The program also performs four continuous activities: Monitoring, Maintenance, Metrics, and Comprehensive Validation.

### 4.1 Detection work-item states

Use the following states in the ticketing system or repository metadata:

```text
NEW
 -> TRIAGED (ACCEPTED | REJECTED | DEFERRED)
 -> INVESTIGATING
 -> READY_FOR_DEVELOPMENT
 -> DEVELOPING
 -> UNIT_TESTED
 -> VALIDATING
 -> EXPERIMENTAL
 -> TEST_REVIEW
 -> STABLE
 -> MAINTENANCE
 -> DEPRECATED / RETIRED
```

A state must not be skipped merely to release a rule faster. A documented exception may be approved for an emergency detection, but the exception must define an owner, expiry/review date, risk, and follow-up validation plan.

### 4.2 Phase gates

| Phase | Required output | Exit condition |
|---|---|---|
| Requirements Discovery | Complete detection requirement | Request has owner, reason, scope, evidence, exception context, and target behavior. |
| Triage | Priority and decision | Risk relevance and existing coverage have been assessed; accepted work is prioritized. |
| Investigate | Technical detection specification | Required telemetry, fields, behavior, research, and validation criteria are known or gaps are recorded. |
| Develop | Rule/query plus tests and documentation | Logic implements the design and passes unit tests against known-bad and known-good data. |
| Test | Validation report | Detection behavior, edge cases, false positives, false negatives, and operational impact have been reviewed. |
| Deploy | Versioned release and rollback record | Rule is deployed at an explicit maturity stage and monitored. |

---

## 5. Phase 1 — Requirements Discovery

**Objective:** discover and record what the organization needs to detect before choosing a query or tool.

### 5.1 Sources of requirements

A requirement may originate from:

- External threat intelligence (OSINT, vendor reporting, research, threat feeds, public incident write-ups).
- Internal threat intelligence, previous incidents, DFIR findings, and threat hunting.
- Business security requirements and acceptable-use policies.
- SOC requests, alert-quality issues, or observed detection gaps.
- Red Team / Purple Team exercises.
- Continuous program activities: production monitoring, maintenance, metrics, and validation.

Create an intake process that stakeholders can use consistently. A ticket, issue, or form is acceptable; the key requirement is that the request is traceable and contains enough context to investigate.

### 5.2 Minimum required fields

Every request must record:

1. **Requesting organization / requester:** team and contact for follow-up.
2. **Description:** what should be detected, expressed in plain language and/or technical language.
3. **Reason:** why the detection is needed and what risk, incident, threat, or policy it addresses.
4. **Exceptions:** expected benign situations that should not alert, or conditions requiring special handling.
5. **Scope:** relevant assets, users, networks, applications, cloud accounts, operating systems, and environments.
6. **Evidence:** logs, PCAP, files, hashes, screenshots, incident references, threat reports, or reproduction steps.

Additional project-required fields: proposed owner, date received, related ticket, data classification, business criticality, requested due date (if any), and links to related detections.

Use [`../templates/detection-requirement.md`](../templates/detection-requirement.md).

### 5.3 Intake checks

Before triage, confirm:

- The requested activity is sufficiently clear to investigate.
- The requester has supplied relevant scope and evidence, or the missing information is explicitly recorded.
- The requirement is not simply a duplicate of an existing rule or unresolved ticket.
- The request is not asking for an unsafe or unauthorized response action.
- The detection objective is stated separately from the implementation idea. A request such as "write a KQL query for process X" should still explain the behavior or risk to detect.

**Output:** a complete, traceable detection requirement.

---

## 6. Phase 2 — Triage and prioritization

**Objective:** decide whether the requirement is relevant, whether coverage already exists, and how urgently the work should be performed.

The book evaluates four factors: Threat Severity, Organizational Alignment, Detection Coverage, and (when a specific exploit is involved) Active Exploits. It uses a simple additive score. The project should preserve the reasoning even if it later adopts a different scoring model.

### 6.1 Threat severity — score 1 to 3

| Score | Meaning | Illustrative example |
|---:|---|---|
| 1 | Passive activity that may enable later malicious activity. | Reconnaissance scanning. |
| 2 | Malicious activity is present but the immediate risk is relatively low at this stage. | Phishing activity without evidence of follow-on execution. |
| 3 | Activity presents a severe threat to the organization. | Ransomware payload execution. |

Score the activity being detected, not the severity of an entire hypothetical attack chain. Consider the actual environment: for example, a remote-access tool in an isolated network may have a different impact from one with unrestricted internet connectivity.

### 6.2 Organizational alignment — score 0 to 3

| Score | Meaning |
|---:|---|
| 0 | Threat is irrelevant to the organization's assets or environment. |
| 1 | Threat is unlikely to target the organization, but some risk remains. |
| 2 | Threat is widespread or untargeted. |
| 3 | Threat specifically targets the organization, industry, geography, or observed environment. |

Use internal asset inventory and threat intelligence to justify the score. Do not assume a public threat report is relevant merely because it is recent or technically interesting.

### 6.3 Detection coverage — score 0 to 2

| Score | Meaning |
|---:|---|
| 0 | In-depth coverage already exists for the requirement; do not create a duplicate rule. |
| 1 | Existing coverage needs a scope or logic update. |
| 2 | No current coverage exists; a new detection is required. |

Search the current detection repository, SIEM rules, alert history, and validation records before assigning the score.

### 6.4 Active exploit factor — only for exploit-specific requirements

**Exploit relevance to the organization:**

| Score | Meaning |
|---:|---|
| 0 | The organization is not vulnerable to the exploit. |
| 1 | The organization is vulnerable, but a patch or mitigation is expected to be deployed quickly. |
| 2 | The organization is vulnerable and a patch is unavailable or will not be deployed soon. |

**Exploit prevalence:**

| Score | Meaning |
|---:|---|
| 1 | No public exploit code or in-the-wild activity has been observed. |
| 2 | Some in-the-wild activity has been observed, but no public exploit code is available. |
| 3 | Public exploit code exists and the exploit is actively used. |

Use this factor only when the requirement concerns a specific exploit. Record affected products/versions, exposure, patch status, and evidence for exploitation claims.

### 6.5 Calculate the priority score

```text
Priority Score = Threat Severity
               + Organizational Alignment
               + Detection Coverage
               + (Exploit Relevance + Exploit Prevalence, when applicable)
```

Reject or return the requirement with an explanation when:

- Organizational Alignment = 0 (the threat is not relevant to the organization).
- Detection Coverage = 0 (in-depth coverage already exists; link the requester to it).
- Exploit Relevance = 0 for an exploit-specific request (the organization is not vulnerable).

For accepted requirements, the book's example priority bands are:

| Total score | Priority |
|---:|---|
| 1–3 | Low |
| 4–6 | Medium |
| 7–9 | High |
| 10+ | Critical |

These bands are a starting point, not a substitute for management judgment. Record a short rationale and any override. For a project-specific scoring model, version the rubric rather than silently changing historical scores.

**Output:** ACCEPTED / REJECTED / DEFERRED decision, priority, scoring rationale, owner, and target review date.

---

## 7. Phase 3 — Investigate

**Objective:** convert a high-level requirement into a technical detection specification and identify missing telemetry or engineering work before rule development begins.

The investigation has four required steps.

### 7.1 Identify data sources

Identify the assets and systems where the activity may be visible. Consider the attack path and the context of the requirement rather than selecting every available log source.

Examples:

- Endpoint activity: process creation, file creation, module/driver load, registry changes, user logon, service creation, PowerShell activity.
- Network activity: firewall events, DNS, proxy/web, VPN, NetFlow/IPFIX, packet capture, IDS/IPS, network security events.
- Identity activity: authentication, account creation, privilege changes, directory service events, cloud identity audit logs.
- Application activity: web server, application, database, email, authentication, and API audit logs.
- Cloud/SaaS activity: control-plane audit, object access, identity events, configuration changes, and application audit events.
- Security tooling: AV/EDR/XDR, email security, NGFW, IDS/IPS, vulnerability scanners, and other security products.

Distinguish **raw telemetry** (events that describe activity) from **security tooling events/alerts** (events produced after a security product has interpreted activity). Both can be useful, but know whether a signal is a raw observation, a vendor analytic, or an already correlated alert.

For each proposed source, identify:

- Source system, owner, asset scope, and collection method.
- Event types and event identifiers that contain the activity.
- Required fields and actual field names in the SIEM.
- Timestamp semantics, timezone, and ingestion delay.
- Parsing, normalization, enrichment, and mapping requirements.
- Collection status, retention, access limitations, and estimated storage/compute cost.
- Known gaps and a named team responsible for resolving them.

If a needed field is absent, create a data-engineering or platform requirement. Do not write a rule that assumes a field exists when it has not been verified in real events.

### 7.2 Determine detection indicator type

Select static, behavioral, statistical, or hybrid indicators (see Section 3.2). Record why the selected type is appropriate.

For IoC-based detections, determine where that IoC can appear. For example, a file hash is more likely in endpoint, EDR, email attachment, or file-analysis telemetry than in a basic web access log. A destination IP associated with exfiltration should be evaluated in the destination field, not indiscriminately across every IP field.

For behavioral detections, identify the events and relationships that describe the behavior: process ancestry, command line, user/session, host, logon type, network destination, file or registry activity, and event sequence.

For statistical detections, specify the aggregation key, baseline, threshold, time window, minimum sample size, and handling of known periodic or seasonal activity.

### 7.3 Research the threat and implementation

Research should establish what is consistent about the activity and what can vary.

- Review the original incident evidence and internal reports.
- Review trustworthy OSINT, vendor reports, technical blogs, whitepapers, and threat assessments.
- Map relevant techniques, procedures, threat groups, and software to ATT&CK where appropriate.
- Review existing detections and identify opportunities to update, reuse, or combine them.
- Identify variations of the behavior and potential adversary evasion paths.
- Determine which attributes are stable enough to use and which attributes are likely to change.
- Record source links, publication dates, confidence, context, and any assumptions.

Avoid overfitting the detection to fields that naturally change between executions (for example, event timestamp, hostname, randomly generated service name, or temporary filename) unless the requirement explicitly depends on them. Conversely, avoid a query so broad that it alerts on ordinary administrative activity.

### 7.4 Establish validation criteria

Before development, define how success will be demonstrated:

- Positive scenario(s): what malicious behavior should cause an alert?
- Negative scenario(s): what benign behavior must not alert?
- Boundary/edge cases: what variations may be legitimate or adversary-controlled?
- Required telemetry: what events and fields must exist?
- Expected alert: which rule, alert fields, severity, and ATT&CK mapping should appear?
- Timing: expected delay from execution to event ingestion and alert creation.
- Test method: curated telemetry, controlled manual test, Atomic Red Team, CALDERA, Purple Team, or historical incident data.
- Pass/fail criteria: what constitutes detection, partial detection, or failure?

A requirement is not ready for development if the team cannot state what evidence would prove the detection works.

**Output:** technical detection specification, test criteria, ATT&CK mapping, data-source dependencies, and data-engineering tasks (if required).

---

## 8. Data-source onboarding and telemetry quality standard

This section is a cross-cutting control. It can be performed before a detection request when onboarding a source, or during Investigate when a detection exposes a telemetry gap.

### 8.1 Maintain an authoritative inventory

Every source must have an inventory entry containing at least:

- Unique source ID and source name.
- Category (endpoint, server, network, identity, application, cloud, SaaS, security tooling).
- Business/technical owner and support contact.
- Asset scope and environment.
- Collection agent/forwarder/integration and protocol.
- Event types and expected datasets.
- Important fields and normalization target (for Elastic, record ECS mappings where applicable).
- Ingestion path and transformations.
- Expected event volume and retention class.
- Timestamp source, timezone, and expected ingestion latency.
- Health checks, last-seen time, and failure alert.
- Data classification, access restrictions, and known exclusions.
- Detections and ATT&CK techniques that depend on the source.

Use [`../templates/data-source-inventory.csv`](../templates/data-source-inventory.csv).

### 8.2 Evaluate four telemetry quality dimensions

| Dimension | Questions to answer | Example failure |
|---|---|---|
| Completeness | Are the event attributes useful for detection and investigation? Are source/destination, user, host, process, and time fields present where needed? | Network connection event exists but has no source/destination address or usable timestamp. |
| Quality | Is the data accurate, consistent, reliable, and interpretable? Are formats and field semantics understood? | Vendor field changes type or uses an undocumented value. |
| Timeliness | How long between activity, event creation, forwarding, ingestion, and alerting? | Post-processing delays events so an alert arrives too late. |
| Coverage | Are all relevant assets/events represented? Is forwarding resilient and are collection failures visible? | Public-facing Exchange servers are not forwarding logs. |

### 8.3 Source acceptance gate

Do not mark a source as production-ready until:

- A representative event sample has been inspected.
- Parsing and field extraction have been validated.
- Timestamp and timezone handling are understood.
- Required fields are present and semantics documented.
- Ingestion latency has been measured or bounded.
- Asset/source coverage has been checked.
- Source outage or pipeline failure can be detected.
- Retention and access controls are approved.

### 8.4 Elastic implementation notes

For the Elastic implementation, the inventory should identify the relevant Elastic Agent/Fleet integration, index or data stream, dataset, ECS fields, ingest pipeline, and detection rule dependencies. Keep version-specific integration configuration in the infrastructure/configuration repository. This SOP intentionally does not hard-code a specific Elastic version or an integration's current field schema.

---

## 9. Phase 4 — Develop

**Objective:** translate the technical specification into a detection implementation that is correct, understandable, efficient, and maintainable.

### 9.1 Design before coding

Write a short design before implementing the query:

- Detection objective and behavior.
- Event source(s) and required fields.
- Query/sequence/aggregation approach.
- Expected alert contents and grouping behavior.
- Known benign exceptions and exclusions.
- Expected performance and data-volume impact.
- Dependencies on data normalization, enrichment, lookup lists, or other rules.
- Planned test cases.

Simple requirements may need only a query. Complex requirements may require changes to collection, parsing, data transformation, correlation, or integrations between systems. Record those dependencies rather than hiding them in the rule.

### 9.2 Write the detection logic

Implementation requirements:

- Query only the data sources needed for the detection objective.
- Prefer normalized fields and consistent field semantics.
- Use the narrowest reliable conditions that preserve the required behavior coverage.
- Avoid tying a rule to volatile values unless those values are the actual indicator being detected.
- Include enough context to make the alert triageable (host, user, process, source/destination, event time, and relevant evidence where available).
- Add exclusions only when supported by a clear business/technical rationale; document each exclusion and its owner.
- Consider rule frequency, look-back window, deduplication, alert suppression, and query cost.
- Avoid embedding secrets, personal data, or unreviewed external content in detection code.
- Keep threat-intelligence indicators in maintainable lists or lookup structures where practical; record source and review/expiry details.

For Elastic, use the rule type and query language that best matches the behavior and available data (for example, KQL for straightforward field matching and EQL for event relationships/sequences where supported). Use the platform's supported rule export/deployment format for the exact deployed version.

### 9.3 Unit-test during development

Testing must guide development, not be deferred until the end. Create the positive and negative tests from the Investigate phase before or alongside the query.

At minimum:

- Positive fixture: expected malicious event(s) match.
- Negative fixture: benign activity does not match.
- Boundary fixture: expected legitimate exception is handled correctly.
- Field-missing fixture: confirm the behavior when a required field is absent or malformed.
- Variation fixture: test a meaningful alternative procedure or changed value.

Run the rule against fixtures and compare the results to the validation criteria. Refactor for readability/performance and rerun all tests after every logic change.

### 9.4 Use reusable building blocks carefully

Where the platform supports it, common event filters, helper rules, or building blocks can reduce duplicated logic. Composite detections can combine signals across multiple sources. Document dependencies and ensure that a helper rule cannot silently break downstream detections when changed.

**Output:** version-controlled detection implementation, test fixtures, completed metadata, and development evidence.

---

## 10. Phase 5 — Test and validate

**Objective:** demonstrate that the detection identifies the intended activity, avoids unacceptable benign alerts, and does not create unacceptable platform or analyst costs.

### 10.1 Required test-data classes

**Known bad data** represents adversary activity. It may come from controlled simulations, sandbox captures, real incident artifacts approved for testing, or carefully constructed telemetry.

**Known good data** represents benign activity. Prefer representative operational data where permitted. Include known administrative and business workflows that might resemble the detected behavior.

Also maintain edge-case datasets for important variants and exceptions.

### 10.2 Test sequence

1. **Syntax and structural test:** rule parses, required fields exist, metadata is valid, and the rule can execute.
2. **Positive test:** run against known-bad data; verify the expected alert and evidence fields.
3. **Negative test:** run against known-good data; identify unwanted alerts.
4. **Boundary/variation test:** vary the relevant parameters or procedure while keeping the same underlying behavior.
5. **Performance test:** review query execution, event volume, alert count, ingestion load, and resource impact.
6. **Operational review:** ask whether the SOC can understand and investigate the alert using its documented guidance.
7. **Regression test:** rerun the full test set after tuning or code changes.

### 10.3 Adversary simulation

Use controlled, authorized simulation in an isolated or explicitly approved environment:

- **Atomic Red Team:** useful for focused tests of individual techniques or procedures.
- **MITRE CALDERA:** useful for orchestrating multi-step adversary emulation with agents, abilities, adversary profiles, and operations.
- **Purple Team / Red Team:** useful for testing realistic activity in the context of the organization's environment.

Record the exact test execution time, host, user, technique/procedure, test result, and resulting SIEM alert. In busier environments, correlate alerts by timestamp and execution evidence rather than relying only on a technique ID.

### 10.4 Evaluate validation failures correctly

A failed validation is not automatically a request for a new rule. Investigate in this order:

1. Did the simulation actually execute successfully?
2. Did the source collect the activity?
3. Did the event reach the SIEM in the expected time window?
4. Were the required fields parsed and normalized correctly?
5. Is an existing detection expected to catch this behavior?
6. Did that rule fail because of a logic bug, overly narrow conditions, a missing field, or an incorrect assumption?
7. Is this a genuine coverage gap, a telemetry/architecture gap, or an existing-rule maintenance issue?
8. Is the value of additional visibility/detection worth the engineering and operational cost?

Classify the outcome as:

- **Detected:** expected behavior produced an appropriate alert.
- **Partially detected:** some tested variants were detected, others were not.
- **Not detected — rule gap:** telemetry exists, but no appropriate detection covers the behavior.
- **Not detected — rule defect:** an existing rule should have fired but did not.
- **Not detected — visibility gap:** the required telemetry is unavailable, incomplete, or delayed.
- **Inconclusive:** simulation or test data was invalid or insufficient.

### 10.5 Validate at the right level of detail

If the detection claims to cover a technique through a behavior, test meaningful variations of the procedure that implement that technique. A test that only identifies a tool or a known C2 address may prove tool/indicator visibility, not broad technique coverage. Document the scope of the claim and any uncovered procedures.

**Output:** validation report, false-positive/false-negative observations, coverage assessment, and follow-up requirements.

---

## 11. Phase 6 — Deploy

**Objective:** move a tested detection into production through controlled maturity stages, with monitoring and a rollback path.

### 11.1 Detection maturity stages

| Stage | Purpose | Minimum gate | Alert handling |
|---|---|---|---|
| Experimental | Observe the rule in a production-like or limited production scope. | Rule executes; positive samples trigger; known-good samples reviewed; lint and documentation exist; author monitors it. | Treat as experimental/low confidence; limit scope where practical. The author owns review. |
| Test | Expose the tuned rule to broader operational review. | Acceptable false-positive level; complete documentation; no unresolved critical test failure; peer review of rule and resulting alerts. | SOC/team may review; label clearly as test and avoid equating it with stable severity. |
| Stable | Full production release. | Thorough testing, independent approval, documented investigation guidance, correct severity/confidence, rollback path. | SOC can rely on the rule according to its documented confidence and severity. |

A stable rule must not be promoted merely because it has existed for a long time. It must satisfy the gates.

### 11.2 Git workflow

The book describes Feature, Experimental, Test, and Main/Stable stages. For GitHub, implement them using protected branches, pull requests, release labels, deployment environments, or tags as appropriate. A practical mapping is:

- `feature/<ticket>-<short-name>`: authoring, tests, and documentation.
- `experimental`: candidate rules deployed with an experimental tag and limited scope where appropriate.
- `test`: rules ready for broader review and operational acceptance.
- `main`: approved stable production content.

Use required reviews and CI checks on protected branches. Do not allow direct pushes to `main` for production detections.

### 11.3 CI/CD gates

Recommended pipeline stages:

```text
Pull Request
  -> YAML/JSON/schema validation
  -> query lint / rule validation
  -> metadata completeness check
  -> test fixture execution
  -> positive / negative / edge-case assertions
  -> duplicate / dependency checks (where feasible)
  -> reviewer approval
  -> deploy to experimental
  -> monitor and tune
  -> promote to test
  -> operational peer review
  -> promote to stable
  -> production deployment
  -> post-deployment health check
```

The actual pipeline must use deployment interfaces supported by the deployed Elastic version. The pipeline must fail closed if validation fails. A manual UI change in production must be reconciled back into Git immediately so that Git remains the source of truth.

### 11.4 Release and rollback record

For every deployment, record:

- Rule ID and version/commit.
- Maturity stage and environment/scope.
- Deployment timestamp and deployer.
- Related ticket and pull request.
- Test report and reviewer approvals.
- Known limitations and expected alert volume.
- Previous known-good version and rollback/disable procedure.

After deployment, verify that the rule is enabled as intended, scheduled correctly, producing expected alert fields, and not causing ingestion/query performance issues.

**Output:** versioned deployed detection, change record, maturity status, and a verified rollback path.

---

## 12. Detection documentation standard

Every rule must have a documentation record that is readable by both detection engineers and SOC analysts.

### 12.1 Mandatory metadata

The book's documentation template includes: Author, Creation Date, Revision History, Maturity, License, Ticket, References, Name, Description, MITRE TTPs, Tags, Rule, Rule Format, Data Sources, Related Detections, Investigation Suggestions, Responses or Remediations, False Positives, Severity, and Confidence.

This project additionally requires:

- Stable rule ID and owner/team.
- Requirement ID and priority.
- Indicator type (static, behavioral, statistical, hybrid).
- Required fields/datasets and known telemetry dependencies.
- Schedule/look-back window and performance considerations.
- Positive, negative, and edge-case test references.
- Validation history and last validated date.
- Known limitations and exclusions with justification.
- Review/expiry date for time-sensitive IoCs or detections.
- SOAR/case routing expectation, if applicable.

### 12.2 Alert quality requirements

The alert should provide enough context to answer:

- What happened?
- Which host, user, process, network endpoint, account, or application was involved?
- Why does this match the detection logic?
- Which ATT&CK technique(s) are relevant, and how directly does the rule identify them?
- What should the analyst inspect next?
- What benign behavior could explain the alert?
- What action is recommended, and what approval is required?

Do not map a technique solely because it is related to the incident narrative. Map the behavior that the rule actually detects, and label indirect or supporting mappings accurately.

Use [`../templates/detection-spec.md`](../templates/detection-spec.md) and [`../templates/detection-rule.yaml`](../templates/detection-rule.yaml).

---

## 13. Threat intelligence operating procedure

Threat intelligence is integrated into Requirements Discovery, Triage, and Investigate; it is not a separate step that automatically creates rules.

### 13.1 Sources

- OSINT: public reports, vendor research, blogs, whitepapers, social media, threat feeds, online scanners, and sandboxes.
- Internal intelligence: SOC/IR incidents, threat hunting, forensic analysis, Red Team results, and historical alert outcomes.

Establish how intelligence is collected, assessed, assigned, documented, and delivered to detection engineering. A feed with many indicators is not automatically high value. Context, reliability, relevance, and operational impact matter.

### 13.2 Processing workflow

1. Register the intelligence item and its source/date.
2. Identify the threat, affected technology, target profile, campaign, and confidence.
3. Extract indicators, tools, techniques, procedures, and observed sequence.
4. Correlate the findings with internal assets, incidents, and current detections.
5. Convert relevant gaps or improvements into detection requirements.
6. Triage using the standard scoring process.
7. During investigation, map the behavior to telemetry and fields.
8. Develop and validate the detection; retain source references in the rule documentation.
9. Reassess time-sensitive indicators and remove or expire stale values.

### 13.3 IoC handling safeguards

- Do not create a high-severity alert from a low-confidence IP/domain/hash match by default.
- Consider indicator age, reassignment, reputation context, and the activity that caused the match.
- Preserve source attribution and confidence.
- Prefer context-aware matching (for example, relevant destination field and event type) over matching the same string in every field.
- Use IoCs as tactical coverage while developing behavior-based detections where feasible.

---

## 14. Continuous activities: Monitor, Maintain, Measure, Validate

These activities run across the entire lifecycle and can create new detection requirements.

### 14.1 Monitoring

Monitor production rules for:

- Alert count and trend by rule.
- True-positive / false-positive / benign disposition.
- Unexpected zero-alert behavior for rules expected to fire.
- New false-positive patterns and changed business workflows.
- Query failures, execution delays, missing data, and resource use.
- Detection/ingestion pipeline health.

A sudden increase or decrease in alerts is a reason to investigate, not proof of failure by itself.

### 14.2 Maintenance

Review detections when:

- Threat behavior or adversary tooling changes.
- IoCs become stale or are reassigned.
- Assets, applications, policies, or business workflows change.
- Data sources, schemas, integrations, or field names change.
- Validation exposes missed procedures or false positives.
- A rule is duplicated, unused, too expensive, or no longer relevant.

Every production rule must have an owner and a review cadence. The cadence should be based on risk and volatility; time-sensitive IoC rules require more frequent review than durable behavior rules.

### 14.3 Metrics

Separate **efficiency** (how much effort/cost the program consumes) from **effectiveness** (how well it detects adversary activity).

Recommended program metrics:

- Requirement throughput and age by priority.
- Time in each lifecycle state; development-to-release lead time.
- Rules by maturity, owner, and ATT&CK mapping.
- Alert volume, analyst handling time, escalation rate, and disposition rate.
- Query failures, ingestion delay, missing-source duration, and pipeline health.
- Validation test coverage and pass rate by technique/procedure.
- Stale rules and rules overdue for review.

Detection-level outcomes:

- True Positive (TP): an alert fired for actual malicious activity.
- False Positive (FP): an alert fired for benign activity.
- True Negative (TN): no alert for benign activity.
- False Negative (FN): malicious activity did not trigger an alert.

Useful formulas:

```text
Precision = TP / (TP + FP)
Noisiness = FP / (TP + FP)      # proportion of this rule's alerts that are benign
Recall    = TP / (TP + FN)
```

The book notes that complete program accuracy is difficult to calculate because true negatives are not readily enumerable. False negatives are also hard to identify without forensic review or controlled testing. Mean Time to Detect (MTTD) is useful as a historical measure, but it only describes attacks that were eventually detected; it should not be the only measure of detection effectiveness.

ATT&CK heatmaps and detection counts are useful low-fidelity views, but the number of rules mapped to a technique is not proof of actual behavior coverage. Prefer validation metrics that show which procedures/variations have tests and how often those tests pass.

### 14.4 Comprehensive validation

Run validation at multiple levels:

- Unit tests for individual rule logic.
- Regression tests for existing behavior after changes.
- Procedure-level tests for meaningful variations of ATT&CK techniques.
- Purple Team / Red Team exercises for broader operational behavior.
- Periodic coverage reviews to identify visibility gaps and stale assumptions.

A failed simulation must be checked for successful execution and telemetry visibility before it is labeled a detection failure. Validation results should create a new requirement, an update to an existing rule, a data-source requirement, or an accepted risk decision.

---

## 15. SIEM-to-SOAR interface standard (project extension)

This section is an additional project control. The source book covers detection engineering but does not define a complete SOAR governance framework.

### 15.1 Alert contract

Every alert eligible for automation should expose stable, normalized fields such as:

- Alert/rule ID and rule version.
- Event and alert timestamps.
- Severity, confidence, risk score, and maturity.
- Host, user, process, source/destination, cloud account, and asset identifiers where relevant.
- ATT&CK tactic/technique mapping.
- Evidence and source event references.
- Investigation guide, recommended next step, and case-routing category.
- Required approvals and automation eligibility.

### 15.2 Automation tiers

| Tier | Action type | Governance |
|---|---|---|
| 0 | Enrichment and read-only lookups. | May be automated after testing; retain audit logs. |
| 1 | Case creation, tagging, notification, evidence collection. | May be automated with deduplication, rate limits, and failure handling. |
| 2 | Reversible containment or account/session action. | Require explicit policy conditions, approval or well-defined emergency authorization, and rollback. |
| 3 | Destructive or business-impacting action (for example, deleting data or shutting down critical systems). | Never trigger solely from one unreviewed detection. Require designated human approval and incident/change controls. |

### 15.3 SOAR safety requirements

- Detection rules should raise signals; playbooks should make response decisions.
- Require idempotency, retries, timeouts, rate limits, and error handling.
- Validate identity and asset scope before action.
- Exclude critical systems unless explicitly approved.
- Keep an audit trail of trigger, enrichment, decision, approval, and action result.
- Define rollback or recovery steps for reversible actions.
- Prevent alert storms from causing repeated high-impact actions.
- Test playbooks with mock or lab data before enabling production actions.
- Feed case outcomes and automation failures back into detection requirements and rule tuning.

---

## 16. Repository and change-management standard

Git is the source of truth for custom detection logic and its supporting artifacts.

### 16.1 Suggested directory structure

```text
.
├── detections/
│   ├── windows/
│   ├── linux/
│   ├── network/
│   ├── identity/
│   ├── cloud/
│   └── application/
├── data-sources/
│   ├── inventory.csv
│   └── mappings/
├── intel/
│   └── assessments/
├── validation/
│   ├── atomic-red-team/
│   ├── caldera/
│   └── reports/
├── runbooks/
├── templates/
└── docs/
```

Each detection directory should contain a stable ID, rule implementation/export, metadata, tests, and a short README or link to the full detection specification.

### 16.2 Branching and pull requests

- Work in a feature branch tied to a requirement/ticket.
- Include rule, metadata, tests, and documentation in the same change.
- Require automated checks and independent review before promotion.
- Promote through experimental, test, and stable maturity stages.
- Record all production changes in Git; reconcile emergency UI changes immediately.

### 16.3 Versioning

Preserve stable rule IDs across edits. Increment a revision/version field for logic changes. Record meaningful behavior changes in revision history. A rename, query change, threshold adjustment, exclusion, severity change, or ATT&CK mapping change must be reviewable in Git history.

### 16.4 Emergency changes

Emergency deployment is permitted only through the project's incident/change approval process. The change must still have:

- An incident/ticket reference.
- An author and approver.
- A clear scope and expiry/review time.
- Minimal tests possible under the circumstances.
- A rollback method.
- A follow-up pull request that restores Git as the source of truth and completes missing documentation/tests.

---

## 17. Decommissioning detections

A rule may be deprecated when its threat, software, asset scope, or business requirement no longer applies; a better rule replaces it; the source telemetry is retired; the rule is too noisy or costly to justify; or its capability is no longer supported.

Before retirement:

1. Confirm the original requirement and affected use cases.
2. Search for dependent rules, dashboards, cases, and SOAR playbooks.
3. Check with the SOC and service owner.
4. Document the reason, risk, and replacement coverage (if any).
5. Disable before deleting when a short observation period is warranted.
6. Preserve the final rule, test report, and revision history in Git.
7. Update ATT&CK coverage and metrics.

---

## 18. Project Definition of Done

A detection is **Stable / Production Ready** only when all applicable items are true:

- [ ] A complete requirement exists and has an approved priority.
- [ ] The requirement has a unique ID and accountable owner.
- [ ] Existing coverage and dependencies were reviewed.
- [ ] Data sources and required fields were verified from actual events.
- [ ] Telemetry completeness, quality, timeliness, and coverage were assessed.
- [ ] Indicator type and ATT&CK mapping reflect the behavior actually detected.
- [ ] Research references and assumptions are documented.
- [ ] Positive, negative, and relevant edge-case tests exist.
- [ ] Unit and validation tests pass, or exceptions are explicitly approved.
- [ ] False-positive scenarios and known limitations are documented.
- [ ] Alert context and investigation guidance are sufficient for the SOC.
- [ ] Severity and confidence are justified.
- [ ] Query and ingestion performance have been reviewed.
- [ ] Independent review is complete.
- [ ] Experimental/test promotion gates have been satisfied.
- [ ] Deployment record and rollback path exist.
- [ ] Monitoring owner and review cadence are assigned.
- [ ] SOAR eligibility and approval constraints are explicit, if applicable.

If an item is not applicable, record why. Do not leave a gate blank without explanation.

---

## 19. Reference map to the source book

This is a navigation aid showing how this SOP was derived. Page numbers are the printed page numbers in the 2023 first edition.

| SOP section | Book section | Printed pages |
|---|---|---:|
| Foundations, attack models, quality and Pyramid of Pain | Chapter 1 — Fundamentals of Detection Engineering | 3–21 |
| Six phases and four continuous activities | Chapter 2 — The Detection Engineering Life Cycle | 23–40 |
| Test lab and initial Elastic detection | Chapter 3 — Building a Detection Engineering Test Lab | 41–69 |
| Telemetry, source categories, source quality | Chapter 4 — Detection Data Sources | 73–97 |
| Requirement intake, triage, priority scoring, investigation | Chapter 5 — Investigating Detection Requirements | 99–113 |
| Static/IoC detections and Sysmon | Chapter 6 — Developing Detections Using Indicators of Compromise | 115–139 |
| Tool- and TTP/behavior-based detections | Chapter 7 — Developing Detections Using Behavioral Indicators | 141–168 |
| Detection documentation, repositories, detection-as-code, CI/CD | Chapter 8 — Documentation and Detection Pipelines | 169–195 |
| Atomic Red Team, CALDERA, validation results and coverage | Chapter 9 — Detection Validation | 199–227 |
| OSINT, internal intelligence, threat assessments | Chapter 10 — Leveraging Threat Intelligence | 229–248 |
| Program efficiency/effectiveness and detection efficacy | Chapter 11 — Performance Management | 251–278 |

---

## 20. Governance and review of this SOP

- SOP owner: SIEM / Detection Engineering project maintainer.
- Review this SOP at least when the detection platform, data architecture, deployment process, or response governance changes materially.
- Every change to the lifecycle gates, scoring rubric, maturity stages, or required metadata must be version-controlled and approved.
- Keep implementation-specific instructions (Elastic API versions, integration package versions, CI actions, SOAR vendor syntax) in separate deployment documentation so that this SOP remains stable as the platform evolves.
