# Security Review — Pre-Publication Audit

**Repository:** `edpapp-public`
**Audit date:** 2026-07-24 (revised)
**Status:** Prepared for owner review. **No Git repository initialized. Nothing published. No remote created. No visibility changed.**

This report documents the security review performed on this repository. It also records the accuracy revision pass in which payment claims, deployment claims, and AI-agent safety language were corrected and de-risked.

## Method

This repository was authored from scratch as sanitized documentation. **No files were copied from the private source repository** (`/Users/ajbowlds/edpapp`), which was treated as strictly read-only and used only to verify facts. Every file here is original prose, diagrams, or clearly fictional example data.

A revision pass then (a) corrected material claims against the private implementation, and (b) removed operationally sensitive detail — replacing enumerated weaknesses with higher-level language about ongoing work, on the principle that a public portfolio document should not double as an operational map of a live system.

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
| 11 | Imported Git history | **PASS** — this directory is **not a Git repository**; no history of any kind exists or was imported. |
| 12 | `.env` / credential files | **PASS** — none present. |
| 13 | `.gitignore` coverage | **PASS** — covers environment files, private keys, credential JSON, data exports/dumps/backups, logs, and (defensively) all image formats in `images/` except the README, so an un-reviewed screenshot cannot be committed by accident. |
| 14 | **Exploit-relevant operational detail** | **PASS after revision** — operationally sensitive detail was removed and replaced with high-level descriptions of ongoing hardening priorities. |

## Operational-detail review

Operationally sensitive implementation and security details were removed and replaced with high-level descriptions of ongoing hardening priorities. Implemented safeguards are stated only where they were verified; desirable-but-incomplete controls are described as active priorities rather than finished features.

## Accuracy corrections made in the revision pass

| Claim | Before | After |
|---|---|---|
| Title | "Architecture Case Study" | "Public Technical Overview"; "case study" removed from title and repo name |
| Rent collection | Described as a live rent-collection flow with Connect payouts | **Production-capable** rent-charge flow + Connect landlord-disbursement pipeline (including a held-funds state); no unsupported adoption or volume claim |
| Invoice payments | Conflated with rent | Stated separately as the **live** payment path |
| Vendor payouts / landlord disbursements | Implied live | Described only to the extent implemented, with adoption distinguished from capability |
| Lines of code | "200,000+ lines" | Removed — no documented figure exists |
| Agent count | "five to six agents have phone/SMS"; a 16-persona roster | Removed. Pierce is the documented property-management agent; no production agent count claimed |
| iOS status | "Live (App Store target)" | **TestFlight distribution**, not publicly released on the App Store; public release labeled Planned |
| Android status | "In-progress port" | **In limited internal use; not production-ready** |
| Third-party rent platform | Named explicitly | **Genericized** to "third-party property-management platform" at owner's request — vendor name removed from all files and diagrams |
| E-signatures | "legally binding" | "Audit-grade, tamper-evident" with ESIGN-consent capture; legal enforceability not claimed |
| Timeline | "more than a year of production operation"; "last several years" of this work | Development began **April 2025**; several years of prior business/real-estate operations experience stated separately |
| User scope | Included inferred vendor-directory nuance | Operator + 5 agents, ~10 landlords, ~20 tenants. **Vendor count corrected against a direct read-only Firestore aggregate query**: ~110 directory records with ~40 flagged active — the earlier "~20" figure was not supported by the data. Stated as two measures rather than one number, since directory size and active flag measure different things. No vendor records, names, or IDs were read or reproduced — counts only. |
| Test coverage | Exact per-app file counts | Relative coverage levels, with gaps stated as priorities |

## Deliberate inclusions (reviewed, judged safe)

- **The owner's real name** — a portfolio piece; the author identifying themselves is the point. Present in the README positioning statement and the LICENSE copyright line.
- **The business name (EDP Realty)** — already-public information a brokerage advertises. No street addresses anywhere.
- **Subproject / module names** (`edpmain`, `edpAgentNet`, etc.) — internal module names, not hosts, endpoints, or secrets.
- **The agent name "Pierce"** — a product persona name, not a credential or PII.
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

The repository contains no credentials, no real personal or customer data, no production identifiers, and no enumerated operational weaknesses. Material claims have been corrected against the private implementation, and features are labeled Live / Production-capable / Partial / Planned so that nothing reads as more deployed than it is. It is safe for the owner to review and, at the owner's sole discretion, publish.

**No Git repository was initialized. No remote was created. Nothing was committed, pushed, or published, and no visibility was changed.**
