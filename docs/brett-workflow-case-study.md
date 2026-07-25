# Brett Workflow Case Study

## The operational problem

Real-estate agents often identify paperwork needs while they are away from a computer. Traditionally, an agent must return to a desktop, identify the correct forms, re-enter transaction details, assemble the packet, configure recipients, and send it for signature. That interrupts field work and creates repeated administrative steps.

I designed **Brett** so a licensed EDP agent can initiate that work conversationally by SMS. Brett is not a legal decision-maker. It translates the agent's instructions into a structured transaction workflow, asks for missing information, prepares eligible documents, and routes the resulting packet through EDP's in-house e-signature system.

## What I designed

The workflow connects four operating surfaces:

1. **SMS intake** — the human agent describes the transaction and requested paperwork.
2. **Clarification** — Brett gathers missing terms in a compact follow-up rather than silently assuming them.
3. **Transaction and document preparation** — the system creates or associates the transaction, prepares eligible forms from the supplied terms, and assembles the packet.
4. **E-sign routing and auditability** — the packet is sent through EDP's signing workflow, where recipients, document versions, status, and an audit timeline are visible to staff.

I defined the business requirements, conversation flow, document dependencies, and acceptance criteria; directed AI coding assistants through implementation; and reviewed, tested, debugged, deployed, and operate the resulting workflow.

## Controlled end-to-end test

In July 2026, I recorded a fictional test using:

- a fictional buyer
- a fictional Evansville property
- a test email address
- buyer-agency and agency-policy documents
- explicit representation, compensation, limited-agency, and date instructions

The observed workflow was:

1. The agent requested a buyer packet by SMS.
2. Brett acknowledged the request and returned one consolidated set of missing questions.
3. The agent supplied the remaining terms.
4. Brett created a new transaction.
5. Brett assembled the eligible documents.
6. EDP's e-signature system created and sent the packet.
7. The EDP interface displayed the sent status, recipients, included documents, and audit event.

Firestore records independently confirmed the transaction, the Brett-filled form record, and the sent packet. The demonstration uses no real client or property information.

## Document dependency demonstrated by the test

The initial fictional request also mentioned a lead-based-paint disclosure. The correct result was **not** to create one.

A buyer-side lead-based-paint acknowledgment depends on a seller-completed disclosure supplied by the listing side. The seller first states whether they know of lead-based paint or related hazards and provides available reports or records; the buyer then acknowledges receiving that disclosure. Because the fictional property had no listing-side disclosure, Brett excluded the document rather than fabricating seller representations or presenting an unsupported acknowledgment to the buyer.

This is an important product behavior: successful automation is not measured by generating every document named in a message. It is measured by completing the valid workflow while respecting prerequisites and keeping the licensed professional responsible for the transaction.

## Human responsibility and boundaries

- The licensed agent supplies the transaction instructions and remains responsible for determining which documents are required.
- Brett asks for missing information but does not provide legal advice.
- Documents should only be prepared when their required source information and dependencies exist.
- The agent remains responsible for reviewing the transaction and resulting packet.
- Packet status and document activity remain visible through the human-facing EDP interface.

## Evidence and claims

A July 2026 audit found:

- 178 Brett-attributed SMS records: 57 inbound and 121 outbound
- 46 inbound queue items
- three e-sign packets created through Brett's prepare-and-send path: two completed and one sent
- one form record explicitly marked as filled by Brett
- one recorded fictional end-to-end test from SMS request through sent packet

These figures demonstrate a working integration path. They are not presented as customer-volume, revenue, or time-savings measurements. The message history includes development and rollout traffic.

## What this demonstrates

This project is evidence of the work I want to do in AI deployment and product delivery:

- translate an operational problem into a usable AI-assisted workflow
- design clarification and human-control points
- connect conversational AI to production systems of record
- verify behavior through both the user interface and underlying records
- recognize that correct automation sometimes means refusing or deferring a requested step
- communicate capabilities and limitations without overstating either
