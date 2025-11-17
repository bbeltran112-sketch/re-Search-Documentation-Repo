# Technical Documentation

**Navigation:**  
[Home](../README.md) › Technical Documentation

This section contains all technical, API, behavioral, and system-level documentation for the re:SearchTX platform. It is intended for CMS vendors, EFSPs, court IT partners, and BIS internal staff who support integrations, message validation, indexing behavior, and security troubleshooting.

The Technical Documentation is organized into four major areas:

- API Reference  
- Support Playbook  
- System Behavior & Integration Architecture  
- Troubleshooting, FAQs, and XML Library

Each section is linked below.

---

## API Reference

The API Reference documents the core services and message contracts used by re:SearchTX, including the authoritative case and document APIs and the event-driven CMS → EFM → re:Search flows.

| API | Description |
|-----|-------------|
| [API Reference Overview](./api-reference/README.md) | Entry point for all APIs |
| [Common Headers & Auth](./api-reference/common-headers-and-auth.md) | SOAP headers, authentication, and transport |
| [Error Codes](./api-reference/error-codes.md) | Error codes and failure behavior |
| [RecordFiling](./api-reference/recordfiling/README.md) | EFM → CMS filing submission |
| [NotifyDocketingComplete](./api-reference/notifydocketingcomplete/README.md) | CMS docketing acknowledgment |
| [NotifyCaseEvent](./api-reference/notifycaseevent/README.md) | CMS-originated updates (case, document, parties, security) |
| [GetCase](./api-reference/getcase/README.md) | Authoritative case metadata provider |
| [GetDocument](./api-reference/getdocument/README.md) | Document retrieval for UI display |

Each API page contains:
- Behavior overview  
- Required fields  
- EventType and security considerations  
- Inline XML placeholders  
- A link to the full XML Library

---

## Support Playbook

The Support Playbook provides a comprehensive view of how re:SearchTX processes updates, evaluates security, indexes content, and applies JCIT rules. It is the primary resource for diagnosing visibility, data mismatch, and indexing issues.

| Section | Description |
|---------|-------------|
| [Playbook Index](./support-playbook/README.md) | Entry point for all support documentation |
| [JCIT Roles](./support-playbook/jcit-roles.md) | Responsibilities and visibility constraints |
| [Security Logic](./support-playbook/security-logic.md) | CaseSecurity and DocumentSecurity behavior |
| [Registered User Matrix](./support-playbook/registered-user-matrix.md) | JCIT case-type visibility rules |
| [Delays & Special Rules](./support-playbook/delays-and-special-rules.md) | 31-day, 180-day, protected style, criminal restrictions |
| [EventType Logic](./support-playbook/eventtype-logic.md) | How EventType drives processing |
| [Integration Architecture](./support-playbook/integration-architecture.md) | CMS/EFM/DAS/re:Search system flow |
| [Troubleshooting Guide](./support-playbook/troubleshooting.md) | Step-by-step diagnostic workflow |
| [Common CMS Mistakes](./support-playbook/common-cms-mistakes.md) | Frequent vendor errors |
| [System Behavior Examples](./support-playbook/system-behavior-examples.md) | Real-world visibility and XML patterns |
| [Escalation Guidelines](./support-playbook/escalation-guidelines.md) | When/how to escalate internally |
| [XML Examples](./support-playbook/xml-examples.md) | Deprecated—use the XML Library instead |
| [References](./support-playbook/references.md) | Standards and documentation sources |

---

## XML Library

The XML Library is the central source of truth for message format examples.  
It contains correct and incorrect structures for:

- NotifyCaseEvent  
- CaseSecurity updates  
- DocumentSecurity updates  
- GetCaseResponse structures  
- RecordDocketingCallbackMessage  
- Minimal/invalid patterns  
- Vendor error patterns

Link:  
[XML Library](./xml-library/xml-library.md)

---

## FAQs

The FAQ consolidates the most common vendor and court questions related to:

- Security visibility  
- JCIT rules  
- Why cases do/do not appear  
- Padlock behavior  
- Criminal document access  
- Family case restrictions  
- EventType failures  

Link:  
[re:SearchTX Visibility & Access FAQ](./faq/re-search-faq.md)

---

## Integration Behavior & System Architecture

Additional deep-dives covering:

- How indexing works  
- How UI behavior maps to API inputs  
- Visibility logic detail  
- Architecture flows and placeholders for diagrams  

These topics are distributed throughout the Support Playbook and API Reference.

---

## Related Topics

- [Support Playbook](./support-playbook/README.md)  
- [API Reference](./api-reference/README.md)  
- [FAQ](./faq/re-search-faq.md)  
- [XML Library](./xml-library/README.md)
