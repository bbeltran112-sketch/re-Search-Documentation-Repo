# API Reference

**Navigation:**  
[Home](../../README.md) › [Technical Documentation](../README.md) › API Reference

The API Reference defines the core web services used by re:SearchTX.  
All APIs follow OASIS ECF standards, use SOAP 1.2 over HTTPS, and rely on mutual TLS authentication.

Each API page includes:

- Purpose and behavior summary  
- Message structure and schema notes  
- XML example placeholder  
- Links to canonical XML samples  
- Related API pages  
- Related troubleshooting and security documentation  

This index serves as the entry point into all re:SearchTX integration APIs.

---

## Purpose of the re:Search API Set

These APIs enable:

- CMS → re:Search case synchronization  
- EFM → CMS filing and docketing workflow  
- Document retrieval and visibility enforcement  
- Event-driven updates (EventType logic)  
- Indexing, UI updates, and JCIT visibility compliance  

All APIs work in coordination.  
Example:  
NotifyCaseEvent → GetCase → update re:Search index.

---

## Transport and Protocol

| Property | Value |
|---------|--------|
| Protocol | SOAP 1.2 over HTTPS |
| Security | mTLS |
| Standard | OASIS ECF |
| Format | XML |
| Authentication | Required; configured per environment |

Authentication requirements:  
[Common Headers & Auth](./common-headers-and-auth.md)

---

## Core APIs

| API | Description |
|-----|-------------|
| [NotifyCaseEvent](./notifycaseevent/README.md) | CMS → re:Search event notifications |
| [GetCase](./getcase/README.md) | Authoritative case metadata retrieval |
| [GetDocument](./getdocument/README.md) | Document binary retrieval |
| [RecordFiling](./recordfiling/README.md) | EFM → CMS filing submission |
| [NotifyDocketingComplete](./notifydocketingcomplete/README.md) | Filing docket confirmation |

These APIs are tightly linked.  
For example:  
RecordFiling → NotifyDocketingComplete → re:Search calls GetCase.

---

## Shared Artifacts

| Section | Description |
|--------|-------------|
| [Common Headers & Auth](./common-headers-and-auth.md) | SOAP transport requirements |
| [Error Codes](./error-codes.md) | Error schema and handling |

---

## Behavior Expectations

All APIs follow these core behavior rules:

- re:Search always calls GetCase after receiving an event  
- Only XML elements relevant to EventType are processed  
- Security updates must use correct CaseSecurity/DocumentSecurity values  
- Incorrect or missing sections in XML → visibility issues  

For full behavior logic, see the Support Playbook.

---

## Example XML Files

All canonical XML samples are found in the XML Library:

[Full XML Library](../xml-library/xml-library.md)

---

## Related API Pages

- [NotifyCaseEvent](./notifycaseevent/README.md)  
- [GetCase](./getcase/README.md)  
- [GetDocument](./getdocument/README.md)  
- [RecordFiling](./recordfiling/README.md)  
- [NotifyDocketingComplete](./notifydocketingcomplete/README.md)  

---

## Related Topics

- [EventType Logic](../support-playbook/eventtype-logic.md)  
- [Security Logic](../support-playbook/security-logic.md)  
- [Troubleshooting Guide](../support-playbook/troubleshooting.md)  
- [Integration Architecture](../support-playbook/integration-architecture.md)  
- [Full XML Library](../xml-library/xml-library.md)
