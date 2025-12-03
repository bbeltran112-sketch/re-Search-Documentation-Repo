# Event Type Logic

> **Breadcrumb:** [Home](../README.md) > [Implementation Playbook](./README.md) > Event Type Logic

Understand when to send each NotifyCaseEvent type and how re:Search processes them. Proper event type usage is critical for accurate case synchronization.

---

## Overview

NotifyCaseEvent tells re:Search that something changed in your CMS. The `EventType` field indicates what kind of change occurred, which helps re:Search:
- Determine what data to retrieve
- Apply appropriate processing logic
- Optimize indexing operations
- Provide audit trails

**Golden Rule:** Send an event whenever case data changes that should be reflected in re:Search.

---

## Core Event Types

### CaseFiling
**When to send:** A new case is filed or initiated

**Triggers:**
- New case created in CMS
- Case number assigned
- Initial case data entered
- E-filing accepted and docketed

**What re:Search does:**
- Calls GetCase to retrieve complete case data
- Creates new case in index
- Applies initial security rules
- Makes case searchable (subject to publication delays)

**XML Example:**
```xml
<NotifyCaseEvent>
  <EventType>CaseFiling</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <Court>
    <CourtID>250</CourtID>
  </Court>
</NotifyCaseEvent>
```

**Common Mistakes:**
- ❌ Sending CaseFiling for case updates (use CaseParty or CaseInformation instead)
- ❌ Not sending CaseFiling when case first created
- ❌ Sending multiple CaseFiling events for the same case

[See CaseFiling examples →](../api-reference/notifycaseevent/examples/case-filing.xml)

---

### CaseParty
**When to send:** Party information changes

**Triggers:**
- Party added to case
- Party name corrected
- Party address updated
- Party removed from case
- Party role changed

**What re:Search does:**
- Calls GetCase to get updated party list
- Updates party index for searching
- Updates case last modified date
- Triggers re-indexing of party names

**XML Example:**
```xml
<NotifyCaseEvent>
  <EventType>CaseParty</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <Court>
    <CourtID>250</CourtID>
  </Court>
</NotifyCaseEvent>
```

**Common Mistakes:**
- ❌ Not sending event when party information changes
- ❌ Sending CasePartyAttorney when you should send CaseParty
- ❌ Forgetting to send when party is removed

[See CaseParty examples →](../api-reference/notifycaseevent/examples/case-party.xml)

---

### CasePartyAttorney
**When to send:** Attorney representation changes

**Triggers:**
- Attorney added to represent party
- Attorney information updated
- Attorney removed from case
- Lead attorney designation changed
- Attorney contact information updated

**What re:Search does:**
- Calls GetCase to get updated attorney list
- Updates attorney index for searching
- Adjusts attorney access permissions
- Updates case metadata

**XML Example:**
```xml
<NotifyCaseEvent>
  <EventType>CasePartyAttorney</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <Court>
    <CourtID>250</CourtID>
  </Court>
</NotifyCaseEvent>
```

**Important:** Use this for attorney changes, not general party changes.

**Common Mistakes:**
- ❌ Using CaseParty when attorney information changes
- ❌ Not sending when attorney is removed
- ❌ Forgetting to update when attorney contact info changes

[See CasePartyAttorney examples →](../api-reference/notifycaseevent/examples/case-party-attorney.xml)

---

### DocumentFiling
**When to send:** New document added to case

**Triggers:**
- Document filed electronically
- Document scanned and added to case
- Document manually entered
- E-filing accepted and docketed

**What re:Search does:**
- Calls GetCase to get updated document list
- Indexes new document metadata
- Applies document-level security
- Makes document searchable/viewable

**XML Example:**
```xml
<NotifyCaseEvent>
  <EventType>DocumentFiling</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <Court>
    <CourtID>250</CourtID>
  </Court>
</NotifyCaseEvent>
```

**Note:** You don't need to specify which document - re:Search calls GetCase and sees all documents.

**Common Mistakes:**
- ❌ Not sending event when document added manually
- ❌ Sending separate event for each document (one event per case change is sufficient)
- ❌ Forgetting to include document security in GetCase response

[See DocumentFiling examples →](../api-reference/notifycaseevent/examples/document-filing.xml)

---

### CaseDisposition
**When to send:** Case reaches final disposition

