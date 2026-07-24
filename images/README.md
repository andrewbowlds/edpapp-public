# Screenshots

This folder is intentionally empty of screenshots. **Do not add a screenshot until it has been redacted or recreated with fictional data**, following the checklist below. A single un-redacted screenshot can leak more private data than all the prose in this repo combined.

## The rule

Every screenshot must contain **zero** real:

- Names (agents, landlords, tenants, vendors, leads, buyers, sellers, staff)
- Email addresses and phone numbers
- Physical / property addresses (street, unit, city+ZIP that identifies a real property)
- Dollar amounts tied to a real transaction, lease, or invoice
- Firestore document IDs, auth UIDs, or URLs containing IDs
- Bank account nicknames, last-4 digits, routing/account numbers
- Internal admin URLs or infrastructure hostnames
- Anything visible in a browser tab, address bar, notification, or background window

## How to produce safe screenshots (in order of preference)

1. **Best: use a seeded demo account with fictional data.** Create a demo tenant/landlord/property with obviously fake names ("Ada Lovelace", "123 Example St", "Demo Property A") and screenshot that. The system already has a demo-account concept that scopes to demo-flagged data, which makes this clean.
2. **Next best: recreate the screen in a throwaway environment** (local emulator) with fictional data.
3. **Last resort: redact a real screenshot.** Use solid opaque boxes (not blur — blur can sometimes be reversed) over every field listed above. Crop out the address bar and browser chrome. Then have a second person check it.

## Recommended screenshots to capture (with fictional/demo data)

These would strengthen the overview without adding risk. Suggested filenames so the README can reference them later:

| Suggested file | What to show | Redaction notes |
|---|---|---|
| `agent-crm-dashboard.png` | The agent CRM lead/transaction dashboard | Fictional lead names, fake deal values, no real UIDs in URL |
| `property-mgmt-portal.png` | Property-management portal — a property with units/leases | Fictional property "123 Example St", demo tenant names |
| `maintenance-request.png` | A maintenance request detail + escalation timeline | Fictional tenant, fake unit, no real vendor names |
| `pierce-activity-log.png` | The AI-agent activity/audit view (calls/texts today/this week) | Fake counts fine; scrub any real numbers/names in call logs |
| `esign-signer-view.png` | The e-signature signer experience on a demo document | Use a fictional "Example Lease Agreement", fake signer |
| `esign-audit-trail.png` | The hash-chained audit trail of a completed signing | Demo document only |
| `billing-invoice.png` | An invoice with line items | Fictional customer, fake amounts, fake invoice number |
| `mobile-home.png` | The iOS app role-based home screen | Demo account login; scrub the status bar if it shows anything |
| `architecture-rendered.png` | A rendered version of `diagrams/ecosystem-architecture.mmd` | No redaction needed — it's already sanitized |

## After adding any screenshot

Re-run the security scan described in [`../SECURITY_REVIEW.md`](../SECURITY_REVIEW.md) — image contents won't be caught by a text `grep`, so a **human must visually inspect every image** before it's committed. Note: image files can also carry EXIF metadata (location, device, original filename); strip metadata before committing (`exiftool -all= file.png` or an equivalent) so a "clean-looking" screenshot doesn't leak data in its headers.
