# RecordFiling Behavior Guide -- TBD

This guide describes how **CMS systems** and **re:Search** must handle RecordFiling, including sequencing rules, validation requirements, and common integration issues.

---

## CMS Responsibilities After Receiving RecordFiling

The CMS must:

1. **Validate Case Context**
   - Ensure the case exists (for subsequent filings)
   - Validate `CourtIdentifier`
   - Confirm filing type/subtype is supported

2. **Validate Documents**
   - Supported formats (PDF required in most markets)
   - Check for missing or corrupt documents

3. **Docket the Filing**
   - Create the docket entry
   - Assign CMS Filing ID
   - Assign Document IDs
   - Apply security rules

4. **Return NotifyDocketingComplete**
   - Must include all CMS-generated IDs
   - Must reference the original FilingID
   - Must match CourtIdentifier and CaseTrackingID

5. **Ensure Data Appears in GetCase**
   - Filing must be present in the `<Filing>` block
   - DocketEntry must be present in `<DocketEntry>`
   - Documents must be discoverable in `<Document>` metadata

---

## How re:Search Interprets RecordFiling

re:Search **never consumes RecordFiling directly**.

Instead it relies on:

1. NotifyDocketingComplete  
2. Subsequent GetCase calls  

If GetCase does not return:
- Filing metadata  
- Docket entries  
- Document metadata  

Then re:Search will not display the filing.

---

## Common CMS Integration Failures

| Failure | Impact |
|---------|--------|
| Filing accepted but not added to GetCase | Filing never appears in re:Search |
| Document IDs missing in GetCase | Documents cannot be retrieved |
| Wrong CaseTrackingID in NotifyDocketingComplete | Re:Search fails to correlate updates |
| Missing FilingType or subtype | Filing rejected |
| CourtIdentifier mismatch | Routing error / filing rejected |

---

## Required Mapping Rules

- `FilingID` from RecordFiling must map to the CMS’s internal filing record.
- CMS must map **every document** in RecordDocketingMessage to a CMS DocumentID.
- CMS must return the final authoritative state of the case in GetCase after docketing.

---

_Back to RecordFiling Overview:_  
➡️ [README.md](./README.md)
