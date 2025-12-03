# Architecture Overview

> **Breadcrumb:** [Home](../README.md) > [Implementation Playbook](./README.md) > Architecture Overview

Understand how re:Search components work together to process case data from your CMS and make it available for public search.

---

## System Components

### Your CMS (Case Management System)
**Role:** Authoritative source of case data

**Responsibilities:**
- Maintain official case records
- Generate case events when data changes
- Provide case data on request (GetCase)
- Provide document PDFs on request (GetDocument)

**Integration Points:**
- Sends NotifyCaseEvent to notify of changes
- Receives GetCase requests from re:Search
- Receives GetDocument requests from re:Search

---

### EFM (Electronic Filing Manager)
**Role:** E-filing gateway and event router (ECF mode only)

**Responsibilities:**
- Accept e-filing submissions from EFSPs
- Route filings to appropriate CMS (RecordFiling)
- Forward NotifyCaseEvent to re:Search
- Coordinate filing workflow

**Integration Points:**
- Receives NotifyCaseEvent from CMS
- Forwards events to re:Search
- Sends RecordFiling to CMS

**Note:** CIP mode bypasses EFM and connects directly to re:Search

---

### CIP (Court Integration Platform)
**Role:** Middleware for legacy systems (CIP mode only)

**Responsibilities:**
- Translate between CMS and re:Search formats
- Expose SOAP endpoints on behalf of CMS
- Generate events from CMS data changes
- Handle authentication and protocol conversion

**Integration Points:**
- Monitors CMS for changes (database, files, etc.)
- Sends NotifyCaseEvent to re:Search
- Provides GetCase/GetDocument endpoints

**Note:** Only used in CIP mode integrations

---

### re:Search Platform
**Role:** Centralized case data repository and public search

**Responsibilities:**
- Receive case event notifications
- Request current case data from CMS
- Index cases for fast searching
- Enforce security and access rules
- Provide public search interface

**Integration Points:**
- Receives NotifyCaseEvent from CMS/EFM/CIP
- Calls GetCase on CMS/CIP
- Calls GetDocument on CMS/CIP

---

## Data Flow by Integration Mode

### ECF Mode Flow
```
┌─────────────┐
│     CMS     │
│  (Odyssey,  │
│  3rd Party) │
└──────┬──────┘
       │
       │ 1. NotifyCaseEvent
       ↓
┌─────────────┐
│     EFM     │
│ (File &     │
│  Serve)     │
└──────┬──────┘
       │
       │ 2. Forward Event
       ↓
┌─────────────┐
│  re:Search  │
│  Platform   │
└──────┬──────┘
       │
       │ 3. GetCase Request
       ↓
┌─────────────┐
│     CMS     │
│  Returns    │
│  Case Data  │
└─────────────┘
       │
       │ 4. Case Data Response
       ↓
┌─────────────┐
│  re:Search  │
│  Indexes &  │
│  Publishes  │
└─────────────┘
```

**Key Characteristics:**
- CMS integrated with EFM for e-filing
- Events route through EFM
- Real-time synchronization
- Most common integration pattern

**Typical Use Case:** Courts with active e-filing programs

---

### CIP Mode Flow
```
┌─────────────┐
│     CMS     │
│ (Enterprise │
│  Justice)   │
└──────┬──────┘
       │
       │ 1. Data Changes
       ↓
┌─────────────┐
│     CIP     │
│ Middleware  │
└──────┬──────┘
       │
       │ 2. NotifyCaseEvent
       ↓
┌─────────────┐
│  re:Search  │
│  Platform   │
└──────┬──────┘
       │
       │ 3. GetCase Request
       ↓
┌─────────────┐
│     CIP     │
│  Translates │
│  & Returns  │
└─────────────┘
       │
       │ 4. Case Data Response
       ↓
┌─────────────┐
│  re:Search  │
│  Indexes &  │
│  Publishes  │
└─────────────┘
```

**Key Characteristics:**
- CIP monitors CMS for changes
- CIP handles SOAP translation
- May not require CMS code changes
- Real-time synchronization