**Triggers:**
- Case closed/disposed
- Judgment entered
- Dismissal entered
- Settlement finalized
- Case archived

**What re:Search does:**
- Calls GetCase to get disposition details
- Updates case status to disposed
- Updates case closure date
- May trigger archive workflows

**XML Example:**
```xml
<NotifyCaseEvent>
  <EventType>CaseDisposition</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <Court>
    <CourtID>250</CourtID>
  </Court>
</NotifyCaseEvent>
```

**Important for Criminal Cases:** Many jurisdictions require CaseDisposition events for proper visibility handling.

**Common Mistakes:**
- ❌ Not sending when case is disposed
- ❌ Sending when case is temporarily stayed (not final disposition)
- ❌ Not including disposition information in GetCase response

[See CaseDisposition examples →](../api-reference/notifycaseevent/examples/case-disposition.xml)

---

## Security-Related Event Types

### CaseSecurity
**When to send:** Case-level security changes

**Triggers:**
- Case sealed or unsealed
- Case confidentiality changed
- Public access restrictions added/removed
- Case security level modified

**What re:Search does:**
- Calls GetCase to get new security settings
- Re-evaluates case visibility
- Updates access permissions
- May hide/show case immediately

**XML Example:**
```xml
<NotifyCaseEvent>
  <EventType>CaseSecurity</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <Court>
    <CourtID>250</CourtID>
  </Court>
</NotifyCaseEvent>
```

**Critical:** This event type requires immediate processing. Send it whenever security changes.

**Common Mistakes:**
- ❌ Not sending when case is sealed/unsealed
- ❌ Delaying event transmission for security changes
- ❌ Not including updated security in GetCase response

[See CaseSecurity examples →](../api-reference/notifycaseevent/examples/case-security.xml)

---

### DocumentSecurity
**When to send:** Document-level security changes

**Triggers:**
- Document sealed or unsealed
- Document confidentiality changed
- Document access restrictions modified
- Document security level adjusted

**What re:Search does:**
- Calls GetCase to get updated document security
- Re-evaluates document visibility
- Updates document access permissions
- Hides/shows document appropriately

**XML Example:**
```xml
<NotifyCaseEvent>
  <EventType>DocumentSecurity</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <Court>
    <CourtID>250</CourtID>
  </Court>
</NotifyCaseEvent>
```

**Note:** Document security can be more restrictive than case security, but never less restrictive.

**Common Mistakes:**
- ❌ Not sending when document security changes
- ❌ Making document security less restrictive than case security
- ❌ Not including document security metadata in GetCase

[See DocumentSecurity examples →](../api-reference/notifycaseevent/examples/document-security.xml)

---

## Administrative Event Types

### CaseReassigned
**When to send:** Case assigned to different judge/division/location

**Triggers:**
- Judge reassignment
- Case transferred to different division
- Venue change
- Judicial assignment updated

**What re:Search does:**
- Calls GetCase to get new assignment information
- Updates case metadata
- Adjusts case categorization
- Updates search filters

**XML Example:**
```xml
<NotifyCaseEvent>
  <EventType>CaseReassigned</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <Court>
    <CourtID>250</CourtID>
  </Court>
</NotifyCaseEvent>
```

**Common Mistakes:**
- ❌ Not sending when judge changes
- ❌ Not updating GetCase response with new assignment

[See CaseReassigned examples →](../api-reference/notifycaseevent/examples/case-reassigned.xml)

---

### CaseDelete
**When to send:** Case is deleted or expunged

**Triggers:**
- Case expunged by court order
- Case deleted due to data entry error
- Case number voided and reassigned
- Case record removed from system

**What re:Search does:**
- Removes case from index
- Deletes all associated documents
- Removes from search results
- Purges case data

**XML Example:**
```xml
<NotifyCaseEvent>
  <EventType>CaseDelete</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <Court>
    <CourtID>250</CourtID>
  </Court>
</NotifyCaseEvent>
```

**Warning:** This is permanent and immediate. Use carefully.

**Common Mistakes:**
- ❌ Using CaseDelete when case should be sealed instead
- ❌ Not sending CaseDelete for expunged cases
- ❌ Sending CaseDelete for case closure (use CaseDisposition instead)

[See CaseDelete examples →](../api-reference/notifycaseevent/examples/case-delete.xml)

---

### CaseExpunge
**When to send:** Case is expunged by court order (alternative to CaseDelete)

