# GetCase Behavior Guide

[← Back to GetCase Overview](./README.md)

This guide defines how **re:Search interprets, indexes, and displays** the data returned in GetCaseResponse. Understanding these rules is critical for correct CMS integration.

## Quick Navigation
- [How re:Search Interprets Responses](#how-research-interprets-responses)
- [EventType Mapping](#eventtype-mapping)
- [Indexing Rules](#indexing-rules)
- [Security Behavior](#security-behavior)
- [Document Handling](#document-handling)
- [Timeline and Sorting](#timeline-and-sorting)
- [What re:Search Ignores](#what-research-ignores)
- [Known Vendor Pitfalls](#known-vendor-pitfalls)
- [Certification Checklist](#certification-checklist)

---

## Core Principle

> **re:Search uses GetCaseResponse as the single source of truth for case data.**

If something is missing or incorrect in GetCaseResponse, it will be missing or incorrect in re:Search. There is no inference, no guessing, no fallback data.

---

## How re:Search Interprets Responses

### XML Element Usage Table

| XML Element | Used For | Importance |
|-------------|----------|------------|
| `ecf:CaseTrackingID` | Primary case reference | **Critical** - required for all updates |
| `tyler:CaseSecurity` | Case-level visibility | **Critical** - determines who can see case |
| `ecf:ActivityDate/nc:DateTime` | Filing/event timestamp | **Critical** - required for timeline |
| `tyler:PageCount` | Document page count | **Critical** - required for document indexing |
| `nc:IdentificationID` (CMSID) | Document identifier | **Critical** - required for GetDocument |
| `ecf:CaseCategoryText` | Case type filtering | **Required** |
| `ecf:CaseTypeText` | Subcategory classification | **Required** |
| `ecf:CaseSubTypeText` | Deep filtering/grouping | Recommended |
| `tyler:FilingEvent` | Filing and docket timeline | **Required** for CaseFiling |
| `ecf:DocumentMetadata` | Filing-level detail | **Required** for documents |
| `ecf:DocumentRendition` | Document metadata | **Required** for documents |
| `tyler:DocumentSecurity` | Document-level visibility | **Required** for document access |

### Missing Data = Missing Features

If an element is not present in GetCaseResponse, re:Search cannot infer or "guess" missing values:

- Missing `ActivityDate` → Filings cannot be sorted reliably
- Missing `CMSID` → Documents cannot be retrieved
- Missing `PageCount` → Document indexing incomplete
- Missing `CaseSecurity` → Incorrect visibility
- Missing party data → No parties shown

---

## EventType Mapping

### The Critical Rule

> **re:Search only processes XML blocks in GetCaseResponse that match the EventType sent in NotifyCaseEvent.**

This is the **most important behavior rule** in the entire integration.

### How It Works

**Example 1: CaseFiling Event**
```
CMS sends: NotifyCaseEvent(CaseTrackingID="CV-123", EventType="CaseFiling")
           ↓
re:Search calls: GetCase(CaseTrackingID="CV-123")
           ↓
re:Search processes: ONLY FilingEvent, DocumentMetadata, DocumentRendition
re:Search ignores: Parties, CaseSecurity, CaseDisposition, etc.
```

**Example 2: CaseParty Event**
```
CMS sends: NotifyCaseEvent(CaseTrackingID="CV-123", EventType="CaseParty")
           ↓
re:Search calls: GetCase(CaseTrackingID="CV-123")
           ↓
re:Search processes: ONLY Party and Attorney blocks
re:Search ignores: Filings, Documents, Security, etc.
```

### EventType Processing Table

| EventType | Blocks Processed | Blocks Ignored | Notes |
|-----------|------------------|----------------|-------|
| `CaseFiling` | FilingEvent, DocumentMetadata, DocumentRendition | Parties, CaseSecurity, CaseDisposition | Most common event |
| `CaseParty` | CaseParticipant, CaseAttorney | Filings, Documents, Security | Party/attorney changes only |
| `CaseSecurity` | tyler:CaseSecurity only | Everything else | **Critical** - immediate processing |
| `DocumentSecurity` | tyler:DocumentSecurity only | Everything else | Individual document security |
| `CaseDisposition` | CaseDisposition block | All others | Disposition entered |
| `CaseReassigned` | Court/judge header | All others | Judge or division change |
| `CaseStatus` | CaseStatus block | All others | Status change |
| `CaseDelete` | Delete/restore flags | All others | Case deletion/restoration |
| `CaseInitiated` | Complete case (initial load) | None | New case - processes everything |

### Common Mistake

**❌ Wrong:**
```xml
<!-- CMS sends CaseParty event -->
<NotifyCaseEvent>
  <EventType>CaseParty</EventType>
</NotifyCaseEvent>

<!-- But GetCase response includes filing changes -->
<GetCaseResponse>
  <Case>
    <Parties><!-- Party changes here --></Parties>
    <FilingEvent><!-- Filing changes here --></FilingEvent> ← IGNORED!
  </Case>
</GetCaseResponse>
```

**✅ Correct:**
```xml
<!-- If BOTH party AND filing changed, send TWO events -->
<NotifyCaseEvent><EventType>CaseParty</EventType></NotifyCaseEvent>
<NotifyCaseEvent><EventType>CaseFiling</EventType></NotifyCaseEvent>

<!-- Or send CaseInitiated to process everything -->
<NotifyCaseEvent><EventType>CaseInitiated</EventType></NotifyCaseEvent>
```

---

## Indexing Rules

### Case Header

- Rendered exactly as provided in GetCaseResponse
- Missing category/type/subtype reduces search quality
- Subtype improves filtering but not required
- Case title displayed as-is (no transformation)

### Filing Events

**Sort Order:**
- **Primary sort:** `ecf:ActivityDate/nc:DateTime` (**required**)
- **Secondary sort:** Appearance order in XML

**Critical:** Events without `ActivityDate` **cannot be reliably ordered** and may cause:
- Incorrect timeline display
- Unstable sorting
- "Unknown date" messages
- Filing appears in wrong position

**Timeline Grouping:**
- Multiple documents grouped under same FilingEvent
- `ActivityStatus` enhances display but does NOT replace ActivityDate
- Filings rendered chronologically

### Documents

**Document Hierarchy:**
```
FilingEvent (filing-level)
  └─ DocumentMetadata (filing info)
       └─ DocumentRendition (rendition info)
            └─ DocumentAttachment (actual document)
```

**Document Requirements:**
- `nc:IdentificationID` (CMSID) **required** for retrieval
- `tyler:PageCount` **required** for indexing and UI
- `structures:id` and `s:ref` must resolve correctly
- `BinaryCategoryText` determines display order (LEAD first)

**Display Order:**
1. LEAD documents (primary filings)
2. ATTACHMENT documents (supporting docs)
3. EXHIBIT documents (exhibits)

### Parties & Attorneys

- Sorted by role → appearance order
- Missing IDs reduce linkage between parties and filings
- Attorney associations via `RepresentsParty`
- Contact information displayed if provided

---

## Security Behavior

### Security Levels

**Case-Level Security (`tyler:CaseSecurity`):**

| Value | Meaning | Who Can See |
|-------|---------|-------------|
| `PublicFilingPublicView` | Public case, public filings | Everyone |
| `PublicFilingRestrictedView` | Public case, some restricted filings | Everyone sees case, limited filings |
| `SealedCase` | Entire case sealed | Court staff only (registered users) |
| `ExpungedCase` | Case expunged/destroyed | Very limited access |

**Document-Level Security (`tyler:DocumentSecurity`):**

| Value | Meaning | Who Can See |
|-------|---------|-------------|
| `PublicView` | Public document | Everyone |
| `RestrictedView` | Restricted document | Limited metadata shown, content hidden |
| `NoAccess` | No public access | Document completely hidden from public |

### Conflict Resolution

**Most restrictive rule wins:**

| CaseSecurity | DocumentSecurity | Effective Result |
|--------------|------------------|------------------|
| `SealedCase` | Any value | **Entire case hidden** (case security wins) |
| `PublicFilingPublicView` | `PublicView` | Document visible to everyone |
| `PublicFilingPublicView` | `RestrictedView` | Case visible, document metadata limited |
| `PublicFilingPublicView` | `NoAccess` | Case visible, document completely hidden |
| `PublicFilingRestrictedView` | `PublicView` | Case visible, document visible |
| `PublicFilingRestrictedView` | `NoAccess` | Case visible, document hidden |

### Security Event Priority

**CaseSecurity and DocumentSecurity events are processed immediately:**
- Never batched or delayed
- Processed before other event types
- Changes take effect within seconds

---

## Document Handling

### Document Retrieval Requirements

For documents to be retrievable via GetDocument:

| Required Element | Purpose | Impact if Missing |
|------------------|---------|-------------------|
| `nc:IdentificationID` (CMSID) | Passed to GetDocument | **Cannot retrieve document** |
| `tyler:PageCount` | Document size validation | **Indexing incomplete, UI issues** |
| `structures:id` + `s:ref` | Cross-reference integrity | **Document metadata disconnected** |
| `BinaryLocationURI` | Optional direct link | Retrieval optimization (optional) |

### Document Cross-Reference Pattern

```xml
<!-- DocumentMetadata with structures:id -->
<ecf:DocumentMetadata s:id="DOC123">
  <nc:IdentificationID>CMSID-789</nc:IdentificationID>
</ecf:DocumentMetadata>

<!-- DocumentRendition references it with s:ref -->
<ecf:DocumentRendition>
  <ecf:DocumentMetadata s:ref="DOC123"/>
  <ecf:DocumentAttachment>
    <tyler:DocumentAttachmentIdentification>
      <nc:IdentificationID>CMSID-789</nc:IdentificationID> ← Must match!
    </tyler:DocumentAttachmentIdentification>
    <tyler:PageCount>15</tyler:PageCount> ← Required!
  </ecf:DocumentAttachment>
</ecf:DocumentRendition>
```

**If cross-references don't resolve:**
- Documents appear orphaned
- Download links broken
- Case appears incomplete

---

## Timeline and Sorting

### Filing Timeline Requirements

**Every filing event must include:**

1. **Valid `ecf:ActivityDate/nc:DateTime`** (REQUIRED)
   - ISO 8601 format
   - UTC timezone recommended
   - Format: `YYYY-MM-DDTHH:MM:SSZ`

2. **Meaningful `RegisterActionDescriptionText`** (REQUIRED)
   - Human-readable filing description
   - Example: "Complaint Filed", "Motion to Dismiss"

3. **Optional `ActivityStatus` block**
   - Enhances display with status details
   - Does NOT replace ActivityDate

### Impact of Missing Timestamps

If `ActivityDate` is missing or invalid:

- ❌ Filing order becomes unpredictable
- ❌ "Unknown date" or "Invalid date" in UI
- ❌ Timeline groupings break
- ❌ Chronological navigation fails
- ❌ Search by date broken

**ActivityDate is MANDATORY for every filing event.**

### Sorting Logic

```
Primary Sort: ActivityDate (oldest to newest)
Secondary Sort: Appearance order in XML
Tertiary Sort: Document category (LEAD → ATTACHMENT → EXHIBIT)
```

---

## What re:Search Ignores

To avoid incorrect expectations, re:Search ignores:

| Ignored Item | Reason |
|--------------|--------|
| **Partial case responses** | Must be complete for EventType |
| **Blocks unrelated to EventType** | See EventType Mapping |
| **Non-standard custom XML extensions** | Outside ECF schema |
| **Lowercase or invalid enumeration values** | Must be exact case |
| **Embedded binary content** | Use GetDocument instead |
| **Missing or non-ISO timestamps** | Cannot parse/sort |
| **Orphaned `s:ref` / `structures:id` mappings** | Cannot resolve |
| **Cached or stale data** | Must be current |
| **Empty `<Case>` blocks** | Return SOAP Fault instead |

**If it doesn't match schema + behavior rules, it is ignored.**

---

## Known Vendor Pitfalls

Common causes of failed updates:

| Pitfall | Result | Solution |
|---------|--------|----------|
| **Wrong EventType used** | re:Search ignores key data | Use correct EventType or CaseInitiated |
| **Missing ActivityDate** | Filings appear out of order or missing | Always include valid ActivityDate |
| **Missing PageCount** | Document indexing fails, UI issues | Include PageCount for all documents |
| **Missing CMSID** | Documents cannot be retrieved | Include stable CMSID for all documents |
| **Missing CaseSecurity** | Case visibility incorrect | Always include CaseSecurity |
| **Partial case payloads** | UI only partially updates | Return complete case data |
| **Incorrect `s:ref` mappings** | Documents appear unlinked | Verify structures:id and s:ref match |
| **Lowercase enum values** | Security rules ignored | Use exact case (e.g., "PublicView") |
| **Cached responses** | Changes don't appear | Always return current data |
| **Empty `<Case>` blocks** | Confuses indexer | Return SOAP Fault if case not found |
| **Embedded binaries** | Response too large, fails | Use GetDocument for binaries |

---

## UI Mapping Table

Understanding what GetCase fields map to in the re:Search UI:

| UI Section | XML Element | Notes |
|------------|-------------|-------|
| **Case Title** | `ecf:CaseTitleText` | Displayed as-is |
| **Case Number** | `ecf:CaseTrackingID` | Primary identifier |
| **Case Category/Type** | `ecf:CaseCategoryText` + `ecf:CaseTypeText` | Used for filtering |
| **Case Subtype** | `ecf:CaseSubTypeText` | Deep filtering |
| **Case Security Badge** | `tyler:CaseSecurity` | "Public", "Sealed", etc. |
| **Parties List** | `ecf:CaseParticipant` blocks | Grouped by role |
| **Attorneys List** | `ecf:CaseAttorney` blocks | With party associations |
| **Filings Timeline** | `tyler:FilingEvent` + `ecf:ActivityDate` | Chronological order |
| **Filing Description** | `j:RegisterActionDescriptionText` | Timeline entry title |
| **Filing Date** | `ecf:ActivityDate/nc:DateTime` | Timeline sorting |
| **Document List** | `ecf:DocumentRendition` / `ecf:DocumentAttachment` | Per filing |
| **Document Title** | `nc:DocumentDescriptionText` | Document name |
| **Document Security** | `tyler:DocumentSecurity` | "Public", "Restricted", etc. |
| **Document Pages** | `tyler:PageCount` | "15 pages" |
| **Judge Name** | Court header (via CaseReassigned) | Case header |
| **Download Link** | `nc:IdentificationID` (CMSID) | Passed to GetDocument |

---

## Certification Checklist

Before a CMS can be certified for production:

### Data Completeness
- [ ] All EventTypes tested and working
- [ ] All required fields present in responses
- [ ] Complete case payload returned (not partial)
- [ ] Parties, attorneys, and filings all included

### Timestamps and Dates
- [ ] `ActivityDate` present and valid for ALL filing events
- [ ] All timestamps valid ISO 8601 format
- [ ] All timestamps in UTC (or with explicit timezone)
- [ ] No missing or "null" dates

### Document Handling
- [ ] `PageCount` present for ALL document attachments
- [ ] All document CMSIDs stable and persistent
- [ ] `structures:id` and `s:ref` resolve correctly
- [ ] No embedded binaries in response
- [ ] BinaryCategoryText set correctly (LEAD/ATTACHMENT/EXHIBIT)

### Security
- [ ] `CaseSecurity` present and correct for all cases
- [ ] `DocumentSecurity` present for documents with restricted access
- [ ] Security enum values use exact case (not lowercase)
- [ ] Sealed cases return minimal data

### Performance
- [ ] Response times < 3 seconds for typical cases
- [ ] Response times < 5 seconds for complex cases
- [ ] No timeouts under normal load
- [ ] Proper error handling (SOAP Faults for missing cases)

### EventType Mapping
- [ ] CaseFiling events update filings only
- [ ] CaseParty events update parties only
- [ ] CaseSecurity events update security only
- [ ] CaseInitiated processes complete case
- [ ] No cross-contamination between EventTypes

### Error Handling
- [ ] SOAP Fault returned for missing cases (not empty `<Case>`)
- [ ] Proper HTTP status codes
- [ ] Clear error messages in faults
- [ ] No malformed XML in responses

---

## Testing Scenarios

### Must Test Successfully

**1. Filing Event Processing**
- Send CaseFiling event
- Verify GetCase called
- Verify filing appears in timeline with correct date
- Verify documents retrievable

**2. Party Update Processing**
- Send CaseParty event
- Verify GetCase called
- Verify party list updated
- Verify attorney associations correct

**3. Security Change Processing**
- Send CaseSecurity event
- Verify GetCase called immediately
- Verify case visibility changes instantly
- Verify public users cannot see sealed case

**4. Document Security Processing**
- Send DocumentSecurity event
- Verify GetCase called
- Verify individual document access restricted
- Verify sealed documents not retrievable

**5. Multiple Events**
- Send multiple events in sequence
- Verify all process correctly
- Verify no cross-contamination between EventTypes

**6. Complex Case**
- Test with 50+ parties, 100+ documents
- Verify response time < 5 seconds
- Verify all data indexed correctly
- Verify timeline sorts correctly

**7. Error Scenarios**
- Request non-existent case
- Verify SOAP Fault returned (not empty response)
- Verify appropriate error message

---

## Next Steps

- **Review request/response format** → Read [Request & Response Specification](./request-response.md)
- **See integration context** → Read [Workflows](./workflows.md)
- **Review examples** → Browse [examples directory](./examples/)
- **Return to overview** → [GetCase Overview](./README.md)

---

**Last Updated:** December 2025