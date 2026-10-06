# Security Review — Public Repository Audit

**Repository:** `edpapp-public`
**Audit date:** 2026-10-06 (revised)
**Status:** Published. The current public contents were re-reviewed after the October 2026 architecture and evaluation update.

This report documents the security review performed on this repository. It also records accuracy revision passes in which payment claims, deployment claims, AI-agent controls, evaluation evidence, and system status were corrected and de-risked.

## Method

This repository was authored from scratch as sanitized documentation. **No files were copied from the private production repository**, which was treated as strictly read-only and used only to verify facts. Every file here is original prose, diagrams, or clearly fictional example data.

Revision passes then (a) corrected material claims against the private implementation, and (b) removed operationally sensitive detail while retaining enough architecture and evaluation evidence to explain the work. Public documentation should demonstrate implemented controls without becoming an operational map of a live system.

## Checklist results

| # | Check | Result |
|---|---|---|
| 1 | Email addresses | **PASS** — none. Reporting routes use GitHub private vulnerability reporting and LinkedIn. |
| 2 | Phone numbers | **PASS** — none (US and E.164 formats scanned). |
| 3 | API-key / token / secret patterns | **PASS** — none. |
| 4 | Assigned-value secrets (`password=`, `token:`, `apiKey=`) | **PASS** — none. |
| 5 | Private keys / credential files | **PASS** — none present; `.gitignore` blocks them defensively. |
| 6 | Production document IDs / auth UIDs | **PASS** — none. Example JSON uses obviously-fake IDs (`mreq_example_0001`). |
| 7 | Real customer / tenant / landlord / vendor / agent names | **PASS** — none. The only personal name is the author's own, intentionally. Example data uses historical figures (Ada Lovelace, Grace Hopper), unmistakably fictional in context. |
| 8 | Real property / street addresses | **PASS** — none. Example uses `123 Example St, Anytown, IN`. |
| 9 | Internal URLs / hostnames / project IDs | **PASS** — none. No real domains, no Cloud Run / Firebase Storage / RTDB hosts, no admin hostnames, no agent-runtime host paths, no Firebase project ID. |
| 10 | Symlinks | **PASS** — none. |
| 11 | Imported Git history | **PASS** — the public repository contains only the history created for this sanitized documentation; no history from the private production repository was imported. |
| 12 | `.env` / credential files | **PASS** — none present. |
| 13 | `.gitignore` coverage | **PASS** — covers environment files, private keys, credential JSON, data exports/dumps/backups, logs, and (defensively) all image formats in `images/` except the README, so an un-reviewed screenshot cannot be committed by accident. |
| 14 | **Exploit-relevant operational detail** | **PASS after revision** — controls are described structurally without endpoint paths, policy configuration, prompts, credential names, hosts, or a catalog of weaknesses. |

## Operational-detail review

Operationally sensitive implementation and security details were removed. Implemented safeguards are stated only where they were verified; continuing work is described as expanding coverage and automated verification rather than implying either that controls do not exist or that security is finished.

## Accuracy corrections made in the revision pass

