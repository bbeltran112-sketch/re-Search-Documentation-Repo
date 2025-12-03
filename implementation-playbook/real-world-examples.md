# Real-World Examples

> **Breadcrumb:** [Home](../README.md) > [Implementation Playbook](./README.md) > Real-World Examples

Common integration scenarios with technical guidance. These examples are based on actual issues encountered in production integrations.

---

## Table of Contents

### Basic Integration Scenarios
- [Scenario 1: New Case Filing](#scenario-1-new-case-filing)
- [Scenario 2: Party Information Updated](#scenario-2-party-information-updated)
- [Scenario 3: Case Sealed by Court Order](#scenario-3-case-sealed-by-court-order)
- [Scenario 4: Document Filed with Mixed Security](#scenario-4-document-filed-with-mixed-security)

### Configuration & Data Issues
- [Scenario 5: Case Type Mismatch (Texas JCIT)](#scenario-5-case-type-mismatch-texas-jcit)
- [Scenario 6: Case-Sensitive Security Values](#scenario-6-case-sensitive-security-values)

### Integration Issues
- [Scenario 7: Events Not Triggering GetCase](#scenario-7-events-not-triggering-getcase)
- [Scenario 8: Complex Case with Performance Issues](#scenario-8-complex-case-with-performance-issues)

### Market-Specific Issues
- [Scenario 9: Publication Delay Not Applied (Texas)](#scenario-9-publication-delay-not-applied-texas)

### Document Retrieval Issues
- [Scenario 10: Document Not Available](#scenario-10-document-not-available)
- [Scenario 11: Documents Return Empty (0 KB Files)](#scenario-11-documents-return-empty-0-kb-files)

### Filing Issues
- [Scenario 12: Multiple Filings Appear as Single Filing](#scenario-12-multiple-filings-appear-as-single-filing)

---

## Scenario 1: New Case Filing

### The Scenario
Court files a new civil case. Case should appear in re:Search for public search.

### What Happens

**In CMS:**
```
1. User creates new case CV-2024-001234
2. Case data committed to database
3. CMS generates NotifyCaseEvent (EventType: CaseFiling)
4. Event sent to re:Search
```

**In re:Search:**
```
5. re:Search receives NotifyCaseEvent
6. re:Search calls CMS GetCase endpoint
7. CMS returns complete case data
8. re:Search indexes case
9. Case becomes searchable (subject to publication delays)
```

### Key Implementation Points

**Event Generation:**
- Send CaseFiling event after case creation
- Include case tracking ID in event
- Don't delay event transmission

**GetCase Response:**
- Return complete case metadata
- Include all parties and attorneys
- Set appropriate case security level
- Map case type to market standard code

**Common Issues:**
- Event not sent → Case never appears
- GetCase returns incomplete data → Case partially indexed
- Case type not mapped correctly → Visibility issues

[See Event Type Logic →](./event-type-logic.md)

---

## Scenario 2: Party Information Updated

### The Scenario
Attorney files appearance for defendant. Party-attorney relationship needs to be reflected in re:Search.

### What Happens

**In CMS:**
```
1. Attorney files appearance
2. CMS adds attorney to case
3. Attorney associated with defendant party
4. CMS generates NotifyCaseEvent (EventType: CasePartyAttorney)
5. Event sent to re:Search
```

**In re:Search:**
```
6. re:Search receives event
7. re:Search calls GetCase
8. GetCase returns updated party/attorney list
9. re:Search re-indexes case
10. Attorney now appears in case display
```

### Key Implementation Points

**Event Type Selection:**
- Use CasePartyAttorney for attorney changes
- Use CaseParty for party information changes
- Send event immediately after database commit

**GetCase Response:**
- Include complete attorney list
- Show which party each attorney represents
- Include attorney bar numbers
- Include lead attorney designation (if applicable)

**Common Issues:**
- Using wrong event type (CaseParty instead of CasePartyAttorney)
- Not including attorney-party association
- Missing attorney contact information

---

## Scenario 3: Case Sealed by Court Order

### The Scenario
Judge orders case sealed. Case must be immediately removed from public search.

### What Happens

**In CMS:**
```
1. Judge issues seal order
2. CMS updates case security to Sealed
3. CMS generates NotifyCaseEvent (EventType: CaseSecurity)
4. Event sent IMMEDIATELY to re:Search
```

**In re:Search:**
```
5. re:Search receives security event
6. re:Search calls GetCase
7. GetCase returns security level: Sealed
8. re:Search re-indexes case
9. Case hidden from public immediately
```

### Key Implementation Points

**Immediate Transmission:**
- Security events must be sent immediately
- Don't queue with other events
- Don't wait for batch processing
- Highest priority handling

**GetCase Response:**
- Return current security level (Sealed)
- Include seal date if available
- All documents inherit Sealed security (unless more restrictive)

**Critical Mistake:**
Delaying security event transmission exposes confidential information to public for hours or days.

[See Security Logic →](./security-logic.md)

---

## Scenario 4: Document Filed with Mixed Security

### The Scenario
New document filed in public case, but document itself is sealed by court order.

### What Happens

**In CMS:**
```
1. Document filed and stored
2. Document marked as Sealed
3. Case remains Public
4. CMS generates NotifyCaseEvent (EventType: DocumentFiling)
5. Event sent to re:Search
```

**In re:Search:**
```
6. re:Search receives event
7. re:Search calls GetCase
8. GetCase returns:
   - Case security: Public
   - Document security: Sealed
9. re:Search indexes case as public
10. Specific document restricted from public access
```

### Key Implementation Points

**Security Hierarchy:**
- Case security is baseline
- Documents can be MORE restrictive
- Documents CANNOT be less restrictive than case

**GetCase Response:**
```
Case: Public
  Document 1: Public (inherits from case)
  Document 2: Sealed (overrides case security)
  Document 3: Public (inherits from case)
```

**GetDocument Enforcement:**
- GetDocument must check document security
- Must deny access to sealed documents
- Return appropriate access denied error

**Common Mistake:**
GetCase shows document as Sealed, but GetDocument returns it anyway because security not checked at retrieval time.

---

## Scenario 5: Case Type Mismatch (Texas JCIT)

### The Scenario
Case displays padlock icon in re:Search UI even though case is public in CMS and GetCase returns correct security.

### What Happens

**Problem:**
```
CMS sends case type: "Injury or Damage - Other Injury of Damage"
JCIT standard requires: "Other Injury or Damage"
re:Search cannot match case type to access rules
re:Search defaults to restricted classification
Padlock appears in UI
```

**Root Cause:**
- Case type string doesn't exactly match JCIT standard
- re:Search cannot map to public access rule
- System defaults to restrictive classification for safety

### Resolution

**CMS Must:**
1. Update case type mapping to exact JCIT standard
2. Correct value: "Other Injury or Damage"
3. Resend NotifyCaseEvent (EventType: CaseFiling or CaseSecurity)
4. re:Search will reindex with correct access rules
5. Padlock removed from UI

### Key Lesson

**Exact Case Type Matching Required:**
- Case types must match market standards exactly
- No partial matching or fuzzy logic
- Incorrect mapping → Restricted access (safe default)
- Validate all case type mappings during development

**Texas courts:** All case types must match [JCIT Technology Standards](../market-standards/texas/case-types/README.md)

[See Case Type Mapping Best Practices →](../integration-guide/best-practices.md#map-case-types-accurately)

---

## Scenario 6: Case-Sensitive Security Values

### The Scenario
Case should be sealed but remains Public in re:Search. Security metadata appears correct in GetCase response.

### What Happens

**Problem:**
```
CMS sends: <tyler:CaseSecurity>SEALED</tyler:CaseSecurity>
re:Search expects: <tyler:CaseSecurity>Sealed</tyler:CaseSecurity>
                                        ↑
                                   Capital S, rest lowercase
Security mapping is case-sensitive
Value doesn't match → Falls through to default (Public)
```

**Symptoms:**
- Case remains Public despite being sealed in CMS
- No errors in re:Search logs
- Multiple GetCase calls may occur (parsing issues)
- Security appears correct in XML but doesn't take effect

### Resolution

**CMS Must:**
1. Update security value to exact format: "Sealed"
2. Resend NotifyCaseEvent (EventType: CaseSecurity)
3. re:Search will reindex with correct security
4. Case becomes properly sealed

### Key Lesson

**Security Values are Case-Sensitive:**
- Must be exactly: `Sealed` (not SEALED, sealed, or SeAlEd)
- Also applies to: `Public`, `Confidential`
- No tolerance for case variations
- Always use proper case formatting

**Best Practice:**
Use constants in CMS configuration rather than typing values manually to avoid case sensitivity issues.

---

## Scenario 7: Events Not Triggering GetCase

### The Scenario
CMS sends NotifyCaseEvent but re:Search never calls GetCase. Case not updated in re:Search.

### Common Causes

**1. Message Broker Issue (RabbitMQ)**

**Problem:**
- Message broker service (RabbitMQ) has stopped or stalled
- Events queued but not being delivered to re:Search
- Messages remain in "Ready" state indefinitely

**Symptoms:**
- CMS logs show event sent successfully
- No GetCase call received by CMS
- Multiple events accumulating without processing
- No errors visible to CMS

**Resolution:**
- Contact Tyler Technical Project Manager
- Submit support ticket to BIS team
- BIS team will restart RabbitMQ services
- Queued messages will then process

**2. Event Format Invalid**

**Problem:**
- Event XML doesn't match expected schema
- Required fields missing
- XML structure malformed

**Resolution:**
- Validate event XML against schema
- Check for required fields (case tracking ID, event type)
- Review event generation logic

**3. Network Connectivity**

**Problem:**
- Network issue between CMS and re:Search
- Firewall blocking outbound connections
- Timeout during transmission

**Resolution:**
- Verify network connectivity
- Check firewall rules
- Test connectivity to re:Search endpoints
- Implement retry logic

### Key Lesson

**Event Queue Monitoring:**
- Monitor event transmission success rate
- Alert on accumulating failed events
- Don't assume successful send means successful delivery
- Implement event queue with visibility into status

**When Events Don't Process:**
1. Check CMS logs for transmission errors
2. If transmission successful but no GetCase, contact TPM
3. BIS team can check message broker status
4. May need RabbitMQ service restart

---

## Scenario 8: Complex Case with Performance Issues

### The Scenario
Case with 15 parties, 8 attorneys, and 75 documents. GetCase times out when re:Search requests it.

### What Happens

**Problem:**
```
re:Search calls GetCase
CMS makes 1 query for case
CMS makes 15 separate queries for parties (N+1 problem)
CMS makes 8 separate queries for attorneys
CMS makes 75 separate queries for documents
Total: 99 database queries
Query time exceeds 30 seconds
Connection timeout
re:Search retries, same problem
Case never indexes
```

### Resolution

**Optimize Query Strategy:**
```
Instead of 99 queries:
- 1 query for case with JOINed parties
- 1 query for attorneys with party associations
- 1 query for documents with metadata
Total: 3 queries
Response time: < 2 seconds
```

**Additional Optimizations:**
- Add database indexes on case_id foreign keys
- Use query execution plans to identify slow queries
- Consider response caching (short TTL)
- Test with production-complexity cases

### Key Lesson

**Test with Real Complexity:**
- Don't just test with simple 2-party cases
- Find your court's most complex cases
- Test GetCase with those cases
- Optimize before production deployment

**Performance Targets:**
- Simple cases: < 1 second
- Average cases: < 3 seconds
- Complex cases: < 5 seconds maximum

[See Performance Optimization →](../integration-guide/best-practices.md#optimize-getcase-response-times)

---

## Scenario 9: Publication Delay Not Applied (Texas)

### The Scenario
Texas criminal case filed. Case immediately visible to public, but should have 31-day publication delay.

### What Happens

**Problem:**
```
Criminal case filed (case type: CR)
CMS sends CaseFiling event
GetCase returns:
  - CaseType: CR
  - CaseSecurity: Public
  - FilingDate: 2024-10-01

Expected: Case hidden from public for 31 days
Actual: Case visible immediately
```

**Root Cause:**
This is typically a re:Search configuration issue, not a CMS issue. The 31-day delay is applied by re:Search based on case type and filing date.

**CMS Responsibilities:**
- Send correct case type (CR for criminal)
- Send correct filing date
- Set security appropriately (Public for most criminal cases)

**re:Search Applies Delay:**
- re:Search recognizes criminal case
- Applies 31-day delay automatically
- Public users cannot see case until delay expires
- Court staff can see case immediately

### Key Lesson

**Publication Delays are Applied by re:Search:**
- CMS doesn't need to calculate delays
- CMS sends case type and filing date correctly
- re:Search applies market-specific delay rules
- If delays not working, this is re:Search configuration issue

**CMS Must Provide:**
- Correct case type mapping to market standard
- Accurate filing date
- Appropriate security level

[See Texas Security Requirements →](../market-standards/texas/security/README.md)

---

## Scenario 10: Document Not Available

### The Scenario
User clicks document in re:Search, but document cannot be retrieved from CMS.

### What Happens

**In re:Search:**
```
User clicks document link
re:Search calls GetDocument
```

**In CMS - Scenarios:**

**Scenario A: Document Archived**
```
Document moved to archive storage
Retrieval requires 5-10 minutes
CMS returns SOAP fault: 503 Service Unavailable
Detail: "Document archived, retrieval in progress"
```

**Scenario B: Document Missing**
```
Document file not found in storage
CMS returns SOAP fault: 404 Not Found
Detail: "Document file not available"
```

**Scenario C: Access Denied**
```
Document is sealed
User is public (not authorized)
CMS returns SOAP fault: 403 Forbidden
Detail: "Document access denied"
```

### Key Implementation Points

**Return Specific Error Codes:**
- 404: Document doesn't exist
- 403: Access denied (sealed/confidential)
- 503: Temporarily unavailable (archived, processing)
- 500: System error

**Include Helpful Details:**
- Estimated retrieval time (if archived)
- Reason for denial (if access issue)
- Instructions for user (if applicable)

**Don't:**
- Return generic "Error" message
- Expose system details in error
- Return successful response with empty content

### Key Lesson

**Document Availability Varies:**
- Not all documents immediately available
- Archived documents need retrieval time
- System errors can occur
- Return appropriate errors, not failures

---

## Scenario 11: Documents Return Empty (0 KB Files)

### The Scenario
User clicks document in re:Search UI. Document downloads but is 0 KB and opens as blank page.

### What Happens

**In re:Search:**
```
User clicks document link
re:Search calls GetDocument with document ID
```

**In CMS - Problem:**
```
GetDocument called for document 844891
CMS returns GetDocumentResponse
BUT: BinaryBase64Object is empty (contains "0")
AND: PageCount is 0
```

**Result:**
```
User receives 0 KB file
File cannot be opened
Adobe shows error: "not a supported file type or file has been damaged"
```

### Root Cause

**CMS returning empty document payload:**
- `<BinaryBase64Object>` tag is empty or contains just "0"
- `<PageCount>` shows 0 instead of actual page count
- Document metadata is present (ID, title, description)
- But actual file content is missing

**Common CMS-side causes:**
- Document file not found in storage
- Storage path misconfigured
- Permission issues accessing document files
- Error during base64 encoding
- Database references document but file doesn't exist

### How to Identify

**Symptom 1: User Experience**
- Document appears in case display
- Download completes immediately (0 KB)
- File won't open or shows as blank

**Symptom 2: Integration Troubleshooting Tool**
```
Error found: CMS sending empty/non-existent document.
Length is 0 in the XML below.
```

**Symptom 3: GetDocumentResponse XML**
```xml
<DocumentAttachment>
  <BinaryBase64Object xmlns="http://niem.gov/niem/niem-core/2.0">
    0
  </BinaryBase64Object>
  <BinaryDescriptionText xmlns="http://niem.gov/niem/niem-core/2.0">
    2024.03.01 Signed Complaint for Divorce
  </BinaryDescriptionText>
  <BinaryLocationURI xmlns="http://niem.gov/niem/niem-core/2.0">
    844891.pdf
  </BinaryLocationURI>
  <PageCount xmlns="urn:tyler:ecf:extensions:Common">
    0
  </PageCount>
</DocumentAttachment>
```

**Key indicators:**
- BinaryBase64Object is empty or "0"
- PageCount is 0
- BinaryLocationURI and description are present (metadata exists)
- No error code returned (CMS reports "No error")

### Resolution

**CMS Must Fix:**

1. **Verify document file exists**
   - Check document storage location
   - Verify file path in database matches actual file location
   - Confirm file hasn't been moved or deleted

2. **Correct GetDocument implementation**
   - Read document file from storage
   - Base64 encode the file content
   - Populate BinaryBase64Object with encoded content
   - Set PageCount to actual page count

3. **Test before returning**
   - Verify BinaryBase64Object is not empty
   - Verify PageCount > 0
   - Verify base64 string is valid

**Correct GetDocumentResponse:**
```xml
<DocumentAttachment>
  <BinaryBase64Object xmlns="http://niem.gov/niem/niem-core/2.0">
    VBERi0xLjQKJwsedfgwefGRThrthre34563VErgergbe...
    [full base64-encoded document content]
  </BinaryBase64Object>
  <BinaryDescriptionText xmlns="http://niem.gov/niem/niem-core/2.0">
    2024.03.01 Signed Complaint for Divorce
  </BinaryDescriptionText>
  <BinaryLocationURI xmlns="http://niem.gov/niem/niem-core/2.0">
    844891.pdf
  </BinaryLocationURI>
  <PageCount xmlns="urn:tyler:ecf:extensions:Common">
    5
  </PageCount>
</DocumentAttachment>
```

### Key Implementation Points

**Document Retrieval Process:**
```
1. Receive GetDocument request with document ID
2. Query database for document metadata
3. Locate physical document file
4. Read file from storage
5. Base64 encode file content
6. Populate response with encoded content
7. Set accurate PageCount
8. Return complete response
```

**Validation Before Returning:**
- Document file exists in storage
- File is readable (not corrupted)
- Base64 encoding succeeded
- BinaryBase64Object contains data
- PageCount matches actual document pages
- Response size is reasonable (not 0 bytes)

**Error Handling:**
If document cannot be retrieved:
- Return appropriate SOAP fault (404, 503, etc.)
- Don't return empty BinaryBase64Object
- Include meaningful error message
- Log error for investigation

### Common Mistakes

**Mistake 1: Returning Empty Response Instead of Error**
```
Problem: Document not found, but return empty BinaryBase64Object
Result: User gets 0 KB file, no explanation
Should: Return 404 SOAP fault with error message
```

**Mistake 2: Not Validating Response Before Sending**
```
Problem: Don't check if BinaryBase64Object is populated
Result: Empty documents sent to users
Should: Validate response has content before returning
```

**Mistake 3: Database Reference but No File**
```
Problem: Document metadata in database, file doesn't exist in storage
Result: GetDocument returns empty content
Should: Check file exists before attempting to return it
```

### Troubleshooting Steps for CMS

**Step 1: Check document storage**
```
- Where does CMS store document files?
- Does file exist at expected location?
- Are file permissions correct?
- Is storage path configured correctly?
```

**Step 2: Check GetDocument logic**
```
- Does GetDocument read file from storage?
- Is base64 encoding working?
- Are errors being logged?
- Is response being validated?
```

**Step 3: Review CMS logs**
```
- Check logs at time of GetDocument call
- Look for file access errors
- Look for encoding errors
- Check storage path errors
```

**Step 4: Test with working document**
```
- Find document that works
- Find document that doesn't work
- Compare storage paths
- Compare file accessibility
```

### Key Lesson

**GetDocument Must Return Complete Document:**
- BinaryBase64Object must contain full document content
- PageCount must reflect actual pages
- Empty or zero values indicate CMS problem
- This is CMS-side issue, not re:Search issue

**re:Search Cannot Fix This:**
- re:Search can only work with data CMS provides
- Empty response = empty file to user
- CMS must correct document retrieval logic
- Once CMS returns proper content, re:Search will work correctly

---
## Scenario 12: Multiple Filings Appear as Single Filing

### The Scenario
Case has two distinct filing events, but re:Search UI shows only one filing with all documents grouped together incorrectly.

### What Happens

**In EFM:**
```
First filing submitted: Filing A
EFM assigns unique ID: 3d281cb3-b8da-489d-bdee-3f6977c13bdf
EFM calls CMS RecordFiling with this ID

Second filing submitted: Filing B  
EFM assigns unique ID: 8c4e2a3b-1f67-4d0f-9e78-0c2d1b5a9e7f
EFM calls CMS RecordFiling with this ID
```

**In CMS - Problem:**
```
CMS processes both filings
CMS sends NotifyDocketingComplete
BUT: Both filings use same nc:IdentificationID
Both reference: 3d281cb3-b8da-489d-bdee-3f6977c13bdf
```

**In re:Search:**
```
re:Search receives NotifyDocketingComplete
Sees two filing blocks with same nc:IdentificationID
Processes first filing (Filing A)
Treats second filing as duplicate (Filing B ignored)
Result: Only one filing appears in UI
All documents incorrectly grouped under Filing A
```

**User Experience:**
- UI shows only ONE filing row
- That filing shows documents from BOTH filings mixed together
- Second filing completely missing from display
- Document associations incorrect

### Root Cause

**Duplicate nc:IdentificationID in NotifyDocketingComplete:**

Each filing MUST have its own unique `nc:IdentificationID`. This ID is the permanent, unique identifier for that specific filing event. When re:Search receives a message with two filing entries sharing the same ID, it cannot distinguish them as separate events.

**The nc:IdentificationID:**
- Is provided by EFM in original RecordFiling message
- Uniquely identifies that specific filing event
- Must be preserved by CMS in all callback messages
- Must never be reused for different filings

### How to Identify

**Symptom 1: UI Display**
- Only one filing row visible
- Should be two (or more) separate filings
- All documents grouped under single filing

**Symptom 2: Integration Troubleshooting Tool**
- Shows `noop_` message for second filing
- Indicates re:Search did not process the filing
- Message received but treated as duplicate

**Symptom 3: NotifyDocketingComplete XML**

**Problematic structure:**
```xml
<!-- First filing -->
<nc:DocumentIdentification s:id="FilingID001">
  <nc:IdentificationID>3d281cb3-b8da-489d-bdee-3f6977c13bdf</nc:IdentificationID>
</nc:DocumentIdentification>

<!-- Second filing - PROBLEM: Same ID -->
<nc:DocumentIdentification s:id="FilingID002">
  <nc:IdentificationID>3d281cb3-b8da-489d-bdee-3f6977c13bdf</nc:IdentificationID>
</nc:DocumentIdentification>
```

Both filings share the exact same `nc:IdentificationID`. re:Search processes the first and discards the second as a duplicate.

### Resolution

**CMS Must Fix Filing ID Management:**

1. **Track original EFM filing IDs**
   - When EFM calls RecordFiling, it provides unique filing ID
   - CMS must store this ID for that specific filing
   - Never reuse or duplicate these IDs

2. **Preserve IDs in callback messages**
   - Each filing in NotifyDocketingComplete must use its original EFM ID
   - Filing A → Use Filing A's original ID
   - Filing B → Use Filing B's original ID

3. **Correct NotifyDocketingComplete structure**
```xml
<!-- First filing - uses its original EFM ID -->
<nc:DocumentIdentification s:id="FilingID001">
  <nc:IdentificationID>3d281cb3-b8da-489d-bdee-3f6977c13bdf</nc:IdentificationID>
</nc:DocumentIdentification>

<!-- Second filing - uses ITS original EFM ID (different) -->
<nc:DocumentIdentification s:id="FilingID002">
  <nc:IdentificationID>8c4e2a3b-1f67-4d0f-9e78-0c2d1b5a9e7f</nc:IdentificationID>
</nc:DocumentIdentification>
```

4. **Link documents to correct filing**

Each document must reference the correct filing:
```xml
<tyler:ReviewedLeadDocument>
  <!-- Document details -->
  <tyler:FilingIdentification>
    <tyler:FilingIdentificationReference s:ref="FilingID001"></tyler:FilingIdentificationReference>
  </tyler:FilingIdentification>
</tyler:ReviewedLeadDocument>

<tyler:ReviewedLeadDocument>
  <!-- Different document, different filing -->
  <tyler:FilingIdentification>
    <tyler:FilingIdentificationReference s:ref="FilingID002"></tyler:FilingIdentificationReference>
  </tyler:FilingIdentification>
</tyler:ReviewedLeadDocument>
```

### Verification Steps

**For CMS to verify fix:**

1. **Check RecordFiling storage**
   - Does CMS store the filing ID from EFM's RecordFiling call?
   - Is this ID associated with the correct filing?
   - Is each filing tracked separately?

2. **Check NotifyDocketingComplete generation**
   - Does each filing block get its own unique nc:IdentificationID?
   - Are these IDs the ones originally provided by EFM?
   - Are there any ID duplications?

3. **Test with multiple filings**
   - Submit two filings to same case
   - Verify CMS tracks both with separate IDs
   - Verify NotifyDocketingComplete contains both with unique IDs
   - Verify both appear as separate filings in re:Search UI

### Expected Result After Fix

**In re:Search UI:**
```
Events section shows:

Filing #1 (3d281cb3-b8da-489d-bdee-3f6977c13bdf)
├─ Lead Document 1
├─ Attachment 1
└─ Attachment 2

Filing #2 (8c4e2a3b-1f67-4d0f-9e78-0c2d1b5a9e7f)
├─ Lead Document 2
├─ Attachment 3
└─ Attachment 4
```

Each filing appears separately with its own documents correctly associated.

### Key Implementation Points

**Filing ID Lifecycle:**
```
1. EFM → RecordFiling → Provides unique filing ID
2. CMS → Store filing with this ID
3. CMS → Process filing
4. CMS → NotifyDocketingComplete → Return SAME ID
5. re:Search → Index using this ID
```

**Common Mistakes:**

**Mistake 1: Using Same ID for All Filings**
```
Problem: CMS generates or hardcodes one ID for all filings
Result: All filings treated as duplicates, only first processes
Should: Use unique ID from EFM for each filing
```

**Mistake 2: Not Storing Original EFM ID**
```
Problem: CMS doesn't save ID from RecordFiling call
Result: Can't include correct ID in callback message
Should: Store EFM filing ID when RecordFiling is received
```

**Mistake 3: Using CMS Internal ID Instead**
```
Problem: CMS uses its own internal filing ID instead of EFM's
Result: re:Search doesn't recognize filings correctly
Should: Use EFM's nc:IdentificationID, can include CMS ID separately in tyler:CMSID
```

### Relationship to RecordFiling

**Important Context:**

The nc:IdentificationID is NOT generated by the CMS. This ID comes from EFM in the original RecordFiling message:
```xml
<RecordFilingMessage>
  <nc:DocumentIdentification>
    <!-- EFM provides this ID -->
    <nc:IdentificationID>3d281cb3-b8da-489d-bdee-3f6977c13bdf</nc:IdentificationID>
  </nc:DocumentIdentification>
  <!-- Filing details -->
</RecordFilingMessage>
```

The CMS must:
1. Extract this ID from RecordFiling
2. Store it with the filing
3. Return it in NotifyDocketingComplete
4. Never change or duplicate it

### Key Lesson

**Filing IDs Must Be Unique:**
- Each filing event has its own unique ID from EFM
- CMS must preserve and return these IDs exactly
- Duplicate IDs cause filings to be treated as duplicates
- Only first filing with duplicate ID will process
- This is CMS data management issue, not re:Search issue

**re:Search Behavior:**
- re:Search uses nc:IdentificationID as primary key for filings
- Duplicate ID = duplicate filing = skip processing
- `noop_` message indicates duplicate detected
- Once CMS provides unique IDs, re:Search will process all filings correctly

---

## Related Documentation

## Related Documentation

### For Technical Understanding
→ [Architecture Overview](./architecture-overview.md)  
→ [Event Type Logic](./event-type-logic.md)  
→ [Security Logic](./security-logic.md)

### For Implementation Guidance
→ [Best Practices](../integration-guide/best-practices.md)  
→ [Common Mistakes](./common-mistakes.md)

### For Market Requirements
→ [Texas Standards](../market-standards/texas/README.md)  
→ [Illinois Standards](../market-standards/illinois/README.md)

### For Problem Solving
→ [Troubleshooting Guide](../troubleshooting/README.md)

---

**Last Updated:** January 2025