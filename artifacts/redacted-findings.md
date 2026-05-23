# Redacted Findings Report — AskObi Production Environment

**Assessment Type:** Credentialed Nessus vulnerability assessment
**Target:** AskObi production application host (`<production-host>`)
**Operating System:** Ubuntu 24.04 LTS x64
**Initial Scan:** AskObi Initial Scan — 20 May 2026
**Post-Remediation Scan:** AskObi Post-Remediation Scan — 21 May 2026
**Scanner:** Tenable Nessus

---

## Summary

A credentialed Nessus scan was performed against the AskObi production application host on 20 May 2026. The scan identified 15 actionable findings (2 Critical, 10 High, 3 Medium) alongside 80 informational items. Remediation was executed within a single cycle, and a follow-up scan on 21 May 2026 confirmed resolution of 14 of 15 findings. One residual High finding (Python Library Brotli DoS) was accepted with documented compensating controls — see [remediation-log.md](./remediation-log.md) for the decision record.

---

## Initial Scan Results — 20 May 2026

| Severity      | Count |
| ------------- | ----- |
| Critical      | 2     |
| High          | 10    |
| Medium        | 3     |
| Low           | 0     |
| Informational | 80    |

### Critical Findings

| Plugin ID | CVSS v3.0 | EPSS   | Finding |
| --------- | --------- | ------ | ------- |
| 304014    | 9.8       | 0.006  | Ubuntu 18.04 / 20.04 / 22.04 / 24.04 LTS — Mbed TLS vulnerabilities (USN-8123-1) |
| 278076    | 9.8       | 0.0053 | Ubuntu 22.04 / 24.04 / 25.04 / 25.10 — fontTools vulnerabilities (USN-7917-1) |

### High Findings

| Plugin ID | CVSS v3.0 | EPSS   | Finding |
| --------- | --------- | ------ | ------- |
| 198152    | 8.8       | 0.0045 | Ubuntu 16.04 / 18.04 / 20.04 / 22.04 / 23.10 / 24.04 — FFmpeg vulnerabilities (USN-6803-1) |
| 270676    | 8.8       | 0.0012 | Ubuntu 16.04 / 18.04 / 20.04 / 22.04 / 24.04 — FFmpeg vulnerabilities (USN-7823-1) |
| 237868    | 8.8       | 0.0147 | Ubuntu 20.04 / 22.04 / 24.04 / 25.04 — GStreamer Bad Plugins vulnerabilities (USN-7558-1) |
| 314214    | 7.8       | 0.0003 | Ubuntu 16.04 / 18.04 / 20.04 / 22.04 / 24.04 / 26.04 — OpenEXR vulnerabilities (USN-8259-1) |
| 206422    | 7.8       | 0.0003 | Ubuntu 16.04 / 18.04 / 20.04 / 22.04 / 24.04 — FFmpeg vulnerability (USN-6983-1) |
| 274433    | 7.5       | 0.0004 | Python Library Brotli `<= 1.1.0` DoS |
| 297270    | 7.5       | 0.0004 | Ubuntu 16.04 / 18.04 / 20.04 / 22.04 / 24.04 / 25.10 — FFmpeg vulnerabilities (USN-7982-1) |
| 271190    | 7.5       | 0.0027 | Ubuntu 18.04 / 20.04 / 22.04 / 24.04 — FFmpeg vulnerabilities (USN-7830-1) |
| 197836    | 7.5       | 0.0062 | Ubuntu 22.04 / 23.10 / 24.04 — cJSON vulnerabilities (USN-6784-1) |
| 233301    | 7.3       | 0.001  | Ubuntu 16.04 / 18.04 / 20.04 / 22.04 / 24.10 — zvbi vulnerabilities (USN-7367-1) |

### Medium Findings

| Plugin ID | CVSS v3.0 | EPSS   | Finding |
| --------- | --------- | ------ | ------- |
| 296748    | 5.5       | 0.0007 | Ubuntu 20.04 / 22.04 / 24.04 / 25.10 — cJSON vulnerabilities (USN-7973-1) |
| 310928    | 5.5       | 0.0002 | Ubuntu 24.04 LTS — wheel vulnerability (USN-8221-1) |
| 237485    | 5.3       | 0.001  | Ubuntu 16.04 / 18.04 / 20.04 / 22.04 / 24.04 / 25.04 — FFmpeg vulnerabilities (USN-7538-1) |

---

## Finding Analysis

### Threat Surface Overview

The actionable findings clustered into four categories:

**Media processing libraries (8 findings)** — FFmpeg, GStreamer Bad Plugins, OpenEXR, and zvbi accounted for the majority of High-severity exposure. None of these libraries are directly invoked by the AskObi application runtime; they are pulled in as system-level transitive dependencies. Exposure is therefore latent rather than active, but unpatched packages remain a defense-in-depth concern.

**TLS and cryptographic libraries (1 finding)** — Mbed TLS (USN-8123-1) at CVSS 9.8 represented the highest-severity exposure. Patching was prioritized first.

**Font and parser libraries (4 findings)** — fontTools and cJSON vulnerabilities reflect typical Ubuntu transitive surface; patched in routine `apt upgrade` cycle.

**Python ecosystem (2 findings)** — Brotli (`<= 1.1.0` DoS) and `wheel` (USN-8221-1). The Brotli finding required separate analysis since the AskObi application runtime imports Brotli from a virtual environment, not the system package path — see residual risk record in [remediation-log.md](./remediation-log.md).

### EPSS Observations

All findings had EPSS scores below `0.02`, indicating low observed exploitation in the wild at the time of scan. EPSS was used as a secondary prioritization signal; primary prioritization was driven by CVSS severity and exposure context (whether the vulnerable code path was reachable from attacker-controlled input).

---

## Post-Remediation Scan Results — 21 May 2026

| Severity      | Count | Change      |
| ------------- | ----- | ----------- |
| Critical      | 0     | −2 (−100%)  |
| High          | 1     | −9 (−90%)   |
| Medium        | 0     | −3 (−100%)  |
| Low           | 0     | 0           |
| Informational | 79    | −1          |

### Residual Findings

| Plugin ID | Severity | CVSS v3.0 | EPSS   | Finding | Disposition |
| --------- | -------- | --------- | ------ | ------- | ----------- |
| 274433    | High     | 7.5       | 0.0004 | Python Library Brotli `<= 1.1.0` DoS | Accepted — see remediation log |

---

## Remediation Outcome

- **15** actionable findings identified
- **14** remediated in first remediation cycle
- **1** residual High accepted with documented compensating controls and quarterly review
- **100%** Critical findings resolved
- **100%** Medium findings resolved
- **93.3%** reduction in actionable findings

---

## Redaction Notes

This report has been redacted for public release. The following have been removed or generalized:

- Production host IP address (replaced with `<production-host>`)
- Internal hostnames and scan identifiers
- Any artifact that would assist external reconnaissance against the live environment

Plugin IDs, CVSS scores, EPSS values, and Ubuntu Security Notice (USN) references are preserved as-is because they are public Tenable and Ubuntu identifiers and do not expose the environment.

---

**Related Documents**

- [Remediation Decision Log](./remediation-log.md) — per-finding action record and residual risk 