**Typical Use Case:** Courts with Enterprise Justice systems

---

### Batch Mode Flow
```
┌─────────────┐
│     CMS     │
│  Generates  │
│  XML Files  │
└──────┬──────┘
       │
       │ 1. Scheduled Export
       ↓
┌─────────────┐
│   S3/SFTP   │
│   Transfer  │
└──────┬──────┘
       │
       │ 2. File Pickup
       ↓
┌─────────────┐
│  re:Search  │
│   Parses    │
│    Files    │
└──────┬──────┘
       │
       │ 3. Batch Index
       ↓
┌─────────────┐
│  re:Search  │
│  Publishes  │
│   Updates   │
└─────────────┘
```

**Key Characteristics:**
- No real-time events
- Scheduled file generation
- Complete case data in files
- Simpler CMS requirements

**Typical Use Case:** Courts with limited API capabilities or large case volumes

---

## Event-Driven Processing

### How Events Trigger Updates

**1. Change Occurs in CMS**
```
User updates case in CMS → Transaction commits → Trigger fires
```

**2. Event Generation**
```
Trigger → Generate NotifyCaseEvent XML → Send to EFM/re:Search
```

**3. Event Routing**
```
EFM receives event → Validates → Forwards to re:Search
```

**4. Data Request**
```
re:Search receives event → Calls CMS GetCase → Retrieves current data
```

**5. Indexing**
```
re:Search receives case data → Applies security rules → Indexes for search
```

**6. Publication**
```
Index complete → Apply publication delays → Make searchable
```

---

## Security Processing Pipeline

### How Security is Enforced

**Step 1: Case-Level Security (from CMS)**
```
CMS sets case security → Sent in GetCase response → re:Search applies rules
```

**Possible Values:**
- `Public` - Visible to everyone (default)
- `Sealed` - Restricted access, may have delays
- `Confidential` - Limited visibility by role

**Step 2: Document-Level Security (from CMS)**
```
CMS sets document security → Sent in GetCase response → Overrides case security
```

**Document Security Can:**
- Make individual documents sealed even if case is public
- Restrict documents beyond case-level restrictions
- Never make documents MORE visible than case allows

**Step 3: Market-Specific Rules (from re:Search)**
```
re:Search applies market rules → Publication delays → Role-based access
```

**Market Rules Include:**
- Publication delay periods (31-day, 180-day)
- Case type-specific access rules
- Role-based visibility matrix
- Registered user requirements

**Step 4: User Access Check (at search time)**
```
User searches → re:Search checks user role → Filters results → Returns allowed cases
```

[See detailed security logic →](./security-logic.md)

---

## Indexing Pipeline

### How Cases Become Searchable

**1. Event Received**
```
NotifyCaseEvent arrives → Validated → Queued for processing
```

**2. Data Retrieval**
```
GetCase called → CMS responds → XML parsed and validated
```

**3. Data Normalization**
```
CMS-specific formats → Standardized fields → Market mapping applied
```

**4. Security Evaluation**
```
Case security checked → Document security applied → Access rules determined
```

**5. Index Update**
```
Searchable fields extracted → Full-text indexed → Metadata stored
```

**6. Publication Check**
```
Delays calculated → Publication date set → Case made searchable (or delayed)
```

---

## Component Interaction Patterns

### Synchronous Operations

**GetCase / GetDocument Requests**
```
re:Search → CMS (request)
          ← CMS (immediate response)
```

**Characteristics:**
- Real-time HTTP/SOAP call
- CMS must respond within timeout (typically 30 seconds)
- Response expected immediately
- Failures result in retries

**Performance Considerations:**
- Optimize database queries
- Cache responses when appropriate
- Handle timeouts gracefully
- Monitor response times

---

### Asynchronous Operations

**NotifyCaseEvent**
```
CMS → EFM/re:Search (fire and forget)
```

**Characteristics:**
- CMS doesn't wait for processing completion
- Event queued for processing
- Eventual consistency
- Failures handled by retry logic

