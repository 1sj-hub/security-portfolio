# RAG Threat Model — AskObi

**System:** AskObi — production retrieval-augmented generation platform
**Domain:** Civic intelligence (legislative documents, constitutions, federal regulations, ballot measures)
**Framework:** OWASP Top 10 for Large Language Model Applications
**Model Type:** Architectural threat model — public-safe abstraction
**Owner:** Steeve Joseph (AskObi sole operator and security owner)

---

## Disclosure Notice

This model is intentionally public-safe.

It reflects real architectural thinking from production AI system design without disclosing implementation-sensitive details such as:

- Framework internals
- Infrastructure topology
- Provider-specific configurations
- Retrieval implementation specifics
- Operational deployment details

The goal is to demonstrate security reasoning — not publish a blueprint for attackers.

---

## System Overview

AskObi is a retrieval-augmented generation platform that allows users to query a curated corpus of civic and legislative documents. A user-submitted natural-language question is processed through a pipeline that authenticates the request, sanitizes the prompt, retrieves semantically relevant passages from a vector store, filters retrieved content for source trust, generates a grounded response via an LLM, and applies output safety controls before delivery.

The threat model below treats each pipeline stage as a control point with one or more associated threats and mapped controls.

---

## Architecture — Threat & Control Pipeline

![AI Security Threat Model](../images/ai-security-threat.png)

The pipeline is organized into three trust zones:

- **Ingress Controls** — everything before retrieval begins
- **Retrieval Integrity** — protecting the retrieval and grounding layer
- **Response Governance** — protecting model output before user delivery

---

## Trust Boundaries

Three trust transitions define the security model:

**Boundary 1 — Untrusted user input enters the system**
The user query crosses from the public internet into the application at the API Gateway. All downstream stages must treat this input as adversarial until validation passes.

**Boundary 2 — Retrieved content enters the LLM context**
Documents returned from the vector store are concatenated into the LLM prompt. Even when the corpus is curated, retrieved content must be treated as semi-trusted because it can carry instructions that the LLM may interpret as commands (indirect prompt injection).

**Boundary 3 — Model output crosses back to the user**
LLM-generated content is treated as untrusted until it passes output guardrails. The model can produce hallucinated facts, unsafe content, or content that leaks system context.

---

## Pipeline Stages — Threats and Controls

### Stage 1 — API Gateway

| Attribute | Value |
| --- | --- |
| Threat | API Abuse |
| OWASP LLM Mapping | LLM04: Model Denial of Service · LLM10: Unbounded Consumption |
| Control | Rate limiting |
| Status | Implemented |

**Threat description.** Unauthenticated or low-cost requests can be issued in volume to exhaust compute budget, drive up provider costs, or degrade availability for legitimate users. RAG systems are particularly exposed because each request triggers embedding generation, vector search, and LLM inference — all of which carry meaningful per-request cost.

**Control rationale.** Rate limiting at the gateway caps request volume per client identity, contains cost-amplification attacks, and provides early signal of abuse patterns before they reach downstream stages.

**Residual risk.** Distributed abuse across many client identities can evade per-identity limits. Mitigated by aggregate-volume monitoring at the application layer.

---

### Stage 2 — Auth / Validation

| Attribute | Value |
| --- | --- |
| Threat | Unauthorized Access |
| OWASP LLM Mapping | LLM02: Sensitive Information Disclosure · LLM06: Excessive Agency |
| Control | Authentication controls |
| Status | Implemented |

**Threat description.** Without enforced authentication, any caller can access the system's full capabilities — including queries that may surface privileged corpus content or consume system resources beyond intended use. In RAG systems specifically, authentication is the gate that determines what slice of the corpus a caller is allowed to retrieve against.

**Control rationale.** Authentication and request validation establish the caller's identity and authorization scope before any retrieval or LLM operation begins. This is the foundation for downstream access boundaries at the vector store.

**Residual risk.** Compromised credentials remain a threat. Mitigated by monitoring for anomalous query patterns per identity.

---

### Stage 3 — Prompt Guardrails

| Attribute | Value |
| --- | --- |
| Threat | Prompt Injection |
| OWASP LLM Mapping | LLM01: Prompt Injection |
| Control | Query sanitization |
| Status | Implemented |

