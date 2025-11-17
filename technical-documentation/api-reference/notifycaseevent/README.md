# NotifyCaseEvent API

The **NotifyCaseEvent** (NCE) operation is used by the **CMS** to inform **re:Search** that case data has changed.  
Each notification includes an `eventType` that tells re:Search **exactly what kind of update occurred** and which portions of the `GetCaseResponse` should be processed.

NotifyCaseEvent is required in all **ECF Mode** integrations and is the primary mechanism for communicating CMS-originated updates that occur **outside of the e-filing workflow**.

> **Important:** re:Search only processes XML segments that correspond to the declared `eventType`.  
> All other data in GetCaseResponse is ignored.

---

## Purpose

NotifyCaseEvent allows the CMS to:

- Trigger re:Search synchronization when any case data changes  
- Tell re:Search which parts of the case were updated  
- Ensure filings, docket entries, parties, attorneys, security, and metadata stay in sync  
- Maintain parity between CMS and re:Search indexes  

---

## Transport and Protocol

| Property | Value |
|---------|--------|
| **Direction** | CMS → re:Search |
| **Protocol** | SOAP 1.2 over HTTPS |
| **Security** | Mutual TLS (mTLS) |
| **Standard** | OASIS ECF 4.x |
| **Message Type** | NotifyCaseEventMessage |

Header requirements:  
➡️ [Common Headers & Auth](../common-headers-and-auth.md)

---

## When NotifyCaseEvent Occurs

CMS sends NotifyCaseEvent when case updates occur **outside** the e-filing workflow such as:

- Clerk-entered filings  
- Party/attorney updates  
- Case security changes (sealed/unsealed)  
- Disposition updates  
- Judge or location reassignment  
- Document security updates  
- Case deletions or expungements  

---

## Supported Event Types

Detailed descriptions and processing rules for each event type are here:  
➡️ [Event Types Reference](./event-types.md)

| EventType | Description |
|-----------|-------------|
| `CaseFiling` | Filing or docket entry added outside e-filing |
| `CaseParty` | Party changes |
| `CasePartyAttorney` | Party-attorney assignments |
| `CaseReassigned` | Judge/court/location changes |
| `CaseDisposition` | Case status or disposition updated |
| `CaseSecurity` | Case sealed/unsealed/confidential/public |
| `DocumentSecurity` | Document confidentiality/visibility changed |
| `CaseDelete` | Case marked deleted or restored |
| `CaseExpunge` | Case expunged or restricted |

---

## High-Level Workflow

diagram placeholder

### Steps

1. **CMS Modified Case Data**  
   Clerk or automated workflow updates the case.

2. **CMS Sends NotifyCaseEvent**  
   With an accurate `eventType` that reflects the change.

3. **re:Search Calls GetCase**  
   Retrieves the **full case dataset**.

4. **re:Search Indexes Updated Data**  
   Only the XML segments that correspond to the event type are processed.

5. **UI Updates**  
   Search portal reflects new filings, security, parties, or other changes.

---

## Required Message Elements

| Element | Description | Required |
|--------|-------------|----------|
| `CaseTrackingID` | CMS case identifier | Yes |
| `CourtIdentifier` | Routing/county code | Yes |
| `EventType` | Case update type | Yes |
| `EventDate` | Timestamp | Yes |
| `FilingID` | Required for CaseFiling events | Conditional |

---

## Behavior Specifications

To understand exactly how re:Search maps `eventType` to GetCaseResponse processing logic, see:

➡️ [Do / Don’t Guide](./do-dont-guide.md)  
➡️ [Event Types Reference](./event-types.md)

These documents are mandatory for all CMS vendor integrations.

---

## Examples

Example XML files:

| Event Type | Example Folder |
|------------|----------------|
| **CaseFiling** | [examples/case-filing](./examples/case-filing) |
| **CaseParty** | [examples/case-party](./examples/case-party) |
| **CasePartyAttorney** | [examples/case-party-attorney](./examples/case-party-attorney) |
| **CaseReassigned** | [examples/case-reassigned](./examples/case-reassigned) |
| **CaseDisposition** | [examples/case-disposition](./examples/case-disposition) |
| **CaseSecurity** | [examples/case-security](./examples/case-security) |
| **DocumentSecurity** | [examples/document-security](./examples/document-security) |
| **CaseDelete** | [examples/case-delete](./examples/case-delete) |
| **CaseExpunge** | [examples/case-expunge](./examples/case-expunge) |
| **DocumentFiling** | [examples/document-filing](./examples/document-filing) |
| **Generic Example** | [examples/generic-example](./examples/generic-example) |

Each folder may contain one or more example XML messages relevant to that specific event type.


---

## Related API Pages

| API | Path |
|------|------|
| **RecordFiling** | [../recordfiling/README.md](../recordfiling/README.md) |
| **NotifyDocketingComplete** | [../notifydocketingcomplete/README.md](../notifydocketingcomplete/README.md) |
| **GetCase** | [../getcase/README.md](../getcase/README.md) |
| **GetDocument** | [../getdocument/README.md](../getdocument/README.md) |

---

_Back to API Reference Index:_  
➡️ [../README.md](../README.md)
