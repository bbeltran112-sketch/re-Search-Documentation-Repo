# Common Mistakes

> **Breadcrumb:** [Home](../README.md) > [Implementation Playbook](./README.md) > Common Mistakes

Technical pitfalls to avoid when integrating with re:Search. Learn from issues encountered in past integrations.

---

## Event Generation Mistakes

### ❌ Mistake: Generating Events Before Transaction Commit

**The Problem:**
Events sent before database transaction commits can result in re:Search calling GetCase and receiving stale or incorrect data if the transaction rolls back.

**Example Scenario:**
```
1. User updates case in CMS
2. CMS sends NotifyCaseEvent
3. Database transaction fails and rolls back
4. re:Search calls GetCase
5. GetCase returns old data (change never committed)
6. re:Search indexes incorrect state
```

**The Solution:**
Configure event generation to occur only after successful database commit. This ensures GetCase always returns committed data.

**How to Identify:**
- Cases in re:Search don't match CMS
- Intermittent data inconsistencies
- GetCase returns data that doesn't match expected changes

---

### ❌ Mistake: Not Sending Events for "Minor" Changes

**The Problem:**
Skipping events for "small" changes like name corrections or address updates causes re:Search data to drift from CMS over time.

**Example Scenario:**
```
User corrects party name spelling:
  "John Smyth" → "John Smith"

CMS developer thinks:
  "Just a spelling fix, not worth an event"

Result:
  re:Search still shows "John Smyth"
  User searches for "John Smith" → Case not found
```

**The Solution:**
Send events for ALL case changes that should be reflected in re:Search. Let re:Search decide what's important, not your CMS.

**Events should be sent for:**
- Name corrections
- Address updates
- Minor party information changes
- Any docket entry additions
- Any document additions
- All security changes (no matter how small)

---

### ❌ Mistake: Delaying Security Events

**The Problem:**
Security events sent hours or days after security changes expose confidential information or hide public records.

**Example Scenario:**
```
Monday 9:00 AM: Judge orders case sealed
Monday 9:05 AM: CMS updates case security to Sealed
Monday 9:05 AM: CMS queues event for nightly batch

Monday 9:06 AM - Tuesday 2:00 AM: Case visible to public (16+ hours)
Tuesday 2:00 AM: Batch runs, CaseSecurity event sent
Tuesday 2:05 AM: Case finally hidden from public
```

**The Solution:**
Send CaseSecurity and DocumentSecurity events immediately when security changes. Do not batch or delay these events.

**How to Identify:**
- Sealed cases visible to public for hours
- Users report seeing cases they shouldn't
- Security changes don't take effect quickly

---

### ❌ Mistake: Using Wrong Event Types

**The Problem:**
Using incorrect event types confuses the event audit trail and may trigger incorrect processing.

**Common Wrong Choices:**

**Wrong: Using CaseParty for Attorney Changes**
```
Attorney added to case → CaseParty sent

Problem: Semantically wrong, makes audit trail confusing
Should be: CasePartyAttorney
```

**Wrong: Using CaseDelete for Case Closure**
```
Case disposed → CaseDelete sent

Problem: Case is permanently removed from re:Search
Should be: CaseDisposition (case closed but still searchable)
```

**Wrong: Using Generic Event for Everything**
```
Everything changed → Generic "CaseUpdate" event

Problem: No semantic meaning, can't track what changed
Should be: Specific event types for each change
```

**The Solution:**
Use the event type that matches the actual change. Consult [Event Type Logic](./event-type-logic.md) for guidance.

---

### ❌ Mistake: No Retry Logic for Failed Events

**The Problem:**
Events that fail to send are permanently lost, causing data to become stale in re:Search.

**Example Scenario:**
```
Network blip occurs:
  CMS sends NotifyCaseEvent → Network timeout
  
CMS has no retry logic:
  Event lost forever
  
Result:
  Case never updated in re:Search
  User searches, finds stale data
  No error logged, issue invisible
```

**The Solution:**
Implement event queue with automatic retry logic. Failed events should retry 3-5 times before being marked as permanently failed.

**How to Identify:**
- Cases not updating in re:Search
- No errors in logs
- "Silent" failures
- Users report data not matching CMS

---

## GetCase Response Mistakes