**Threat description.** A user query can attempt to override system instructions, exfiltrate the system prompt, or coerce the model into behaviors outside its intended scope. Direct prompt injection is the most well-documented attack class against LLM applications and the highest-priority OWASP LLM finding.

**Control rationale.** Query sanitization screens user input for patterns associated with injection attempts before the query enters the retrieval and generation pipeline. Sanitization runs before retrieval so that injected instructions cannot influence what content is pulled into context.

**Residual risk.** Sanitization cannot catch all adversarial phrasings. Defense-in-depth is provided by downstream output guardrails (Stage 9) which screen final responses for behaviors consistent with successful injection.

---

### Stage 4 — Retriever

| Attribute | Value |
| --- | --- |
| Threat | Retrieval Poisoning |
| OWASP LLM Mapping | LLM05: Improper Output Handling · LLM08: Vector and Embedding Weaknesses |
| Control | Input validation |
| Status | Implemented |

**Threat description.** Adversarial queries can be crafted to manipulate the retrieval step — for example, by embedding instructions designed to be picked up by similarity search, or by using query construction that surfaces unintended document segments. The retriever sits at the boundary between user intent and corpus content, making it a high-leverage manipulation target.

**Control rationale.** Input validation at the retriever layer enforces structural constraints on the query before semantic search executes. This narrows the surface for retrieval manipulation and ensures the search operates on well-formed input.

**Residual risk.** Semantic manipulation attacks that operate within structurally valid queries are harder to detect at this stage. Mitigated by the Source Trust Filter (Stage 7), which screens retrieved content regardless of how the query was constructed.

---

### Stage 5 — Vector Store

| Attribute | Value |
| --- | --- |
| Threat | Data Leakage |
| OWASP LLM Mapping | LLM02: Sensitive Information Disclosure · LLM08: Vector and Embedding Weaknesses |
| Control | Access boundaries |
| Status | Implemented |

**Threat description.** A vector store contains embedded representations of corpus documents. Without enforced access boundaries, a caller could retrieve content outside their authorized scope, or use carefully constructed queries to probe what is indexed. Embedding inversion attacks — where original content is partially reconstructed from embeddings — are an emerging concern documented in OWASP LLM08.

**Control rationale.** Access boundaries scope every retrieval operation to the authorized corpus partition for the caller's identity (established at Stage 2). The vector store does not return results outside the authorized boundary regardless of query content.

**Residual risk.** Inference attacks that probe boundary edges through repeated queries are partially mitigated by the rate limiting at Stage 1 and aggregate query monitoring.

---

### Stage 6 — Source Trust Filter

| Attribute | Value |
| --- | --- |
| Threat | Source Integrity |
| OWASP LLM Mapping | LLM03: Supply Chain · LLM04: Data and Model Poisoning |
| Control | Source verification |
| Status | Implemented |

**Threat description.** RAG systems are only as trustworthy as the documents in their corpus. If the corpus accepts content without provenance checks, an attacker who can influence ingestion can poison the corpus — embedding malicious instructions or false information that will later be retrieved and presented to users as grounded fact. This is one of the highest-impact attack classes against RAG systems because corpus poisoning persists across many user sessions.

**Control rationale.** The Source Trust Filter verifies that retrieved content originates from validated sources before passing it to the LLM context. Documents from unverified origins are excluded from generation regardless of relevance score.

**Residual risk.** Trusted sources can themselves be compromised upstream. Mitigated by source diversity (no single source can singularly anchor a response) and by output-stage audit logging that creates a forensic trail if a poisoned source is later identified.

---

### Stage 7 — LLM Response Layer

| Attribute | Value |
| --- | --- |
| Threat | Hallucination Risk |
| OWASP LLM Mapping | LLM09: Misinformation |
| Control | Audit logging |
| Status | Implemented |

**Threat description.** Even with grounded retrieval, the LLM can produce content that is not supported by the retrieved sources — confidently asserting facts the corpus does not contain. In a civic intelligence domain, hallucinations carry real downstream harm: users may make decisions based on fabricated legislative content.

**Control rationale.** Audit logging captures the retrieved context, the generated response, and the source attribution for every query. This creates a forensic trail that enables post-hoc detection of hallucinations and supports continuous quality measurement. Logging is the operational control that makes hallucination measurable rather than invisible.

