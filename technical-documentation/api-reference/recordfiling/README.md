# RecordFiling API

The **RecordFiling** operation is used by the **Electronic Filing Manager (EFM)** to deliver one or more accepted filings to the court **CMS** for docketing.  
This message officially transfers the filing package (metadata + documents) into the CMS, initiating the workflow that ensures filings, docket entries, and documents appear in re:Search.

RecordFiling contains two coordinated payloads:

### CoreFilingMessage
Captures the contextual/legal metadata of the filing:
- Parties and attorneys  
- Filing metadata (type, subtype, description)  
- Fees and payment  
- Case header information  
- Case initiation vs. subsequent indicators  

### RecordDocketingMessage
Represents the docketable content the CMS needs to process the filing:
- Documents (lead, attachments, exhibits)  
- Clerk reviewer info  
- Docket date, notes, and submission attributes  
- Redaction mode  

Together, these payloads allow the CMS to docket filings accurately and return identifiers later through NotifyDocketingComplete.

---

## Purpose

RecordFiling enables the EFM to:
- Deliver filing metadata and documents to the CMS  
- Initiate the docketing workflow  
- Align filings to ECF 4.x message standards  
- Establish IDs consumed by NotifyDocketingComplete and GetCase  

RecordFiling is required for all **ECF Mode** environments.

---

## Transport and Protocol

| Property | Value |
|---------|--------|
| **Direction** | EFM → CMS |
| **Protocol** | SOAP 1.2 over HTTPS |
| **Security** | Mutual TLS (mTLS) |
| **Standard** | OASIS ECF 4.x |
| **Message Type** | RecordFilingMessage |

Header requirements:  
➡️ [Common Headers & Auth](../common-headers-and-auth.md)

---

## When RecordFiling Occurs

RecordFiling is sent after:
1. EFSP submits a filing  
2. EFM validates the envelope (fees, metadata, documents, participants)  
3. Filing is accepted and ready for CMS docketing  

Upon receiving RecordFiling, the CMS must:
- Docket the filing  
- Assign CMS identifiers (e.g., DocketEntryID, CMS Filing ID, Document IDs)  
- Return them through `NotifyDocketingComplete`  
- Ensure the updates appear in `GetCase`  

---

## High-Level Workflow

diagram placeholder

### Steps

1. **EFSP Submits Filing** → EFM  
2. **EFM Validates Envelope**  
3. **EFM Sends RecordFiling to CMS**  
4. **CMS Dockets Filing**  
5. **CMS Sends NotifyDocketingComplete**  
6. **EFM Notifies re:Search**  
7. **re:Search Calls GetCase** (authoritative refresh)  

---

## Required Message Elements

| Element | Description | Required |
|--------|-------------|----------|
| `Filing` | Root container | Yes |
| `FilingID` | EFM filing identifier | Yes |
| `CaseTrackingID` | CMS case ID | Yes (for subsequent filings) |
| `CourtIdentifier` | Routing/court code | Yes |
| `FilingType` | Filing category | Yes |
| `Documents` | Filing documents | Yes |
| `FilingAttorney` / `SubmittingAttorney` | Attorney context | Required for attorney-driven filings |
| `Payment` | Fees and payment info | Required for fee-bearing filings |

---

## How re:Search Uses RecordFiling

**re:Search does not consume RecordFiling directly.**

Instead, it depends on:

1. **CMS docketing** (processing the RecordFiling)  
2. **CMS returning IDs** via NotifyDocketingComplete  
3. **re:Search pulling the authoritative record** via GetCase  

If the CMS does not reflect the filing in GetCase, it **will not appear** in re:Search — even if RecordFiling was accepted.

---

## Related API Pages

| API | Path |
|------|------|
| **NotifyDocketingComplete** | [../notifydocketingcomplete/README.md](../notifydocketingcomplete/README.md) |
| **GetCase** | [../getcase/README.md](../getcase/README.md) |
| **NotifyCaseEvent** | [../notifycaseevent/README.md](../notifycaseevent/README.md) |
| **GetDocument** | [../getdocument/README.md](../getdocument/README.md) |

---

## Examples

See example XML files here:


| Example | Link |
|---------|------|
| **RecordFiling – Annotated Request** | [examples/recordfiling-request-annotated.xml](./examples/recordfiling-request-annotated.xml) |
| **RecordFiling – Annotated Response (MessageReceipt)** | [examples/recordfiling-response-receipt-annotated.xml](./examples/recordfiling-response-receipt-annotated.xml) |

---

_Back to API Reference Index:_  
➡️ [../README.md](../README.md)