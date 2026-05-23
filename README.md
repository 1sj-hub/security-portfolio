# Security Portfolio — Steeve Joseph

**Security+ Certified | AI Security | Vulnerability Management | GRC Translation**

Real security work across production AI systems, infrastructure hardening, compliance operations, and AI security research.

This repository documents practical security work, not classroom labs.

---

## What This Portfolio Shows

- Production vulnerability assessment & remediation
- AI / RAG security architecture
- Risk identification and mitigation
- Compliance-to-technical control translation
- Security documentation for technical and executive audiences

---

## Featured Projects

### 1. Production Vulnerability Assessment & Remediation

Credentialed security assessment of a live production AI application environment.

![Vulnerability Remediation Dashboard](./images/vuln-rem-dashboard.png)

**Scope**

- Host vulnerability analysis
- Package exposure review
- Patch remediation planning
- Risk prioritization
- Remediation execution
- Residual risk documentation

**Outcome**

- 15 actionable vulnerabilities identified
- 2 critical findings (CVSS 9.8)
- 14 remediated in first remediation cycle
- 1 risk documented with compensating controls

**Skills Demonstrated**

- Vulnerability management
- CVE triage
- Risk analysis
- Patch governance
- Linux remediation workflows
- Security reporting

**Artifacts**

- [Executive Assessment Summary](./artifacts/vuln-assessment-summary.md)
- [Redacted Findings Report](./artifacts/redacted-findings.md)
- [Remediation Decision Log](./artifacts/remediation-log.md)

---

### 2. AI / RAG Security Architecture — AskObi

Security architecture analysis and control design for a production retrieval-augmented generation platform.

![AI Security Threat Model](./images/ai-security-threat.png)

**Focus Areas**

- API access control
- Source trust enforcement
- Prompt injection risk
- Hallucination containment
- Retrieval integrity
- Vector security considerations
- Application-layer validation

**Skills Demonstrated**

- AI security architecture
- Threat modeling
- Trust boundary analysis
- Secure API design
- Adversarial AI risk mitigation

**Artifacts**

- [Threat Model](./artifacts/rag-threat-model.md)
- [Security Architecture Overview](./artifacts/rag-security-architecture.md)
- [Control Mapping](./artifacts/ai-control-mapping.md)

---

### 3. Regulatory Compliance Intelligence Pipeline — Rulewatch

Security-minded compliance automation pipeline for regulatory monitoring and structured intelligence delivery.

**Focus Areas**

- Source authenticity validation
- Completeness verification
- Workflow integrity
- Audit-ready output design
- Control-minded automation

**Skills Demonstrated**

- Compliance engineering
- Control design
- Governance workflow thinking
- Structured data validation

**Artifacts**

- [Workflow Overview](./artifacts/rulewatch-workflow.md)
- [Control Logic Notes](./artifacts/control-design.md)

---

### 4. AI Security Research

Applied research into modern AI security threats.

**Research Topics**

- Prompt injection
- Retrieval poisoning
- Vector database attack surface
- Hallucination as operational risk
- AI governance controls
- NIST-aligned AI security concepts

**Artifacts**

- [Prompt Injection Analysis](./research/prompt-injection.md)
- [RAG Threat Research](./research/rag-threats.md)
- [AI Risk Governance Notes](./research/ai-governance.md)

---

## Technical Environment

**Security**
Nessus · CVSS · vulnerability triage · remediation governance · risk documentation

**Infrastructure**
Linux · Ubuntu · server hardening · package management · secure deployment workflows

**Application Security**
API validation · authentication controls · input sanitization · trust boundary analysis

**AI Security**
RAG architecture · LLM threat modeling · prompt security · retrieval integrity · adversarial AI analysis

**Compliance / Governance**
Control mapping · policy-to-technical translation · audit documentation · operational governance

---

## Professional Background

**Compliance Foundation**

- HIPAA-regulated operational leadership
- Zero violation operational environment
- Regulated aviation exposure (FAA / TSA / DHS)
- Licensed financial services compliance exposure

**Technical Foundation**

Builder/operator of production AI systems with practical security ownership.

---

## Certifications

- CompTIA Security+ (SY0-701)

---

## Repository Structure

```text
/artifacts
    vuln-assessment-summary.md
    redacted-findings.md
    remediation-log.md
    rag-threat-model.md
    rag-security-architecture.md
    ai-control-mapping.md

/research
    prompt-injection.md
    rag-threats.md
    ai-governance.md

/images
    ai-security-threat.png
    vuln-rem-dashboard.png
```

---

## Production Vulnerability Assessment & Remediation — Detail

Performed a credentialed Nessus vulnerability assessment against a live production AI application environment.

### Initial Risk Snapshot

| Severity      | Count |
| ------------- | ----- |
| Critical      | 2     |
| High          | 10    |
| Medium        | 3     |
| Low           | 0     |
| Informational | 80    |

### Post-Remediation Snapshot

| Severity      | Count |
| ------------- | ----- |
| Critical      | 0     |
| High          | 1     |
| Medium        | 0     |
| Low           | 0     |
| Informational | 79    |

### Remediation Outcome

- 15 actionable findings identified
- 14 findings remediated in first remediation cycle
- 100% of Critical findings resolved
- 100% of Medium findings resolved
- 93.3% reduction in actionable findings
- 1 residual High accepted with documented compensating controls and quarterly review

### Residual Risk Decision

One remaining High finding was accepted after validation that the production application runtime uses a patched Brotli dependency path. The vulnerable system package remains due to operating-system dependency constraints and is not used by the application to process attacker-controlled Brotli streams.

### Skills Demonstrated

- Credentialed vulnerability assessment
- CVE triage
- Risk prioritization
- Linux remediation
- Dependency analysis
- False-positive / residual-risk validation
- Risk acceptance documentation
- Executive security reporting

---

## Residual Risk Acceptance — Python Brotli DoS

### Finding

Nessus flagged a High severity finding related to `Python Library Brotli <= 1.1.0 DoS`.

### Decision

Accepted residual risk with quarterly review.

### Rationale

The production application runtime imports Brotli `1.2.0` from its application virtual environment. The flagged system package remains installed due to operating-system dependency constraints and is not used by the application process to deserialize attacker-controlled Brotli streams.

### Compensating Control

The application runtime does not use the vulnerable system package path for attacker-controlled Brotli processing.

### Review Triggers

- New upstream Brotli security release
- EPSS rises above 0.01
- Application dependency path changes
- Relevant OS package update becomes available

### Next Review

Quarterly.