**Triggers:**
- Court order to expunge
- Expungement granted
- Case record sealed under expungement statute

**What re:Search does:**
- Similar to CaseDelete
- May retain audit trail
- Removes from public access
- Follows jurisdiction-specific expungement rules

**XML Example:**
```xml
<NotifyCaseEvent>
  <EventType>CaseExpunge</EventType>
  <CaseTrackingID>CR-2024-005678</CaseTrackingID>
  <Court>
    <CourtID>250</CourtID>
  </Court>
</NotifyCaseEvent>
```

**Market-Specific:** Check your [market standards](../market-standards/README.md) for expungement requirements.

[See CaseExpunge examples →](../api-reference/notifycaseevent/examples/case-expunge.xml)

---

## Event Processing Rules

### Event Consolidation

**Scenario:** Multiple events sent rapidly for the same case

**re:Search Behavior:**
```
Events Sent:
  10:00:00 - CaseParty
  10:00:15 - DocumentFiling
  10:00:30 - CasePartyAttorney

re:Search Processing:
  10:01:00 - Processes all three events together
  10:01:05 - Calls GetCase once
  10:01:10 - Indexes complete current state
```

**Implication:** You don't need to throttle events. Send them as changes occur.

---

### Event Ordering

**Problem:** Events may arrive out of order

**Example:**
```
Sent:
  Event 1 (10:00) - CaseFiling
  Event 2 (10:05) - CaseParty

Received:
  Event 2 (10:05) - CaseParty (arrives first)
  Event 1 (10:00) - CaseFiling (arrives second)
```

**re:Search Handling:**
- Each event triggers a GetCase call
- GetCase always returns current state
- Final state is always correct
- Event order doesn't affect accuracy

**Design Implication:** Your CMS doesn't need to guarantee event order.

---

### Duplicate Events

**Scenario:** Same event sent multiple times (e.g., retry logic)

**re:Search Behavior:**
```
Receives:
  Event 1 (10:00) - CaseParty for CV-2024-001234
  Event 2 (10:05) - CaseParty for CV-2024-001234 (duplicate)

Processing:
  Consolidates duplicates
  Calls GetCase once
  Indexes current state
```

**Implication:** Safe to retry failed event transmissions.

---

## Event Generation Strategies

### Database Trigger Approach

**Pattern:** Generate events at database level
```sql
CREATE TRIGGER after_case_update
AFTER UPDATE ON cases
FOR EACH ROW
BEGIN
  INSERT INTO event_queue (event_type, case_id, timestamp)
  VALUES ('CaseParty', NEW.case_id, NOW());
END;
```

**Pros:**
- ✅ Never miss a change
- ✅ Consistent event generation
- ✅ Independent of application code
- ✅ Easy to monitor and debug

**Cons:**
- ❌ Less flexibility for business logic
- ❌ May generate unnecessary events
- ❌ Database-specific implementation

**Best for:** Ensuring no updates are missed

---

### Application-Level Approach

**Pattern:** Generate events in application code
```python
def update_case_party(case_id, party_data):
    # Update database
    db.update_party(case_id, party_data)
    db.commit()
    
    # Generate event
    send_event('CaseParty', case_id)
```

**Pros:**
- ✅ Business logic integration
- ✅ Conditional event generation
- ✅ Platform-independent
- ✅ Easier to test

**Cons:**
- ❌ Must instrument all update paths
- ❌ Risk of missing code paths
- ❌ More complex to maintain

**Best for:** Fine-grained control over when events are sent

---

### Hybrid Approach

**Pattern:** Triggers write to queue, application sends events
```
Database Trigger → Event Queue Table → Background Service → Send to re:Search
```

**Pros:**
- ✅ Never miss changes (trigger-based)
- ✅ Controlled transmission (application-based)
- ✅ Can batch and optimize
- ✅ Retry logic built-in

**Cons:**
- ❌ More complex architecture
- ❌ Eventual consistency
- ❌ Requires queue monitoring

**Best for:** High-volume systems needing reliability and optimization

---

## Common Event Scenarios

### Scenario 1: E-Filing Accepted

**What happens:**
1. E-filing arrives at CMS via RecordFiling
2. CMS dockets the filing
3. CMS generates document IDs

**Events to send:**
```
If new case:
  → CaseFiling (new case created)

If existing case:
  → DocumentFiling (new document added)
```