**Design Implications:**
- Events can arrive out of order
- Multiple events may be consolidated
- Processing is eventually consistent
- CMS should log event transmission

---

## Scalability Patterns

### Event Throttling
**Problem:** CMS generates events faster than re:Search can process

**Solution:**
- re:Search queues events
- Processes in order
- Consolidates duplicate events for same case
- CMS doesn't need to throttle

---

### GetCase Optimization
**Problem:** Expensive database queries for case data

**Solutions:**
- **Caching:** Cache GetCase responses with short TTL (5-10 min)
- **Indexing:** Ensure CMS database properly indexed
- **Lazy Loading:** Don't retrieve documents until GetDocument called
- **Pagination:** For large docket entries, consider pagination

---

### Document Retrieval
**Problem:** Large PDFs slow down GetDocument

**Solutions:**
- **Streaming:** Stream documents rather than loading entirely into memory
- **Size Limits:** Document reasonable size limits (e.g., 25MB)
- **Compression:** Consider compression for transmission
- **Caching:** Cache frequently accessed documents

---

## Error Handling Patterns

### Event Delivery Failures

**Scenario:** NotifyCaseEvent fails to reach re:Search

**CMS Responsibilities:**
- Log event transmission attempts
- Implement retry logic (3-5 attempts)
- Alert on persistent failures
- Provide manual re-sync mechanism

**re:Search Behavior:**
- Accepts duplicate events safely
- Consolidates multiple events for same case
- Eventually consistent - later successful event brings data current

---

### GetCase Failures

**Scenario:** re:Search can't retrieve case data

**CMS Responsibilities:**
- Return appropriate SOAP fault
- Log the failure reason
- Handle timeout scenarios
- Provide meaningful error messages

**re:Search Behavior:**
- Retries failed GetCase calls (exponential backoff)
- Keeps case in "pending" state
- Alerts on persistent failures
- Maintains last known good data

---

### GetDocument Failures

**Scenario:** Document not available or access denied

**CMS Responsibilities:**
- Check document availability
- Verify security/access rules
- Return appropriate error code
- Log access attempts

**re:Search Behavior:**
- Shows "document unavailable" to user
- Doesn't retry indefinitely
- Logs for troubleshooting
- Handles gracefully in UI

---

## Monitoring and Observability

### What to Monitor

**Event Generation (CMS)**
- Events sent per hour/day
- Event transmission failures
- Event types distribution
- Time from case change to event sent

**API Performance (CMS)**
- GetCase response times
- GetDocument response times
- Error rates
- Timeout occurrences

**Integration Health (Overall)**
- End-to-end latency (change to searchable)
- Failed case updates
- Stale cases (not updated recently)
- Security mismatches

---

## Common Architecture Mistakes

### ❌ Mistake: Generating events before transaction commit
**Problem:** Event sent but database rollback means data inconsistent

**Solution:** Generate events AFTER successful commit

---

### ❌ Mistake: GetCase returns stale cached data
**Problem:** re:Search indexes outdated case information

**Solution:** Use short cache TTL or invalidate cache on change

---

### ❌ Mistake: No retry logic for failed events
**Problem:** Missed events mean cases not updated in re:Search

**Solution:** Implement retry with exponential backoff

---

### ❌ Mistake: GetCase/GetDocument don't enforce security
**Problem:** Sealed cases/documents exposed inappropriately

**Solution:** Always check security before returning data

---

### ❌ Mistake: No monitoring of integration health
**Problem:** Silent failures go unnoticed for days/weeks

**Solution:** Implement comprehensive monitoring and alerting

---

## Related Documentation

### For API Specifications
→ [API Reference](../api-reference/README.md) - Detailed endpoint documentation

### For Security Implementation
→ [Security Logic](./security-logic.md) - How to enforce access rules

### For Event Handling
→ [Event Type Logic](./event-type-logic.md) - When to send which events

### For Best Practices
→ [Best Practices](./best-practices.md) - Proven implementation patterns

---

**Last Updated:** January 2025