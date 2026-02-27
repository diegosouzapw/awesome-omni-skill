---
name: naiba-openai-government-it-staff
description: A quick-start guide for IT teams at any level of government who just received ChatGPT access | Part of naiba-openai-work-assistant
version: 1.0.0
aliases: government, gov, public sector, civil service
role: Government IT Staff
---

# Government IT Staff Assistant

This skill provides **21+ professional prompts** tailored for Government IT Staff professionals.

## Available Categories

### System Security & Vulnerability Management

**Analyze vulnerability scan results**

```
Analyze these weekly vulnerability-scan results for the [system name] and group findings by severity and affected component. Recommend remediation steps ranked by risk reduction.
```

---

**Summarize application-security exceptions**

```
Draft a one-page summary of all application-security exceptions granted last quarter and map each to the relevant control in our [cybersecurity baseline].
```

---

**Extract attack vectors from logs**

```
Extract the ten most frequent attack vectors from these intrusion-detection logs and visualise them in a bar chart for the monthly security briefing.
```

---

### DevOps & Release Management

**Merge code-coverage reports**

```
Merge these code-coverage reports from the last three builds, calculate test-coverage percentage for each module, highlight any module below 75 percent, and produce a bar chart of the results with a short narrative explaining the biggest gaps.
```

---

**Summarize performance-test data**

```
Summarize performance-test data and highlight endpoints exceeding the [value] millisecond Service Level Agreement (SLA). Present findings as a table.
```

---

**Create change-management request template**

```
Create a change-management request template for rolling back version [ # ] of the [application]. Include impact analysis, rollback steps, and stakeholder notifications.
```

---

### Infrastructure & Cloud Operations

**Compare IaC definitions against policies**

```
Here are our existing IaC definitions for the standby database cluster (in YAML/JSON). Compare each resource against our [policy name] requirements—data-at-rest encryption, network isolation, tag standards—and produce a table of any non-compliant items with suggested fixes.
```

---

**Review server configuration manifests**

```
Review these server configuration manifests and suggest security baselines aligned with widely accepted benchmarks (e.g., CIS, NIST). Present recommendations in a table with columns for config area, current setting, recommended baseline, and rationale.
```

---

**Generate weekly capacity report**

```
Generate a weekly capacity report for virtual machines hosting the [system name], including CPU, memory, and storage trends. Include 30-day forecasts using historical usage. Present results as: (1) summary table of resource utilization, (2) line charts by metric, and (3) a short narrative identifying any projected constraints.
```

---

### Data Quality, Analysis & Visualization

**Deduplicate dataset**

```
Deduplicate this dataset of [dataset name / type] by identity number and date, flagging conflicting entries for review. Provide a summary of duplicates removed.
```

---

**Create dashboard-ready summary**

```
Create a dashboard-ready summary showing the distribution of response times in these help-desk logs, and highlight outliers beyond two standard deviations.
```

---

**Combine and normalize tab-delimited files**

```
Combine these three tab-delimited exports into one normalized table, add a 'last_updated' timestamp, and output the result in JavaScript Object Notation (JSON).
```

---

### Service Desk & End-User Support

**Generate knowledge-base article**

```
Generate a knowledge-base article on enrolling devices in our mobile-device-management solution, with step-by-step screenshots and plain-language instructions.
```

---

**Analyze ticket logs**

```
Analyze last quarter's ticket logs and surface the top five recurring issues by department. Suggest self-service resources for each.
```

---

**Draft decision-tree for ticket categorization**

```
Draft a decision-tree for categorizing new support tickets based on keywords and priority. Present it as indented plain text.
```

---

### Procurement & Vendor Oversight

**Compare SLAs in proposals**

```
Compare the Service Level Agreements (SLAs) in these three cloud-hosting proposals and highlight gaps against our uptime requirement.
```

---

**Generate RFP template**

```
Generate a draft Request for Proposal (RFP) template for a Security Information and Event Management (SIEM) platform, referencing our jurisdiction's procurement rules and minimum cybersecurity controls.
```

---

**Summarize vendor-performance metrics**

```
Summarize quarterly vendor-performance metrics and draft a letter requesting a service-credit discussion.
```

---

### Incident & Continuity Response

**Draft incident communications**

```
Draft an initial incident ticket, public statement, and internal chat update for a suspected ransomware event affecting [agency] mail servers.
```

---

**Generate post-incident report outline**

```
Generate a post-incident report outline for last week's network outage, including root cause, mitigation, and lessons-learned sections.
```

---

**Create continuity-of-operations checklist**

```
Create a continuity-of-operations checklist for migrating critical apps to an alternate data center within 24 hours.
```

---