### ❌ Mistake: Missing Required Fields

**The Problem:**
GetCase responses missing required fields cause indexing failures or poor search results.

**Commonly Missed Fields:**
- CaseSecurity (even if case is public, must be included)
- Party roles (Plaintiff, Defendant, etc.)
- Attorney bar numbers
- Document filing dates
- Document security (when different from case)
- Case type (must map to market standards)

**Example Impact:**
```
GetCase response omits party roles:
  
re:Search doesn't know who is Plaintiff vs Defendant
  
User searches: "cases where John Smith is defendant"
  
Result: Case not found because role is unknown
```

**The Solution:**
Validate GetCase responses against a checklist of required fields before certification. Test with actual case data complexity.

**How to Identify:**
- re:Search logs showing validation errors
- Cases not appearing in expected searches
- Missing information in re:Search case display
- Certification failures

---

### ❌ Mistake: Returning Stale Cached Data

**The Problem:**
GetCase returns cached data that doesn't reflect recent changes in CMS.

**Example Scenario:**
```
10:00 AM: Party name updated in CMS
10:01 AM: NotifyCaseEvent sent
10:02 AM: re:Search calls GetCase
10:02 AM: GetCase returns cached response from 9:00 AM
10:03 AM: re:Search indexes old party name

Result: Change not reflected in re:Search despite event
```

**The Solution:**
If using response caching:
- Use short TTL (5-10 minutes maximum)
- Invalidate cache when case data changes
- Never cache longer than 10 minutes

**Better: Don't cache GetCase responses unless performance requires it.**

**How to Identify:**
- Events sent but changes not appearing
- Intermittent data inconsistencies
- Changes appear after long delay
- GetCase response time very fast (< 100ms) suspicious

---

### ❌ Mistake: N+1 Query Performance Problems

**The Problem:**
GetCase makes separate database query for each party, attorney, and document, resulting in hundreds of queries for complex cases.

**Example Scenario:**
```
Case has:
  - 1 case record
  - 10 parties
  - 5 attorneys
  - 20 documents
  
Poor implementation:
  1 query for case
  + 10 queries for parties
  + 5 queries for attorneys  
  + 20 queries for documents
  = 36 total queries
  
Result: GetCase takes 5-10 seconds for this case
```

**The Solution:**
Use JOIN operations to retrieve all case data in minimal queries (ideally 1-3 queries total).

**How to Identify:**
- GetCase response times > 5 seconds
- Database shows high query count per case
- Database CPU usage high during GetCase
- Timeout errors from re:Search

---

### ❌ Mistake: Not Handling Complex Cases

**The Problem:**
GetCase works fine for simple test cases but times out or fails for complex real-world cases.

**Complex Case Characteristics:**
- 10+ parties
- 5+ attorneys
- 50+ documents
- 100+ docket entries
- Multiple cross-references

**Example Scenario:**
```
Testing: Cases with 2 parties, 1 attorney, 5 documents
  GetCase: 500ms (looks great!)

Production: Case with 15 parties, 8 attorneys, 75 documents
  GetCase: 45 seconds → Timeout
```

**The Solution:**
Test GetCase with production-complexity cases during development, not just simple test cases.

**How to Identify:**
- Works in test, fails in production
- Timeouts on specific large cases
- Performance degrades with case complexity

---

## GetDocument Mistakes

### ❌ Mistake: Not Enforcing Security in GetDocument

**The Problem:**
GetDocument returns sealed/confidential documents to public users because security check is only in GetCase.

**Example Scenario:**
```
Case is public, but document is sealed:

GetCase: Returns case with document marked as Sealed ✓

GetDocument: Doesn't check security, returns PDF ✗

Result: Public user gets sealed document content
```

**The Solution:**
GetDocument MUST enforce security independently. Always check:
- Case security level
- Document security level
- User role
- Access permissions

Never assume document is safe to return without checking.

**How to Identify:**
- Security audit shows sealed documents accessed by public
- Users report seeing documents they shouldn't
- GetDocument has no security checking logic

---

### ❌ Mistake: GetDocument Security Doesn't Match GetCase

**The Problem:**
GetCase says document is sealed, but GetDocument returns it anyway due to inconsistent security logic.

