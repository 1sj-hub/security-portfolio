# Remediation Decision Log — AskObi Production Environment

**Target:** AskObi production application host (`<production-host>`)
**Operating System:** Ubuntu 24.04 LTS x64
**Remediation Window:** 20 May 2026 – 21 May 2026
**Validation:** Tenable Nessus credentialed re-scan (21 May 2026)

---

## Remediation Approach

The remediation cycle followed standard Ubuntu Security Notice (USN) hygiene: patched packages were drawn from the Ubuntu 24.04 LTS security repository via the system package manager. The full upgrade was executed as:

```bash
sudo apt update
sudo apt upgrade
```

Followed by a controlled reboot to apply kernel and library updates that required process restart. A credentialed Nessus re-scan was then performed to verify resolution.

This approach was chosen over surgical per-package patching because:

1. All but one finding mapped to standard Ubuntu USN advisories, which are delivered through the same repository channel.
2. The system runs a single production application with well-understood dependencies; broad upgrade carries acceptable change risk.
3. A coordinated upgrade reduces the chance of leaving partial dependency states (a common source of follow-on findings).

Pre-upgrade state was captured via `dpkg -l` for rollback reference. Application functionality was verified post-upgrade against a smoke-test checklist before the re-scan was initiated.

---

## Per-Finding Action Record

| # | Plugin ID | Severity | Finding | Action | Status |
| - | --------- | -------- | ------- | ------ | ------ |
| 1 | 304014 | Critical | Mbed TLS — USN-8123-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 2 | 278076 | Critical | fontTools — USN-7917-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 3 | 198152 | High | FFmpeg — USN-6803-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 4 | 270676 | High | FFmpeg — USN-7823-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 5 | 237868 | High | GStreamer Bad Plugins — USN-7558-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 6 | 314214 | High | OpenEXR — USN-8259-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 7 | 206422 | High | FFmpeg — USN-6983-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 8 | 274433 | High | Python Library Brotli `<= 1.1.0` DoS | Risk accepted — see below | **Accepted** |
| 9 | 297270 | High | FFmpeg — USN-7982-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 10 | 271190 | High | FFmpeg — USN-7830-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 11 | 197836 | High | cJSON — USN-6784-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 12 | 233301 | High | zvbi — USN-7367-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 13 | 296748 | Medium | cJSON — USN-7973-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 14 | 310928 | Medium | wheel — USN-8221-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |
| 15 | 237485 | Medium | FFmpeg — USN-7538-1 | `apt upgrade` — Ubuntu 24.04 security repo | Remediated |

**Result:** 14 of 15 findings remediated. 1 finding (Plugin 274433, Brotli DoS) accepted as residual risk after validation of compensating control. See record below.

---

## Validation

A credentialed Nessus re-scan was performed on 21 May 2026 against the same host. The re-scan confirmed:

- 0 Critical findings (down from 2)
- 1 High finding (down from 10) — the accepted Brotli finding
- 0 Medium findings (down from 3)

All 14 remediated findings dropped from the report. The single residual finding matched the expected accepted-risk record below.

---

## Residual Risk Acceptance — Python Library Brotli DoS

### Finding

Nessus Plugin **274433** — `Python Library Brotli <= 1.1.0 DoS`. CVSS v3.0 `7.5` (High), EPSS `0.0004`.

### Decision

**Accepted residual risk with quarterly review.**

### Rationale

The AskObi application runtime imports Brotli `1.2.0` from its application virtual environment. The flagged system package remains installed at version `<= 1.1.0` due to operating-system dependency constraints (it is pulled in transitively by other system packages and cannot be removed without breaking unrelated functionality).

The system-level Brotli package is not used by the AskObi application process to deserialize attacker-controlled Brotli streams. The application's request-handling path resolves Brotli through its virtual environment, which contains the patched `1.2.0` release.

### Compensating Control

The application runtime does not use the vulnerable system package code path for processing attacker-controlled Brotli streams. The virtual-environment Brotli `1.2.0` is the only Brotli code path reachable from the application's request lifecycle.

### Review Triggers

This acceptance will be re-evaluated when any of the following occur:

- A new upstream Brotli security release supersedes `1.2.0`
- EPSS score for the associated CVE(s) rises above `0.01`
- The AskObi application's dependency path changes such that the system Brotli package becomes reachable
- An Ubuntu Security Notice publishes a patched system Brotli package for 24.04 LTS

### Next Scheduled Review

**Quarterly** — next review due August 2026.

### Acceptance Owner

Steeve Joseph (AskObi sole operator and security owner)

---

## Lessons Learned

**What worked:**

- Single-cycle remediation closed 14 of 15 findings in under 24 hours. The standard Ubuntu USN flow is highly effective when the host is current on its LTS release.
- Pre-upgrade `dpkg -l` snapshot gave a clean rollback reference, though it was not needed.
- Separating "package vulnerable" from "vulnerable code path reachable" allowed the Brotli finding to be accepted on factual grounds rather than ignored or force-patched into a broken state.

**What to improve:**

- Establish a recurring scan cadence (target: monthly credentialed scans) rather than ad-hoc assessment, so findings are caught closer to USN publication rather than accumulating between scans.
- Track virtual-environment package versions in a manifest committed to the repository, so dependency-path arguments (like the Brotli rationale) are documented before they are needed for a risk decision.

---

**Related Documents**

- [Redacted Findings Report](./redacted-findings.md) — full scan results, initial and post-remediation