**GetCase must include:**
- New document in document list
- Updated docket entries
- Document security settings

---

### Scenario 2: Attorney Hired

**What happens:**
1. Attorney files appearance
2. CMS adds attorney to case
3. Attorney-party relationship created

**Event to send:**
```
→ CasePartyAttorney
```

**GetCase must include:**
- Attorney in party attorney list
- Correct party association
- Attorney contact information
- Lead attorney designation (if applicable)

---

### Scenario 3: Case Sealed

**What happens:**
1. Judge orders case sealed
2. CMS updates case security
3. Case should disappear from public search

**Event to send:**
```
→ CaseSecurity (CRITICAL - send immediately)
```

**GetCase must include:**
- Updated case security level (Sealed)
- Reason for sealing (if applicable)
- Seal date

**re:Search will:**
- Remove case from public search immediately
- Restrict access per security rules
- Apply any publication delays

---

### Scenario 4: Multiple Changes in Transaction

**What happens:**
1. User updates party name
2. User adds attorney
3. User updates case status
4. All saved in one transaction

**Events to send:**
```
Option 1 (Recommended):
  → CaseParty (covers all changes)

Option 2 (Also valid):
  → CaseParty
  → CasePartyAttorney
  → CaseInformation

Both result in same outcome:
  - re:Search calls GetCase once
  - Gets complete current state
  - Indexes all changes
```

**Design Principle:** When in doubt, send an event. re:Search consolidates.

---

### Scenario 5: Case Disposed

**What happens:**
1. Judge enters final judgment
2. CMS marks case disposed
3. Case closure date set

**Event to send:**
```
→ CaseDisposition
```

**GetCase must include:**
- Case status = Disposed
- Disposition date
- Disposition type
- Disposition details

**Important:** Some markets require CaseDisposition for proper case handling.

---

## Market-Specific Event Requirements

Different markets may have specific event type requirements.

### Texas (JCIT)
**Required events:**
- CaseFiling (all cases)
- CaseDisposition (criminal cases especially)
- CaseSecurity (any security changes)
- DocumentFiling (all document additions)

[See Texas event requirements →](../market-standards/texas/event-types/README.md)

### Illinois (AOIC)
**Required events:**
- Check Illinois-specific requirements
- May differ from Texas

[See Illinois event requirements →](../market-standards/illinois/event-types/README.md)

### Other Markets
**Standard events:**
- CaseFiling
- CaseParty / CasePartyAttorney
- DocumentFiling
- CaseSecurity / DocumentSecurity

[See other market requirements →](../market-standards/other-markets/README.md)

---

## Troubleshooting Event Issues

### Case not appearing in re:Search

**Check:**
1. Was NotifyCaseEvent sent successfully?
2. Did re:Search receive the event?
3. Did GetCase return the case data?
4. Is case security preventing visibility?
5. Are publication delays in effect?

[See troubleshooting guide →](../troubleshooting/event-handling.md)

---

### Case data is stale

**Check:**
1. Are events being sent when data changes?
2. Are events reaching re:Search?
3. Is GetCase returning current data?
4. Is GetCase response being cached too long?

[See troubleshooting guide →](../troubleshooting/common-integration-issues.md)

---

### Too many events being generated

**Check:**
1. Are triggers firing on read operations?
2. Are events being sent for non-material changes?
3. Is event consolidation working?

**Solution:** Review event generation logic, add business logic filters

---

## Best Practices

### ✅ DO:
- Send events immediately after data changes
- Send events after transaction commit
- Include all required fields in NotifyCaseEvent
- Send security-related events without delay
- Log all event transmissions
- Implement retry logic for failures
- Monitor event queue depth

### ❌ DON'T:
- Don't send events before transaction commit
- Don't throttle event sending (re:Search consolidates)
- Don't skip events to "save API calls"
- Don't delay security-related events
- Don't send CaseDelete unless truly deleting
- Don't omit events for "minor" changes
- Don't assume event order matters

---

## Related Documentation

### For API Details
→ [NotifyCaseEvent API Reference](../api-reference/notifycaseevent/README.md)

### For Implementation Guidance
→ [Best Practices](./best-practices.md)

### For Security
→ [Security Logic](./security-logic.md)

### For Market Rules
→ [Market Standards](../market-standards/README.md)

---

**Last Updated:** January 2025