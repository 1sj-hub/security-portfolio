# RAG Threat Model — AskObi

**Production AI security threat model for a retrieval-augmented civic intelligence platform.**

Framework: **OWASP LLM Top 10**  
Model Type: **Public-safe architectural threat model**  
Owner: **Steeve Joseph**

---

## Disclosure

This model is intentionally public-safe.

It reflects real production security thinking without exposing implementation-sensitive details, infrastructure topology, provider configurations, or deployment specifics.

The objective is to demonstrate security reasoning—not publish an attack blueprint.

---

## System Context

AskObi is a retrieval-augmented AI platform for civic intelligence.

Users submit natural-language questions about legislative and regulatory materials. Requests pass through authentication, validation, retrieval, source trust enforcement, LLM generation, and output governance before delivery.

Each stage is treated as a control boundary.

---

## Threat Model

![AI Security Threat Model](../images/ai-security-threat.png)

### Trust Zones
- **Ingress Controls** — untrusted user input entering the system
- **Retrieval Integrity** — protecting retrieval, grounding, and corpus trust
- **Response Governance** — controlling generated output before delivery

---

## Core Trust Boundaries

### 1. User → Application
Public input enters the system as adversarial by default until validation and control enforcement occur.

### 2. Retrieved Content → LLM Context
Retrieved content is treated as semi-trusted. Even curated sources can introduce indirect prompt injection or integrity risk.

### 3. Model Output → User
LLM output remains untrusted until output controls complete.

---

## Control Mapping

| Stage | Threat | Primary Control | Status |
|------|--------|----------------|--------|
| API Gateway | API Abuse | Rate Limiting | Implemented |
| Auth / Validation | Unauthorized Access | Authentication Controls | Implemented |
| Prompt Guardrails | Prompt Injection | Query Sanitization | Implemented |
| Retriever | Retrieval Poisoning | Input Validation | Implemented |
| Vector Store | Data Leakage | Access Boundaries | Implemented |
| Source Trust Filter | Source Integrity | Source Verification | Implemented |
| LLM Response Layer | Hallucination Risk | Audit Logging | Implemented |
| Output Guardrails | Unsafe Output | Output Controls | Implemented |

---

## OWASP LLM Coverage

This model maps controls against the OWASP LLM Top 10 threat landscape, including:

- Prompt Injection
- Sensitive Information Disclosure
- Supply Chain / Data Poisoning
- Improper Output Handling
- Excessive Agency
- System Prompt Leakage
- Vector / Embedding Weaknesses
- Misinformation
- Unbounded Consumption

Defense-in-depth is intentionally applied where residual risk remains highest.

---

## Security Philosophy

AI security is trust-boundary security.

The critical question is not:

**“Is the model secure?”**

It is:

**“What untrusted behavior can cross into trusted execution paths?”**

This model exists to answer that.

---

## Assumptions

This model assumes:

- infrastructure patch governance is maintained
- secrets are managed outside application code
- provider communications occur over authenticated encrypted transport
- corpus ingestion remains under trusted operational control

---

## Out of Scope

Not covered here:

- physical infrastructure security
- provider-side model security
- insider threat
- network-layer denial-of-service
- emerging long-horizon embedding inversion attacks

---

## Review Cadence

Reviewed:
- quarterly
- after architectural changes
- after material security findings
- when OWASP LLM guidance materially changes

---

## Related Artifacts

- [Redacted Findings Report](./artifacts/redacted-findings.md)
- [Remediation Decision Log](./artifacts/remediation-log.md)