**Example Scenario:**
```
GetCase response:
  <Document>
    <DocumentID>DOC-123</DocumentID>
    <DocumentSecurity>
      <SecurityLevel>Sealed</SecurityLevel>
    </DocumentSecurity>
  </Document>

GetDocument for DOC-123:
  Checks case security (Public) ✓
  Doesn't check document security ✗
  Returns document to public user ✗
```

**The Solution:**
GetDocument security logic must use the SAME rules as GetCase. Use shared security checking logic if possible.

**How to Identify:**
- re:Search shows document as sealed but users can view it
- Inconsistent security behavior
- Security works in search but not in document retrieval

---

### ❌ Mistake: Not Handling Missing Documents

**The Problem:**
GetDocument returns error when document file is missing, but doesn't return appropriate SOAP fault.

**Example Scenario:**
```
GetDocument called for DOC-123:
  
File missing from document storage:
  
CMS returns: Generic "Internal Server Error"
  
re:Search can't tell if:
  - Document doesn't exist
  - Document is archived
  - Temporary system problem
  - Permission issue
```

**The Solution:**
Return specific SOAP faults indicating the exact problem:
- 404: Document doesn't exist
- 503: Document archived, retrieval in progress
- 403: Access denied
- 500: Temporary system error

**How to Identify:**
- Generic error messages in re:Search logs
- Unable to distinguish between error types
- Users see generic "unavailable" message

---

## Security Mistakes

### ❌ Mistake: Document Security Less Restrictive Than Case

**The Problem:**
Documents marked as more visible than their case, violating fundamental security principle.

**Example Scenario:**
```
Case security: Sealed

Document security: Public ✗

Result: Invalid configuration
```

**The Rule:**
Document security must be >= case security in restrictiveness:
- If case is Sealed, documents must be at least Sealed
- If case is Confidential, documents must be Confidential
- Documents can be MORE restrictive, never LESS

**The Solution:**
Validate document security against case security. Use most restrictive level if misconfigured.

**How to Identify:**
- Sealed cases with public documents
- Confidential cases with sealed documents
- Security validation errors in re:Search logs

---

### ❌ Mistake: Security Not Synchronized with CMS

**The Problem:**
re:Search security doesn't match current CMS security due to missing events or stale data.

**Example Scenario:**
```
Monday: Case sealed in CMS
Monday: No CaseSecurity event sent ✗
Tuesday: Case still showing as public in re:Search
Wednesday: User reports seeing sealed case
```

**Common Causes:**
- CaseSecurity event not sent
- Event sent but GetCase returns old security
- Security cached too long
- Batch process hasn't run

**The Solution:**
- Send CaseSecurity events immediately
- GetCase must return current security
- Don't cache security settings
- Test security change timing

**How to Identify:**
- Security in re:Search doesn't match CMS
- Delayed security changes
- Inconsistent security enforcement

---

### ❌ Mistake: Ignoring Market-Specific Security Rules

**The Problem:**
Integration doesn't implement jurisdiction-specific security requirements like publication delays.

**Example Scenario (Texas):**
```
Criminal case filed:

CMS sends CaseFiling event ✓

GetCase returns Public security ✓

re:Search makes case public immediately ✗

Problem: Should have 31-day publication delay
```

**Market-Specific Rules:**
- **Texas:** 31-day delay for criminal cases, 180-day for sealed documents
- **Illinois:** AOIC-specific rules
- **Other markets:** May have their own requirements

**The Solution:**
Understand and implement your market's security rules. Work with TPM to clarify requirements.

**How to Identify:**
- Cases visible when they shouldn't be
- Publication timing doesn't match market rules
- Certification failures on security tests

---

## Case Type Mapping Mistakes

### ❌ Mistake: Incorrect Case Type Mappings

**The Problem:**
CMS case types mapped to wrong market-standard codes.

**Example Scenario (Texas):**
```
CMS case type: "Criminal Misdemeanor"
Mapped to: "FM" (Family) ✗
Should be: "CR" (Criminal)

Result:
  - Case indexed under wrong type
  - Wrong security rules applied (family vs criminal)
  - Wrong publication delays
  - Case doesn't appear in correct searches
```

**The Solution:**
Validate all case type mappings with Tyler TPM before testing. Don't guess at mappings.

