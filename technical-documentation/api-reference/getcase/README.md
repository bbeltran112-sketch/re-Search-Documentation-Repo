# GetCase

**Navigation:**  
[Home](../../../README.md) › [Technical Documentation](../../README.md) › [API Reference](../README.md) › GetCase

GetCase returns authoritative case metadata, including filings, parties, attorneys, docket entries, and security.  
re:Search evaluates only the XML elements relevant to the EventType that triggered the lookup.

---

## Purpose

GetCase provides the complete, authoritative CMS dataset so re:Search can:

- Reindex case details  
- Render accurate UI information  
- Evaluate security and visibility  
- Update filings, parties, documents, and metadata  

---

## Transport and Protocol

| Property | Value |
|---------|--------|
| Direction | re:Search → CMS |
| Protocol | SOAP 1.2 over HTTPS |
| Security | Mutual TLS |
| Standard | OASIS ECF |
| Message Type | GetCaseRequest / GetCaseResponse |

Authentication details:  
[Common Headers & Auth](../common-headers-and-auth.md)

---

## When This API Is Used

GetCase is called immediately after:

- A NotifyCaseEvent is received  
- A NotifyDocketingComplete is processed  
- A system-wide reindex operation is triggered  

---

## Behavior Summary

re:Search:

- Calls GetCase with the CaseTrackingID and CourtIdentifier  
- Receives the full case dataset  
- Processes only the XML sections relevant to the originating EventType  
- Updates search indexes and UI accordingly  

Incorrect or incomplete GetCaseResponse often results in:

- Visibility mismatches  
- Missing filings  
- Incorrect security  

---
## Processing Workflow

### NotifyCaseEvent Trigger (ECF mode)
```mermaid
sequenceDiagram
    participant CMS as Case Management System (Court)
    participant EFM as E-Filing Manager
    participant MQ as RabbitMQ
    participant RS as re:Search

    Note over CMS: An event occurs in the case lifecycle
    CMS->>EFM: NotifyCaseEvent (eventType, case identifiers)
    EFM->>MQ: Publish event message
    MQ-->>RS: Deliver event message
    Note over RS: Event consumed<br/>Determine follow-up actions
    RS->>EFM: GetCase() and/or GetDocument()
    EFM->>CMS: Proxy request
    CMS-->>EFM: Case/Document data
    EFM-->>RS: Return requested data
    Note over RS: Update case index and UI
```
---
### NotifyDocketingComplete Trigger (ECF mode)
```mermaid
sequenceDiagram
    participant CMS as Clerk Case Management System
    participant EFM as E-Filing Manager
    participant RS as re:Search

    Note over CMS: Case is fully docketed
    CMS->>EFM: NotifyDocketingComplete
    EFM-->>CMS: Message Receipt (ack)
    Note over EFM: Notify re:Search of new/updated case
    EFM->>RS: Event notification (new case recorded)
    Note over RS: Pull filing metadata
    RS->>EFM: GetCase (Request for filing metadata)
    EFM->>CMS: Proxy GetCase
    CMS-->>EFM: CaseResponseMessage (filing payload)
    EFM-->>RS: CaseResponseMessage (filing payload)
    Note over RS: Index filing and refresh UI
```
---
### RecordFiling Trigger (ECF mode)
```mermaid
sequenceDiagram
    participant EFM as E-Filing Manager
    participant CMS as Clerk Case Management System
    participant RS as re:Search

    Note over EFM: Instruct CMS to officially record the filing
    EFM->>CMS: RecordFiling (filing package)
    CMS-->>EFM: Message Receipt / success
    Note over EFM: Notify re:Search that the case/filing was recorded
    EFM->>RS: NotifyNewCase (or similar)
    Note over RS: Request initial case data for first index
    RS->>CMS: GetCase (initial case metadata) via EFM
    CMS-->>RS: CaseResponseMessage (case metadata) via EFM
    Note over RS: Create initial case in index
```
---


## XML Example
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:wsdl:WebServiceMessagingProfile-Definitions-4.0"
                  xmlns:tyler="urn:tyler:ecf:extensions:GetCaseMessage"
                  xmlns:nc="http://niem.gov/niem/niem-core/2.0">
  <soapenv:Header/>
  <soapenv:Body>
    <ecf:GetCaseRequest>
      <tyler:GetCaseRequestMessage>

        <!-- [REQUIRED] unique per request -->
        <tyler:MessageID>0f1c68b3-8a55-4f7c-a41a-123456789abc</tyler:MessageID>

        <!-- [REQUIRED] UTC -->
        <tyler:RequestDateTime>2025-11-11T10:30:00Z</tyler:RequestDateTime>

        <!-- [REQUIRED] mapped court/jurisdiction -->
        <tyler:CourtIdentification>
          <nc:IdentificationID>Hopkins:cc</nc:IdentificationID>
        </tyler:CourtIdentification>

        <!-- [REQUIRED] authoritative CMS case id -->
        <tyler:CaseTrackingID>CASE-12345678</tyler:CaseTrackingID>

        <!-- [OPTIONAL][RECOMMENDED] include doc metadata for Document correlation -->
        <tyler:IncludeDocumentMetadata>true</tyler:IncludeDocumentMetadata>

        <!-- [OPTIONAL][RECOMMENDED] include parties/attorneys -->
        <tyler:IncludePartyInformation>true</tyler:IncludePartyInformation>

        <!-- [NEW][OPTIONAL] return only changes since timestamp (if supported by your CMS) -->
        <!-- <tyler:SinceDateTime>2025-10-01T00:00:00Z</tyler:SinceDateTime> -->

      </tyler:GetCaseRequestMessage>
    </ecf:GetCaseRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

---

## Example XML Files

| File | Description |
|------|-------------|
| [getcase-request-annotated.xml](./examples/getcase-request-annotated.xml) | Annotated request |
| [getcase-response-annotated.xml](./examples/getcase-response-annotated.xml) | Annotated response |
| [getcase-request-withpartiesanddocuments.xml](./examples/getcase-request-withpartiesanddocuments.xml) | Request with full detail |
| [getcase-request-partys.xml](./examples/getcase-request-partys.xml) | Party-only request |

---

## Related API Pages

- [RecordFiling](../recordfiling/README.md)  
- [NotifyDocketingComplete](../notifydocketingcomplete/README.md)  
- [NotifyCaseEvent](../notifycaseevent/README.md)  
- [GetDocument](../getdocument/README.md)  

---

## Related Topics

- [Security Logic](../../support-playbook/security-logic.md)
- [Registered User Matrix](../../support-playbook/registered-user-matrix.md)
- [Troubleshooting Guide](../../support-playbook/troubleshooting.md)
- [Full XML Library](../../xml-library/xml-library.md)