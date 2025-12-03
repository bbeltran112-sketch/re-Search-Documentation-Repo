# GetCase Behavior Guide – re:Search Integration

This guide defines how **re:Search** interprets, indexes, and displays the data returned in the `GetCaseResponse` from a CMS.  
Because GetCase drives 100% of data shown in the re:Search UI, correct behavior is mandatory for all CMS vendors and Tyler engineers.

This guide expands on the core API specification found in `README.md` and documents the deeper rules and expectations that determine how re:Search behaves.

---

# 1. How re:Search interprets GetCaseResponse

re:Search uses the GetCaseResponse as the authoritative representation of a case.  
The UI renders only what the CMS returns — missing or malformed data results in missing or incorrect UI behavior.

| XML Element | Used For | Notes |
|-------------|----------|-------|
| `ecf:CaseTrackingID` | Primary case reference | Required for all updates |
| `ecf:CaseCategoryText` | Case type filtering | Expected |
| `ecf:CaseTypeText` | Subcategory/type classification | Expected |
| `ecf:CaseSubTypeText` | Deep filtering and grouping | New / expected |
| `tyler:CaseSecurity` | Case-level visibility | Required |
| `tyler:FilingEvent` | Filing and docket timeline | Required when eventType=CaseFiling |
| `ecf:ActivityDate/nc:DateTime` | Filing/event timestamp | **Required for re:Search timeline indexing** |
| `ecf:DocumentMetadata` | Filing-level detail | Expected |
| `ecf:DocumentRendition` | Document metadata | Expected |
| `tyler:DocumentAttachmentIdentification` | Document → CMSID mapping | Required |
| `tyler:PageCount` | Document page count | **Required for re:Search document indexing and UI** |

If an element is not present in the response, re:Search cannot infer or “guess” missing values.

---

# 2. Indexing rules

The re:Search indexer constructs the case model as follows:

### Case header

- Rendered exactly as provided  
- Missing category/type/subtype reduces search and filtering quality  
- Subtype improves filtering but is not required

### Filing events

- **Sorted by `ecf:ActivityDate/nc:DateTime` (required)**
- Events without ActivityDate **cannot be reliably ordered** and may cause incorrect or unstable timelines
- Multiple documents grouped under the same filing event  
- `ActivityStatus` enhances docket state messages but does not replace ActivityDate

### Documents

- Grouped under FilingEvent → DocumentRendition → Attachment  
- Ordering determined by appearance and BinaryCategoryText (LEAD first)  
- `nc:IdentificationID` (CMSID) must be present for every document  
- **`tyler:PageCount` is required for re:Search’s document indexing and UI display** (page count controls how the document is represented and validated)

### Parties & attorneys

- Sorted by role → order of appearance  
- Missing IDs reduce linkage between parties and filings

### Security

- Most restrictive rule wins  
- CaseSecurity overrides DocumentSecurity whenever there is a conflict

---

# 3. EventType mapping

This is the most critical re:Search rule:

> **re:Search only processes XML blocks in GetCaseResponse that match the EventType sent in NotifyCaseEvent.**

If the CMS includes unrelated changes in the GetCaseResponse, re:Search **ignores** those updates.

| EventType | Blocks Processed | Blocks Ignored |
|-----------|------------------|----------------|
| CaseFiling | FilingEvent, DocumentMetadata, DocumentRendition | Parties, CaseSecurity |
| CaseParty | Parties, PartyAttorney | Filings/dockets |
| CaseSecurity | CaseSecurity only | Everything else |
| CaseReassigned | Court/judge header | All others |
| CaseDisposition | Disposition block | All others |
| CaseDelete | Delete/restore flags | All others |

If the CMS mixes fields from different categories into a single payload:  
They will not update unless the EventType matches.

---

# 4. Document security behavior

### Case-level (`tyler:CaseSecurity`)

Controls entire-case visibility. Examples include:

- `PublicFilingPublicView`  
- `PublicFilingRestrictedView`  
- `SealedCase`  
- `ExpungedCase`  

### Document-level (`tyler:DocumentSecurity`)

Controls individual document visibility:

- `PublicView`  
- `RestrictedView`  
- `NoAccess`  

### Conflict resolution

| CaseSecurity | DocumentSecurity | Effective Result |
|--------------|------------------|------------------|
| Sealed | Any | Entire case hidden |
| PublicFilingPublicView | PublicView | Document visible |
| PublicFilingPublicView | RestrictedView | Visible with limited metadata |
| PublicFilingPublicView | NoAccess | Document hidden |

---

# 5. Document retrieval mapping

re:Search requires:

| Required Element | Purpose |
|------------------|---------|
| `nc:IdentificationID` (CMSID) | Passed into GetDocument |
| `BinaryLocationURI` | Optional direct link/handle |
| `structures:id` + `s:ref` | Cross-reference integrity |
| `tyler:PageCount` | Required for re:Search document indexing & UI validation |

If CMSID is missing:
- Document cannot be retrieved  
- Downloads fail  
- Case appears incomplete  

If `PageCount` is missing:
- Document indexing and UI representation may be incomplete or incorrect  
- Some internal validation behaviors may fail  

---

# 6. Filing timeline behavior

Each filing event must include:

- **A valid ISO8601 `ActivityDate` (required)**
- A meaningful `RegisterActionDescriptionText`
- Optionally an `ActivityStatus` block

If timestamps are missing or invalid:

- Filing order becomes unpredictable  
- “Unknown date” or inconsistent ordering may appear in UI  
- Timeline groupings may break

`ActivityDate` is a **required field for re:Search** and must be present and valid for every filing event that should appear in the timeline.

---

# 7. Error & fault handling

### SOAP faults

If the case does not exist:

- CMS must return a **SOAP Fault**  
- Never return an empty `<ecf:Case>` block

### Invalid XML

If content is malformed:

- re:Search logs an error  
- The case is not updated  
- Vendor must correct their payload

### Missing required fields

Impacts include:

- Missing filings  
- Missing documents  
- Incorrect or missing events  
- Broken document links  
- Incorrect security behavior  

---

# 8. What re:Search ignores

To avoid incorrect expectations, be aware that re:Search ignores:

- Partial case responses  
- Blocks unrelated to EventType  
- Non-standard custom XML extensions  
- Lowercase or invalid enumeration values  
- Embedded binary content  
- Missing or non-ISO timestamps  
- Orphaned `s:ref` / `structures:id` mappings  

If it doesn’t match schema + behavior rules, it is ignored.

---

# 9. Annotated examples

The examples folder contains:

- `getcase-response-full.xml`
- `getcase-response-filing-update.xml`
- `getcase-response-party-update.xml`
- `getcase-response-sealed.xml`
- `getcase-response-document-security.xml`

These illustrate:

- Correct EventType mapping  
- Proper `ActivityDate` usage  
- Correct `PageCount` handling  
- Document attachment cross-references  
- Valid CaseSecurity and DocumentSecurity values  
- Required CMSID fields  

---

# 10. Validation rules

### Required for re:Search to index properly

| Rule | Explanation |
|------|-------------|
| `tyler:CaseSecurity` present | Determines case-level visibility |
| `ecf:ActivityDate/nc:DateTime` present and valid | **Required for timeline and event ordering** |
| `tyler:PageCount` present for each document | **Required for re:Search document indexing & UI** |
| CMSID present on all documents (`nc:IdentificationID`) | Required for retrieval (GetDocument) |
| ISO8601 timestamps (UTC) | Sorting, caching, and history |
| Full case payload returned | re:Search does not accept partial updates |
| Correct EventType | Determines which blocks get parsed |
| `structures:id` and `s:ref` resolve | Links attachments to metadata |

### Expected

- CaseCategory, CaseType, CaseSubType  
- Party blocks  
- ActivityStatus  
- Document description and size  
- BinaryCategoryText (LEAD/ATTACHMENT/EXHIBIT)  

### Forbidden

- Embedded binaries  
- Empty `<Case>` blocks  
- Lowercase enum values for security  
- Custom XML outside permitted namespaces  

---

# 11. Known vendor pitfalls

Common causes of failed updates:

| Pitfall | Result |
|---------|--------|
| Wrong EventType | re:Search ignores key data |
| Missing ActivityDate | Filings appear out of order or are unreliable in the timeline |
| Missing PageCount | Document handling and indexing can fail or be incomplete |
| Missing CMSID | Documents cannot load |
| Missing CaseSecurity | Case may be exposed or hidden incorrectly |
| Partial case payloads | UI only partially updates |
| Incorrect `s:ref` | Attachments appear unlinked |
| Lowercase enums | Security rules ignored |

---

# 12. Certification checklist

Before a vendor can be approved:

✔ All EventTypes tested  
✔ `ActivityDate` present and valid for all events  
✔ `PageCount` present for all document attachments  
✔ All required fields present  
✔ All timestamps valid ISO8601 UTC  
✔ All document CMSIDs stable and persistent  
✔ CaseSecurity correct and consistent  
✔ No embedded binaries  
✔ `structures:id` and `s:ref` resolve correctly  
✔ FilingEvents sort correctly in UI  
✔ Full payload returned every time  

---

# 13. UI mapping table

| UI Section | XML Element |
|------------|-------------|
| Case Title | `ecf:CaseTitleText` |
| Case Category/Type | `ecf:CaseCategoryText` + `ecf:CaseTypeText` |
| Subtype | `ecf:CaseSubTypeText` |
| Parties | Party blocks |
| Filings Timeline | FilingEvent + `ecf:ActivityDate` |
| Filing Description | `j:RegisterActionDescriptionText` |
| Document List | DocumentRendition / DocumentAttachment |
| Document Security | `tyler:DocumentSecurity` |
| Case Security | `tyler:CaseSecurity` |
| Judge / Court | Header (via CaseReassigned) |
| Timeline Sorting | `ecf:ActivityDate/nc:DateTime` |

---

# Summary

The GetCaseResponse defines the entire case experience in re:Search.

For correct behavior, CMS vendors must:

- Use the correct EventType and associated XML blocks  
- Always include a complete case payload  
- Provide valid, ISO8601 `ActivityDate` values for all events  
- Provide `PageCount` for all documents  
- Use exact security enumerations  
- Supply stable, resolvable CMSIDs and cross-references  

This guide is mandatory for any CMS vendor onboarding to re:Search and should be used alongside the main GetCase API README.
