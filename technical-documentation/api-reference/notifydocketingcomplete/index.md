# NotifyDocketingComplete – re:Search Integration API

Version: 3.4.x  
Audience: Technical  
Owner: BIS – Bryce Beltran  
Last Updated: 2025-11-12  

`NotifyDocketingComplete` is the message the **CMS sends to the EFM** after a filing has been successfully docketed.  
This message confirms acceptance of the filing, provides CMS-assigned identifiers, and signals that the CMS has completed its internal processing.

It is a critical part of the ECF workflow because it ensures that:

- The EFM can finalize the filing,
- The EFSP can notify the filer,
- re:Search can synchronize the case using a subsequent `GetCase` call.

---

## Purpose

NotifyDocketingComplete allows the CMS to:

- Confirm the court has accepted and docketed a filing  
- Return CMS-assigned document IDs, filing IDs, and docket entry identifiers  
- Indicate docketing details including date, time, and clerk reviewer  
- Complete the ECF filing lifecycle initiated by RecordFiling  

This message also indirectly triggers re:Search synchronization because the EFM sends an ECF event to re:Search after receiving this confirmation.

---

## Transport and Protocol

| Property | Value |
|----------|-------|
| Direction | **CMS → EFM** |
| Protocol | SOAP 1.2 over HTTPS |
| Security | Mutual TLS (mTLS) |
| Schema | OASIS ECF 4.x – `NotifyDocketingCompleteMessage` |
| Triggered By | Successful docketing of a filing |

Headers follow ECF standard SOAP envelope extensions.  
See: `../common-headers-and-auth.md`

---

## When NotifyDocketingComplete Occurs

NotifyDocketingComplete is always sent **after a RecordFiling request** has been processed by the CMS and the filing is officially docketed.  
Common triggers:

- Clerk approves the filing  
- Automatic docketing occurs (auto-accept sites)  
- CMS processes envelope and attaches documents to the case  
- CMS generates filing and document IDs  

If a CMS does not send this message, the EFM cannot complete the transaction and re:Search may not receive accurate filing metadata.

---

## High-Level Workflow

Diagram here

### Steps

1. **EFM Sends RecordFiling**  
   CMS receives filing payload and extracts documents, metadata, parties, and associations.

2. **CMS Dockets Filing**  
   The CMS creates:
   - CMS Filing ID  
   - Docket Entry ID  
   - CMS Document IDs for each attachment  

3. **CMS Sends NotifyDocketingComplete**  
   Returned data includes:
   - Envelope ID  
   - Filing IDs and document IDs  
   - Docket entry information  
   - Timestamp and reviewed‐by data  

4. **EFM Acknowledges and Finalizes**  
   EFSP receives confirmation and updates its transaction history.

5. **EFM Sends Event to re:Search**  
   re:Search calls `GetCase` to retrieve the updated case data.

---

## Required Message Elements

### Request (CMS → EFM)

| Element | Description | Required |
|---------|-------------|----------|
| `EnvelopeID` | ID of the original filing envelope | Yes |
| `FilingID` | CMS-assigned filing identifier | Yes |
| `DocumentIdentifiers` | All CMS document IDs created during docketing | Yes |
| `DocketEntryID` | Unique docket entry identifier | Recommended but expected |
| `ReviewedBy` | Clerk or auto-accept processor ID | Recommended |
| `DocketDate` | When docketing occurred | Yes |
| `CaseTrackingID` | CMS case number used for consistency | Yes |
| `CourtIdentifier` | Routing identifier | Yes |

### Response (EFM → CMS)

| Element | Description |
|---------|-------------|
| `MessageReceipt` | Standard ECF acknowledgment |
| `Status` | Success or error context |
| `Timestamp` | When receipt processed |

---

## Relationship to Other APIs

NotifyDocketingComplete works together with:

- **RecordFiling** – submitted from EFM → CMS to initiate docketing  
- **GetCase** – used after docketing to retrieve updated metadata  
- **NotifyCaseEvent** – may also be sent when CMS performs non-filing updates  
- **GetDocument** – re:Search uses this to retrieve actual documents referenced in docketing  

This message finalizes the e-filing transaction and ensures data flows downstream consistently.

---

## Validation Requirements

### CMS Must Ensure:

- All returned DocumentIDs match those referenced in GetCase  
- DocumentIDs are unique and stable  
- CMS dates and timestamps are accurate  
- EnvelopeID matches the RecordFiling request  
- CaseTrackingID matches the docketed case  

### EFM Will Reject:

- Missing or duplicate DocumentIDs  
- Mismatched EnvelopeID  
- Invalid XML schema  
- Missing required filing identifiers  

---

## Common Vendor Mistakes

| Mistake | Result |
|---------|--------|
| Not returning all document IDs | re:Search and EFSP show missing documents |
| Returning different IDs in GetCase | Inconsistent indexing and broken UI references |
| Using temporary or session‐based document identifiers | Future GetDocument calls fail |
| Incorrect envelope or filing ID mapping | EFM rejects the transaction |
| Missing docket date | EFM may mark the filing as incomplete |

---

## Examples

XML examples can be found here:

technical/api-reference/notify-docketing-complete/examples/
---

## Example URLs (Developer Quick Links)

| Example | URL |
|---------|-----|
| Successful Docketing | `/technical/api-reference/notify-docketing-complete/examples/success.xml` |
| Missing Document ID Fault | `/technical/api-reference/notify-docketing-complete/examples/missing-doc-id.xml` |
| Envelope Mismatch | `/technical/api-reference/notify-docketing-complete/examples/envelope-mismatch.xml` |
| Docket Failure | `/technical/api-reference/notify-docketing-complete/examples/docket-failure.xml` |

(Placeholders until populated.)

---

## Related API Reference Pages

| API | Path | Purpose |
|------|------|---------|
| **RecordFiling** | [../record-filing/index.md](../record-filing/index.md) | Submits filings to CMS for docketing |
| **GetCase** | [../get-case/index.md](../get-case/index.md) | Retrieves updated case metadata post-docketing |
| **NotifyCaseEvent** | [../notify-case-event/index.md](../notify-case-event/index.md) | CMS-initiated update events for all non-filing changes |
| **GetDocument** | [../get-document/index.md](../get-document/index.md) | Retrieves document binaries |

---