| Claim | Before | After |
|---|---|---|
| Title | "Architecture Case Study" | "Public Technical Overview"; "case study" removed from title and repo name |
| Rent collection | Described as a live rent-collection flow with Connect payouts | **Production-capable** rent-charge flow + Connect landlord-disbursement pipeline (including a held-funds state); no unsupported adoption or volume claim |
| Invoice payments | Conflated with rent | Stated separately as the **live** payment path |
| Vendor payouts / landlord disbursements | Implied live | Described only to the extent implemented, with adoption distinguished from capability |
| Lines of code | "200,000+ lines" | Removed — no documented figure exists |
| Agent count | "five to six agents have phone/SMS"; a 16-persona roster | Pierce and Brett are documented in detail; additional deployed specialized roles are acknowledged without publishing a count, roster, prompts, contact details, or internal responsibilities |
| iOS status | "Live (App Store target)" | **TestFlight distribution**, not publicly released on the App Store; public release labeled Planned |
| Android status | "In-progress port" | **In limited internal use; not production-ready** |
| Third-party rent platform | Named explicitly | **Genericized** to "third-party property-management platform" at owner's request — vendor name removed from all files and diagrams |
| E-signatures | "legally binding" | "Audit-grade, tamper-evident" with ESIGN-consent capture; legal enforceability not claimed |
| Timeline | "more than a year of production operation"; "last several years" of this work | Development began **April 2025**; several years of prior business/real-estate operations experience stated separately |
| User scope | Treated changing user and vendor counts as portfolio evidence | Replaced exact counts with the supported user groups and a statement that usage figures are intentionally omitted because they change with the business |
| Test coverage | Exact per-app file counts | Relative coverage levels, with gaps stated as priorities |
| Agent-policy controls | Previously described primarily as future hardening | Updated to reflect verified OAuth-scoped access, shared role policy, confirmation and write gates, storage-path controls, audit records, and authenticated human review; ongoing work is expansion and testing |
| Agent evaluation | Not included in the original overview | Added a sanitized explanation of the versioned harness and its limitations without publishing changing suite totals |
| Realtime voice | Described only as a generic voice agent | Updated with high-level streaming, interruption, transcript, validation, and restricted-capability behavior without publishing endpoints or configuration |

## Deliberate inclusions (reviewed, judged safe)

- **The owner's real name** — authorship is intentionally public. Present in the README positioning statement and the LICENSE copyright line.
- **The business name (Euphoric Development Partners, abbreviated EDP)** — already-public information the brokerage advertises. No street addresses anywhere.
- **Subproject / module names** (`edpmain`, `edpAgentNet`, etc.) — internal module names, not hosts, endpoints, or secrets.
- **The agent name "Pierce"** — a product persona name, not a credential or PII.
- **The agent name "Brett"** — a product persona name already documented through a fictional workflow case study.
- **Approximate user counts** — aggregate, non-identifying business scale.
- **Third-party vendor names** (Stripe, Plaid, Twilio, SendGrid, OpenAI, Gemini) — standard stack disclosure, no configuration detail. The rent-platform vendor is deliberately **not** named, at the owner's request; it is referred to only as "a third-party property-management platform."

## Information deliberately omitted

Real names, emails, phone numbers, and addresses of all parties; Firestore document IDs and auth UIDs; collection contents; the Firebase project ID, production domains, and agent-runtime host; credential names and values; endpoint paths and payment-security specifics; real dollar figures, margins, and fee economics; proprietary agent system prompts; and operationally sensitive implementation details.

## Residual risk & standing guidance

- **Screenshots remain the largest residual risk.** None are committed. Text scanning cannot inspect image contents or EXIF metadata, so [`images/README.md`](images/README.md) requires human visual inspection and metadata stripping before any image is added; `.gitignore` blocks images by default as a backstop.
- **Re-run the scan below after any future edit**, especially one that adds a screenshot or pastes text from the private repository.

### Re-runnable scan

```bash
cd edpapp-public
grep -rInE 'sk_(live|test)_|pk_(live|test)_|AIza[0-9A-Za-z_-]{20,}|AKIA[0-9A-Z]{16}|-----BEGIN|eyJ[A-Za-z0-9_-]{10,}' . --include='*.md' --include='*.json' --include='*.mmd'
grep -rInE '[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}' . --include='*.md' --include='*.json' | grep -viE 'example\.com|add (link|email|contact)'
grep -rInE '\+1[0-9]{10}|\(?[0-9]{3}\)?[-. ][0-9]{3}[-. ][0-9]{4}' . --include='*.md' --include='*.json'
find . -type l
```

## Conclusion

The repository contains no credentials, no real personal or customer data, no production identifiers, and no enumerated operational weaknesses. The October 2026 update's secret, email, telephone, and symlink scans returned no findings beyond the scan pattern printed inside this review itself. Material claims were checked against the private implementation, and features remain labeled Live / Production-capable / Partial / Planned so that nothing reads as more deployed than it is. The current contents are approved for continued public use, subject to re-review after future changes.
