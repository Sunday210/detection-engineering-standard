# SIEM Detection Engineering Standard

A version-controlled operating standard for the end-to-end Detection Engineering process used by this SIEM project.

This repository standardizes how the team discovers, prioritizes, investigates, develops, documents, tests, deploys, validates, monitors, and maintains detections. It is designed to be technology-aware but not tightly coupled to one query language. Initial implementation examples are oriented toward Elastic Security; platform-specific rule exports and deployment adapters belong in the implementation layer.

## Start here

1. Read [`docs/01-DETECTION-ENGINEERING-SOP.md`](docs/01-DETECTION-ENGINEERING-SOP.md) for the complete lifecycle and mandatory gates.
2. Use [`templates/detection-requirement.md`](templates/detection-requirement.md) to submit a new detection request.
3. Use [`templates/data-source-inventory.csv`](templates/data-source-inventory.csv) to record telemetry sources and their health.
4. Use [`templates/detection-spec.md`](templates/detection-spec.md) to translate an investigated requirement into a technical design.
5. Use [`templates/detection-rule.yaml`](templates/detection-rule.yaml) as a platform-neutral metadata and rule-development starting point; adapt the query/export to the target SIEM.
6. **Create a completed documentation record for every detection** using [`templates/detection-documentation.md`](templates/detection-documentation.md). This is required for SOC triage, investigation, response, and future maintenance; it is not replaced by the raw rule file.
7. Use [`templates/validation-report.md`](templates/validation-report.md) to record test outcomes.
8. Apply [`templates/detection-review-checklist.md`](templates/detection-review-checklist.md) and the [Pull Request template](.github/PULL_REQUEST_TEMPLATE.md) before merging.

## Lifecycle at a glance

```text
Requirement intake
      |
      v
Requirements Discovery -> Triage -> Investigate -> Develop -> Document -> Test
                                                                  |
                                                                  v
                                                     Deploy: Experimental
                                                                  |
                                                                  v
                                                        Deploy: Test / Review
                                                                  |
                                                                  v
                                                          Deploy: Stable
                                                                  |
                                                                  v
                                  Monitor / Maintain / Measure / Validate
                                                                  |
                                      feedback creates new requirements
```

The six lifecycle phases and four continuous activities follow the framework in *Practical Threat Detection Engineering* (Roddie, Deyalsingh, Katz, Packt, 2023). Documentation is treated as a required development/deployment deliverable, following Chapter 8's dedicated section **Documenting a detection**.

## Repository layout

```text
.
├── README.md
├── docs/
│   └── 01-DETECTION-ENGINEERING-SOP.md
├── templates/
│   ├── data-source-inventory.csv
│   ├── detection-documentation.md
│   ├── detection-requirement.md
│   ├── detection-review-checklist.md
│   ├── detection-rule.yaml
│   ├── detection-spec.md
│   └── validation-report.md
└── .github/
    └── PULL_REQUEST_TEMPLATE.md
```

## Where the completed detection documentation lives

The file in `templates/` is a blank form. For each actual detection, copy it into that detection's directory and complete it alongside the rule, tests, and validation evidence. For example:

```text
detections/
└── windows/
    └── DE-0007-suspicious-service-install/
        ├── rule.yaml
        ├── documentation.md
        ├── tests/
        └── validation/
```

The completed `documentation.md` is the analyst-facing and engineer-facing record for that particular detection. Keep it version-controlled with the rule so logic changes and documentation changes are reviewed together.

## Baseline principles

- Every detection starts with a documented requirement and an owner.
- A detection must be based on telemetry that actually exists, is trustworthy, and arrives in time.
- Prefer behavior and TTP-oriented detections when telemetry supports them; use static IoCs for timely tactical coverage, not as the only detection strategy.
- Every rule must have positive, negative, and boundary-case tests appropriate to its scope.
- Every detection has its own documentation record, including rule purpose, data sources, ATT&CK mapping, false positives, severity, confidence, investigation suggestions, and response guidance.
- No rule reaches stable production without documentation, review, validation evidence, and a rollback path.
- Alerts must be actionable: the SOC must understand why the alert fired and what to investigate next.
- Detection coverage is measured by validated behavior/procedure coverage, not merely by counting ATT&CK tags or rules.
- SIEM detections and SOAR response actions are governed separately. High-impact response actions require explicit authorization and safeguards.

## Scope and implementation note

This standard governs the detection engineering workflow and its interfaces with SIEM ingestion, alert triage, case management, and SOAR. It does not prescribe one universal Elastic deployment architecture, retention period, risk score, or query syntax. Those are implementation decisions that must be recorded in the project configuration and reviewed against the deployed Elastic version.

## Source basis

Primary source: *Practical Threat Detection Engineering: A hands-on guide to planning, developing, and validating detection capabilities*, Megan Roddie, Jason Deyalsingh, and Gary J. Katz, Packt, 1st edition, July 2023.

Relevant sections: Chapters 1–2 (foundations and lifecycle), Chapter 3 (test lab), Chapter 4 (data sources), Chapter 5 (requirements and triage), Chapters 6–7 (IoC and behavioral detections), Chapter 8 (Documenting a detection, detection repositories, and detection-as-code), Chapter 9 (validation), Chapter 10 (threat intelligence), and Chapter 11 (performance management).

This repository is an original operational synthesis and adaptation. It does not reproduce the book's text or lab instructions.
