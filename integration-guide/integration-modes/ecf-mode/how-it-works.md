# How ECF Mode Works

[← Back to ECF Mode Overview](./README.md)

## Quick Navigation
- [The Two Workflows](#the-two-workflows)
- [E-Filing Workflow](#e-filing-workflow-via-efm)
- [Direct Case Updates](#direct-case-updates-workflow)
- [Key Integration Points](#key-integration-points)
- [What Gets Sent vs. Retrieved](#what-gets-sent-vs-retrieved)

---

## The Two Workflows

ECF Mode operates through two distinct workflows that work together to keep re:Search synchronized with your CMS.

### Overview

**1. E-Filing Workflow (via EFM)**
- Handles filings submitted through Electronic Filing Service Providers (EFSPs)
- Flows through Tyler's Electronic Filing Manager (EFM)
- Coordinates between attorney, EFM, CMS, and re:Search

**2. Direct Case Updates Workflow**
- Handles changes that occur outside e-filing
- Direct communication from CMS to re:Search
- Examples: party changes, security updates, dispositions

Both workflows share a common pattern: **notify then retrieve**
- CMS (or EFM) sends notification: "Something changed"
- re:Search retrieves current data: "Show me the current state"

---

## E-Filing Workflow (via EFM)

When an attorney files a document electronically, the filing flows through multiple systems before appearing in re:Search.

### Complete Flow Diagram

```
┌──────────┐         ┌─────────┐         ┌─────────┐         ┌──────────┐
│ Attorney │         │   EFM   │         │   CMS   │         │re:Search │
│  (EFSP)  │         │         │         │         │         │          │
└──────────┘         └─────────┘         └─────────┘         └──────────┘
      │                    │                    │                    │
      │ 1. Submit filing   │                    │                    │
      ├───────────────────►│                    │                    │
      │                    │                    │                    │
      │                    │ 2. RecordFiling    │                    │
      │                    ├───────────────────►│                    │
      │                    │                    │                    │
      │                    │                    │ 3. Accept filing   │
      │                    │                    │    Return receipt  │
      │                    │◄───────────────────┤                    │
      │                    │                    │                    │
      │ 4. Filing receipt  │                    │                    │
      │◄───────────────────┤                    │                    │
      │                    │                    │                    │
      │                    │ 5. Notify: Filing received             │
      │                    ├───────────────────────────────────────►│
      │                    │                    │                    │
      │                    │                    │ 6. GetCase (pre)   │
      │                    │                    │◄───────────────────┤
      │                    │                    │                    │
      │                    │                    │ 7. Case data       │
      │                    │                    ├───────────────────►│
      │                    │                    │                    │
      │                    │                    │ 8. Index pre-      │
      │                    │                    │    docket data     │
      │                    │                    │                    │
      │                    │    9. CMS dockets the filing            │
      │                    │       (background process)              │
      │                    │                    │                    │
      │                    │ 10. NotifyDocketingComplete            │
      │                    │◄───────────────────┤                    │
      │                    │                    │                    │
      │                    │ 11. Notify: Docketed                   │
      │                    ├───────────────────────────────────────►│
      │                    │                    │                    │
      │                    │                    │ 12. GetCase (final)│
      │                    │                    │◄───────────────────┤
      │                    │                    │                    │
      │                    │                    │ 13. Case data      │
      │                    │                    ├───────────────────►│
      │                    │                    │                    │
      │                    │                    │ 14. Index docketed │
      │                    │                    │     filing         │
```

### Step-by-Step Breakdown

**Phase 1: Filing Submission (Steps 1-4)**

1. **Attorney submits filing through EFSP**
   - Attorney uses electronic filing service provider
   - Submits documents and metadata
   - Pays filing fees

2. **EFM validates and routes to CMS (RecordFiling)**
   - EFM validates filing package
   - Routes to appropriate court CMS
   - Sends RecordFiling SOAP request

3. **CMS accepts filing, returns receipt**
   - CMS validates filing package
   - Stores documents
   - Returns filing receipt to EFM
   - Does NOT docket yet (docketing happens later)

4. **Attorney receives filing receipt**
   - EFM sends receipt to attorney
   - Filing confirmed accepted (but not docketed)

**Phase 2: Pre-Docket Indexing (Steps 5-8)**

5. **EFM notifies re:Search: "Filing received"**
   - EFM sends notification to re:Search
   - Indicates filing accepted but not docketed
   - Contains case ID and filing details

6. **re:Search calls CMS GetCase (pre-docket)**
   - re:Search requests current case state
   - Captures preliminary filing information
   - Gets case metadata before docketing

7. **CMS returns case data**
   - Includes preliminary filing details
   - Shows filing as "pending docketing"
   - Returns current case state

8. **re:Search indexes pre-docket data**
   - Filing visible as "pending" in re:Search
   - Allows tracking of undocketed filings
   - Useful for court staff monitoring

**Phase 3: Docketing (Step 9)**

9. **CMS dockets the filing** (background process)
   - Clerk reviews and approves filing
   - Filing assigned docket number/sequence
   - Document officially added to case record
   - Timing varies: minutes to hours after receipt

**Phase 4: Final Indexing (Steps 10-14)**

10. **CMS sends NotifyDocketingComplete to EFM**
    - CMS confirms docketing completed
    - Triggers final notification chain

11. **EFM notifies re:Search: "Filing docketed"**
    - EFM forwards notification to re:Search
    - Indicates filing officially docketed
    - Triggers final data refresh

12. **re:Search calls CMS GetCase (final)**
    - re:Search requests updated case state
    - Gets fully docketed filing information
    - Retrieves complete metadata

13. **CMS returns case data**
    - Includes fully docketed filing
    - Complete docket entry with sequence
    - Final filing metadata

14. **re:Search indexes docketed filing**
    - Filing now shows as officially docketed
    - Complete filing information visible
    - Public can search and view (if not sealed)

### Why Two GetCase Calls?

**First GetCase (Pre-Docket):**
- Captures that filing was received
- Allows tracking of pending filings
- Useful for court staff monitoring workflow
- Filing shows as "pending docketing"

**Second GetCase (Final):**
- Retrieves fully docketed filing
- Includes official docket sequence
- Complete metadata available
- Filing shows as officially part of record

**Best Practice:** Both calls should return current data from CMS - don't cache GetCase responses.

---

## Direct Case Updates Workflow

For changes that occur outside the e-filing workflow, your CMS notifies re:Search directly.

### Complete Flow Diagram

```
┌─────────┐                           ┌──────────┐
│   CMS   │                           │re:Search │
│         │                           │          │
└─────────┘                           └──────────┘
      │                                     │
      │ 1. Change occurs in CMS             │
      │    (party added, case sealed, etc.) │
      │                                     │
      │ 2. NotifyCaseEvent                  │
      ├────────────────────────────────────►│
      │    {caseId, eventType}              │
      │                                     │
      │                                     │ 3. Process notification
      │                                     │    Determine what changed
      │                                     │
      │                        4. GetCase   │
      │◄────────────────────────────────────┤
      │                                     │
      │ 5. Return complete current data     │
      ├────────────────────────────────────►│
      │                                     │
      │                                     │ 6. Index updated case
      │                                     │    Apply changes
      │                                     │
      │                                     │ 7. Data searchable
```

### Step-by-Step Breakdown

**1. Change occurs in CMS**

Common triggers:
- Party or attorney added/changed/removed
- Case security changed (sealed/unsealed)
- Disposition entered
- Judge or division reassigned
- Document added outside e-filing
- Document security changed
- Case status changed

**2. CMS sends NotifyCaseEvent to re:Search**

Message contains:
- `CaseTrackingID`: Case identifier
- `EventType`: What changed (e.g., "CaseParty", "CaseSecurity")

**Example:**
```xml
<NotifyCaseEvent>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <EventType>CaseParty</EventType>
</NotifyCaseEvent>
```

**Critical:** Send event IMMEDIATELY after database commit, especially for security events.

**3. re:Search processes notification**
- Receives event
- Logs event for monitoring
- Determines action based on event type
- Prepares to retrieve fresh data

**4. re:Search calls CMS GetCase**
- Requests complete current case data
- Uses CaseTrackingID from event
- Expects current data, not cached

**5. CMS returns complete current data**

Must include:
- All case metadata
- All parties with roles and contact info
- All attorneys with associations
- All documents with metadata
- Current security settings
- Docket entries
- Case events/hearings

**6. re:Search indexes updated case**
- Parses returned data
- Updates search index
- Applies security filters
- Makes changes searchable

**7. Data becomes searchable**
- Changes visible in re:Search portal
- Security filters enforced
- Search results updated

### Event Type Examples

| Change in CMS | Event Type to Send | What Gets Updated |
|---------------|-------------------|-------------------|
| Party added/changed | `CaseParty` | Party list and associations |
| Attorney added/changed | `CasePartyAttorney` | Attorney list and party associations |
| Case sealed/unsealed | `CaseSecurity` | Security level and access controls |
| Document sealed | `DocumentSecurity` | Individual document access |
| Disposition entered | `CaseDisposition` | Disposition data and case status |
| Judge changed | `CaseReassigned` | Judge assignment |
| Document added (non-filing) | `DocumentFiling` | Document list |
| Hearing scheduled | `CaseHearing` | Hearing/event list |

[Complete event type reference →](../../implementation-playbook/event-type-logic.md)

---

## Key Integration Points

### Notifications: Small and Fast

**What CMS sends:**
- Just enough to identify what changed
- Case ID + Event Type
- Small XML messages (< 1 KB typically)
- Fast to generate and transmit

**What CMS does NOT send:**
- Full case data
- Document binaries
- Complete party lists
- Detailed docket entries

**Why this approach?**
- Lightweight notifications
- Single source of truth (GetCase always returns current data)
- Simpler to implement (don't duplicate data in notifications)
- More reliable (always get fresh data from CMS)

### Data Retrieval: Complete and Current

**What re:Search retrieves via GetCase:**
- Complete case metadata
- All parties and attorneys
- All documents (metadata, not binaries)
- Current security settings
- Docket entries
- Case events

**Requirements:**
- Must be current (not cached)
- Must be complete (all parties, not just changed party)
- Must be fast (< 3 seconds for typical cases)
- Must be accurate (security settings critical)

**Why retrieve everything, not just what changed?**
- Simpler logic (no need to calculate diffs)
- More reliable (always consistent)
- Easier to debug (can compare full state)
- Handles complex multi-field changes

---

## What Gets Sent vs. Retrieved

### CMS → re:Search (NotifyCaseEvent)

**Sends:**
```xml
<NotifyCaseEvent>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <EventType>CaseParty</EventType>
</NotifyCaseEvent>
```

**Size:** < 1 KB  
**Frequency:** Every time case changes  
**Purpose:** Trigger re:Search to refresh

### re:Search → CMS (GetCase)

**Requests:**
```xml
<GetCaseRequest>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
</GetCaseRequest>
```

**Returns:**
```xml
<GetCaseResponse>
  <Case>
    <CaseNumber>CV-2024-001234</CaseNumber>
    <CaseTitle>Smith v. Jones</CaseTitle>
    <FilingDate>2024-03-15</FilingDate>
    <CaseType>Civil</CaseType>
    <CaseStatus>Active</CaseStatus>
    <Security>Public</Security>
    <Parties>
      <Party>
        <Name>John Smith</Name>
        <Role>Plaintiff</Role>
        ...
      </Party>
      <Party>
        <Name>Jane Jones</Name>
        <Role>Defendant</Role>
        ...
      </Party>
    </Parties>
    <Attorneys>
      <Attorney>
        <Name>Robert Attorney</Name>
        <BarNumber>12345678</BarNumber>
        <RepresentsParty>John Smith</RepresentsParty>
        ...
      </Attorney>
    </Attorneys>
    <Documents>
      <Document>
        <DocumentID>DOC-001</DocumentID>
        <DocumentTitle>Complaint</DocumentTitle>
        <FilingDate>2024-03-15</FilingDate>
        <Security>Public</Security>
        ...
      </Document>
    </Documents>
    ...
  </Case>
</GetCaseResponse>
```

**Size:** Varies (1 KB - 1 MB depending on case complexity)  
**Frequency:** Only when triggered by event  
**Purpose:** Provide complete current case data

---

## Performance Characteristics

### Typical Latency

| Metric | Target | Notes |
|--------|--------|-------|
| **NotifyCaseEvent delivery** | < 1 second | From CMS to re:Search |
| **GetCase response** | < 2 seconds | Average cases |
| **GetCase response** | < 3 seconds | Complex cases (many parties/docs) |
| **End-to-end update** | < 30 seconds | From CMS change to searchable in re:Search |

### Volume Expectations

| Scenario | Volume | Handling |
|----------|--------|----------|
| **E-filing (busy day)** | 100-500 filings | Each triggers 2 GetCase calls |
| **Case updates (typical)** | 50-200 events/day | Each triggers 1 GetCase call |
| **Peak periods** | 10-20 events/minute | System handles bursts |

### Optimization Tips

**For NotifyCaseEvent:**
- Send immediately after commit (don't batch)
- Keep payload minimal (just ID + event type)
- Implement async sending (don't block CMS transaction)
- Use message queue if sending many events

**For GetCase:**
- Optimize database queries (avoid N+1 problems)
- Use proper indexes on case lookups
- Query once, not per party/attorney/document
- Don't cache responses (always return current data)
- Consider read replicas for high load (ensure low lag)

---

## Error Handling

### Notification Failures

**If NotifyCaseEvent fails to send:**
- Log error in CMS
- Implement retry logic (exponential backoff)
- Alert operations team
- Don't let failure block CMS operations
- Consider message queue for reliability

**Best practice:** Separate event publishing from CMS transaction - use async messaging.

### GetCase Failures

**If CMS can't respond to GetCase:**
- re:Search will retry (3 attempts typical)
- Return proper HTTP error codes
- Log failure for troubleshooting
- Alert if failures exceed threshold
- Ensure retry doesn't cause duplicate work

**Best practice:** Monitor GetCase response times and error rates closely.

---

## Next Steps

- **Review technical requirements** → Read [Technical Setup](./technical-setup.md)
- **Plan your implementation** → Read [Implementation Guide](./implementation-guide.md)
- **Learn about operations** → Read [Operations](./operations.md)
- **Return to overview** → [ECF Mode Overview](./README.md)

---

**Last Updated:** December 2025