**Residual risk.** Logging detects hallucinations after the fact, not before. The output guardrails at Stage 9 provide the pre-delivery screening layer.

---

### Stage 8 — Output Guardrails

| Attribute | Value |
| --- | --- |
| Threat | Unsafe Output |
| OWASP LLM Mapping | LLM05: Improper Output Handling · LLM02: Sensitive Information Disclosure |
| Control | Output controls |
| Status | Implemented |

**Threat description.** Model output can contain content that is unsafe to deliver to users — including responses shaped by successful prompt injection upstream, content that leaks system context, or responses that include unverified claims presented as authoritative. Output is the last point at which the system can intervene before content reaches the user.

**Control rationale.** Output controls screen generated responses against safety and integrity criteria before delivery. This stage catches failures that slipped through earlier controls — a defense-in-depth role rather than a primary defense.

**Residual risk.** Output controls operate on completed responses and cannot prevent the model from being influenced by upstream attacks — they can only prevent the resulting unsafe content from being delivered. The upstream prompt guardrails (Stage 3) remain the primary control for injection.

---

## OWASP LLM Top 10 Coverage Matrix

| OWASP Category | Covered By |
| --- | --- |
| LLM01: Prompt Injection | Stage 3 (Prompt Guardrails) + Stage 8 (Output Guardrails) |
| LLM02: Sensitive Information Disclosure | Stage 2 (Auth) + Stage 5 (Access Boundaries) + Stage 8 (Output Controls) |
| LLM03: Supply Chain | Stage 6 (Source Trust Filter) |
| LLM04: Data and Model Poisoning | Stage 6 (Source Trust Filter) |
| LLM05: Improper Output Handling | Stage 4 (Input Validation) + Stage 8 (Output Controls) |
| LLM06: Excessive Agency | Stage 2 (Auth Controls) |
| LLM07: System Prompt Leakage | Stage 3 (Prompt Guardrails) + Stage 8 (Output Controls) |
| LLM08: Vector and Embedding Weaknesses | Stage 4 (Input Validation) + Stage 5 (Access Boundaries) |
| LLM09: Misinformation | Stage 6 (Source Trust Filter) + Stage 7 (Audit Logging) |
| LLM10: Unbounded Consumption | Stage 1 (Rate Limiting) |

All ten OWASP LLM Top 10 categories are addressed by at least one implemented control. Categories with elevated residual risk (LLM01, LLM09) are addressed by multiple controls in defense-in-depth configurations.

---

## Defense-in-Depth Summary

The architecture treats no single stage as sufficient. Key defense-in-depth pairings:

- **Prompt injection** is screened at ingress (Stage 3) and again at egress (Stage 8), so a query that evades sanitization can still be caught when its effect surfaces in the response.
- **Sensitive information disclosure** is constrained by authentication (Stage 2), access boundaries (Stage 5), and output screening (Stage 8) — three independent layers.
- **Source integrity and misinformation** are addressed both pre-generation (Stage 6 source filtering) and post-generation (Stage 7 audit logging) so that poisoning is both blocked and discoverable.

---

## Assumptions

This threat model assumes:

- The host operating system and runtime dependencies are patched on a recurring cadence (see [redacted-findings.md](./redacted-findings.md) and [remediation-log.md](./remediation-log.md) for the current vulnerability posture).
- The LLM provider's API endpoints are reachable over authenticated, TLS-protected transport.
- Operational secrets (API keys, credentials) are managed outside the application code path.
- The corpus ingestion process is operated by the same trusted party that operates the production system.

---

## Out of Scope

This threat model does not address:

- Physical security of the underlying infrastructure
- Provider-side LLM model security (treated as a trust assumption)
- Insider threat from the system operator
- Long-term embedding inversion attacks against the vector store (monitored as an emerging research area; no production-grade mitigation exists at this time)
- Denial-of-service at the network layer below the API gateway

---

## Review and Maintenance

This threat model is reviewed:

- Quarterly, as part of recurring security review
- When new pipeline stages are introduced
- When the OWASP LLM Top 10 is updated by OWASP
- After any security finding that touches a pipeline stage covered here

**Next scheduled review:** August 2026

---

**Related Documents**

- [Redacted Findings Report](./redacted-findings.md) — host vulnerability posture
- [Remediation Decision Log](./remediation-log.md) — patch governance and residual risk acceptance