**How to Identify:**
- Cases appearing under wrong case type
- Security rules not working as expected
- Users can't find cases by type
- Certification failures

---

### ❌ Mistake: No Mapping for All CMS Case Types

**The Problem:**
Some CMS case types don't have mappings to market codes, causing indexing failures.

**Example Scenario:**
```
Court adds new case type "Mental Health Commitment"

CMS has no mapping configured:
  
Case filed → CaseFiling event sent
  
GetCase returns unmapped case type "MHC"
  
re:Search doesn't know how to handle it → Error
```

**The Solution:**
- Map ALL case types used by your court
- Define default mapping for unknown types
- Log warnings when unknown types encountered
- Review mapping completeness during testing

**How to Identify:**
- Cases failing to index
- "Unknown case type" errors in logs
- Cases showing as "Other" type unexpectedly

---

### ❌ Mistake: Case Type Changed Without Event

**The Problem:**
Case type changes in CMS but no event sent to trigger re-indexing.

**Example Scenario:**
```
Case starts as "CV" (Civil)
Case type corrected to "FM" (Family)
No event sent ✗

Result:
  Case still indexed as Civil in re:Search
  Wrong security rules applied
  Case in wrong category for searches
```

**The Solution:**
Send appropriate event (typically CaseFiling or generic case update) when case type changes.

**How to Identify:**
- Case type in re:Search doesn't match CMS
- Cases in wrong category
- Users report incorrect case type

---

## Integration Architecture Mistakes

### ❌ Mistake: No Event Queue, Direct Sending

**The Problem:**
Events sent directly to re:Search without queue means any failure loses the event permanently.

**Example Scenario:**
```
CMS updates case:
  
Try to send NotifyCaseEvent directly:
  
Network timeout → Event lost ✗
  
No retry, no queue, no recovery
  
Case never updated in re:Search
```

**The Solution:**
Use event queue pattern:
- Write events to queue table
- Background process sends from queue
- Automatic retry on failure
- Failed events flagged for investigation

**How to Identify:**
- "Random" cases not updating
- No failed event logs
- Silent failures
- Can't explain why case isn't updated

---

### ❌ Mistake: Inadequate Logging

**The Problem:**
Integration issues can't be diagnosed because API interactions aren't logged.

**Example Scenario:**
```
User: "Case CV-2024-001234 isn't showing correct data"

Support checks logs:
  No GetCase logs found
  No event logs found
  No error logs found
  
Can't determine:
  - Was event sent?
  - Did GetCase return data?
  - What data was returned?
  - When did this happen?
```

**The Solution:**
Log all API interactions:
- Every NotifyCaseEvent sent
- Every GetCase/GetDocument call received
- Success/failure status
- Response times
- Error details

**How to Identify:**
- Can't troubleshoot integration issues
- No visibility into API activity
- No audit trail of events
- Support can't diagnose problems

---

### ❌ Mistake: No Monitoring or Alerting

**The Problem:**
Integration breaks but nobody notices until users complain.

**Example Scenario:**
```
Friday 5 PM: Integration stops sending events
  
Weekend: No monitoring, nobody notices
  
Monday 8 AM: Court opens
Monday 9 AM: Users report stale data
Monday 10 AM: IT discovers event queue stuck

48+ hours of missed events
```

**The Solution:**
Implement monitoring and alerting:
- Event queue depth
- Failed event count
- API error rates
- Processing lag
- Alert when thresholds exceeded

**How to Identify:**
- Issues discovered by users, not monitoring
- Long delays before issue detection
- No visibility into integration health

---

### ❌ Mistake: No Testing of Error Scenarios

**The Problem:**
Integration only tested with "happy path" scenarios, error handling never validated.

**Example Scenario:**
```
Testing: All cases process successfully ✓

Production scenarios never tested:
  - GetCase timeout
  - Database unavailable
  - Missing required fields
  - Invalid case type
  - Network failures
  
Production: First error → System crashes/hangs
```

**The Solution:**
Test error scenarios:
- GetCase timeout handling
- Missing data handling
- Invalid input handling
- Network failure recovery
- Database unavailable scenarios

**How to Identify:**
- Errors in production cause system instability
- No graceful error handling
- Crashes on unexpected input
- Poor error recovery

---

## Testing Mistakes

### ❌ Mistake: Only Testing Simple Cases

**The Problem:**
Testing with simple 2-party, 3-document cases doesn't reveal real-world complexity issues.

**Example Scenario:**
```
Test cases:
  - 2 parties
  - 1 attorney
  - 5 documents
  - All public
  
Real production cases:
  - 15 parties
  - 8 attorneys
  - 50 documents
  - Mixed security
  - Multiple cross-references
  
Result: GetCase times out on real cases
```

**The Solution:**
Test with production-complexity cases:
- Maximum party counts you'll encounter
- Maximum document counts
- Mixed security scenarios
- Complex relationships
- Edge cases from your actual data

**How to Identify:**
- Works in test, fails in production
- Performance problems only in production
- Complex cases cause timeouts

---

### ❌ Mistake: Not Testing All Event Types

**The Problem:**
Only testing common event types (CaseFiling, DocumentFiling) leaves gaps in certification.

**Example Scenario:**
```
Tested: CaseFiling, DocumentFiling ✓

Not tested:
  - CaseSecurity
  - CaseDisposition
  - CaseReassigned
  - CaseDelete
  
Certification: Fails on untested event types
```

**The Solution:**
Test every event type your integration supports before certification. Include market-required events.

**How to Identify:**
- Certification failures
- Event types don't work in production
- Gaps in event handling

---

### ❌ Mistake: No Security Testing

**The Problem:**
Security enforcement never validated before production.

**Critical Security Tests:**
- Sealed case visibility
- Sealed document access
- Mixed security (public case, sealed doc)
- Security change timing
- Role-based access
- Publication delays (if applicable)

**Example Scenario:**
```
Security "works" because:
  - All test cases public
  - Never tested sealed cases
  - Never tested role-based access
  
Production: First sealed case exposed to public
```

**The Solution:**
Dedicate significant testing time to security scenarios. Test all security combinations.

**How to Identify:**
- Security vulnerabilities in production
- Confidential information exposed
- Certification failures on security

---

### ❌ Mistake: Not Testing Market-Specific Rules

**The Problem:**
Market-specific requirements (JCIT delays, AOIC rules) not validated.

**Example Scenario (Texas):**
```
Tested: Case filing and indexing ✓

Not tested:
  - 31-day publication delay
  - Criminal case visibility
  - Registered user access
  - 180-day seal period
  
Production: Texas criminal cases visible immediately ✗
```

**The Solution:**
Understand and test your market's specific requirements. Consult market standards documentation.

**How to Identify:**
- Non-compliant with market standards
- Certification failures
- Users report incorrect behavior for case types

---

## Performance Mistakes

### ❌ Mistake: No Database Indexes

**The Problem:**
Missing database indexes cause slow GetCase queries as data volume grows.

**Example Scenario:**
```
Development: 1,000 cases
  GetCase: 200ms (seems fine)

Production: 100,000 cases
  GetCase: 15 seconds (timeouts)
  
Problem: No index on case_id lookups
```

**The Solution:**
Create appropriate indexes:
- Case lookup by tracking ID
- Party/attorney lookup by case_id
- Document lookup by case_id
- Docket entries by case_id

**How to Identify:**
- Performance degrades as data grows
- Database CPU high
- Slow query logs show missing indexes

---

### ❌ Mistake: GetCase Times Out on Complex Cases

**The Problem:**
GetCase works for average cases but times out on complex ones.

**The Solution:**
- Optimize queries for worst-case complexity
- Test with production's most complex cases
- Set appropriate timeout expectations (< 30 seconds)
- Return error if truly can't respond in time

**How to Identify:**
- Timeout errors for specific cases
- Complex cases never index
- GetCase works for simple cases only

---

## Related Documentation

### For Understanding re:Search Behavior
→ [Architecture Overview](./architecture-overview.md)  
→ [Event Type Logic](./event-type-logic.md)  
→ [Security Logic](./security-logic.md)

### For Integration Success
→ [Best Practices](../integration-guide/best-practices.md)  
→ [Integration Guide](../integration-guide/README.md)

### For Troubleshooting
→ [Troubleshooting Guide](../troubleshooting/README.md)

### For Examples
→ [Real-World Examples](./real-world-examples.md)

---

**Last Updated:** January 2025