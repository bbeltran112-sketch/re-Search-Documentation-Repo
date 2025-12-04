# NotifyCaseEvent Request & Response Specification

[← Back to NotifyCaseEvent Overview](./README.md)

## Quick Navigation
- [Request Format](#request-format)
- [Response Format](#response-format)
- [Required Fields](#required-fields)
- [Error Handling](#error-handling)
- [Complete Examples](#complete-examples)

---

## Request Format

### Request Structure

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:wsdl:WebServiceMessagingProfile-Definitions-4.0"
                  xmlns:tyler="urn:tyler:ecf:extensions:CaseNotifyMessage"
                  xmlns:nc="http://niem.gov/niem/niem-core/2.0"
                  xmlns:jxdm="http://niem.gov/niem/domains/jxdm/4.0">
  <soapenv:Header/>
  <soapenv:Body>
    <ecf:NotifyCaseEvent>
      <ecf:NotifyCaseEventRequest>
        <tyler:NotifyCaseEventMessage>
          
          <!-- Court identification -->
          <jxdm:CaseCourt>
            <nc:OrganizationIdentification>
              <nc:IdentificationID>[court-code]</nc:IdentificationID>
            </nc:OrganizationIdentification>
          </jxdm:CaseCourt>
          
          <!-- Case identification -->
          <nc:CaseTrackingID>[case-id]</nc:CaseTrackingID>
          
          <!-- Reason for notification -->
          <tyler:NotificationReason>[reason-text]</tyler:NotificationReason>
          
          <!-- Event(s) that occurred -->
          <tyler:Event>
            <tyler:ID>[event-id]</tyler:ID>
            <tyler:EventType>[event-type]</tyler:EventType>
          </tyler:Event>
          
          <!-- Optional: Multiple events -->
          <tyler:Event>
            <tyler:ID>[event-id-2]</tyler:ID>
            <tyler:EventType>[event-type-2]</tyler:EventType>
          </tyler:Event>
          
        </tyler:NotifyCaseEventMessage>
      </ecf:NotifyCaseEventRequest>
    </ecf:NotifyCaseEvent>
  </soapenv:Body>
</soapenv:Envelope>
```

### Request Fields

| Field | Type | Required? | Description |
|-------|------|-----------|-------------|
| `CaseCourt/OrganizationIdentification/IdentificationID` | string | **Required** | Court code (e.g., "Hopkins:cc") |
| `CaseTrackingID` | string | **Required** | Authoritative CMS case identifier |
| `NotificationReason` | string | **Required** | Human-readable reason for notification |
| `Event/ID` | string | **Required** | Unique identifier for this event instance |
| `Event/EventType` | string | **Required** | Type of change (see Event Types guide) |

---

## Response Format

### Success Response

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Body>
    <ecf:NotifyCaseEventResponse>
      <tyler:Success>true</tyler:Success>
    </ecf:NotifyCaseEventResponse>
  </soapenv:Body>
</soapenv:Envelope>
```

**HTTP 200** indicates event received successfully. re:Search will call GetCase separately.

### Error Response

```xml
<soapenv:Envelope>
  <soapenv:Body>
    <soapenv:Fault>
      <faultcode>soapenv:Client</faultcode>
      <faultstring>Invalid EventType</faultstring>
      <detail>
        <tyler:ErrorDetail>
          <tyler:ErrorCode>INVALID_EVENT_TYPE</tyler:ErrorCode>
          <tyler:ErrorMessage>EventType 'InvalidType' is not supported</tyler:ErrorMessage>
        </tyler:ErrorDetail>
      </detail>
    </soapenv:Fault>
  </soapenv:Body>
</soapenv:Envelope>
```

---

## Required Fields

### Court Identification

```xml
<jxdm:CaseCourt>
  <nc:OrganizationIdentification>
    <nc:IdentificationID>Hopkins:cc</nc:IdentificationID>
  </nc:OrganizationIdentification>
</jxdm:CaseCourt>
```

**Format:** `[County/Court Name]:[court-type]`
- Examples: `Hopkins:cc`, `Travis:dc`, `Bexar:jp1`
- Must match court codes configured in re:Search
- Case-sensitive

### Case Identification

```xml
<nc:CaseTrackingID>CV-2024-00123</nc:CaseTrackingID>
```

**Requirements:**
- Must be the same CaseTrackingID used in GetCase
- Stable and persistent (doesn't change)
- Unique within the court

### Notification Reason

```xml
<tyler:NotificationReason>Party added to case</tyler:NotificationReason>
```

**Guidelines:**
- Human-readable explanation
- Brief but descriptive
- Examples:
  - "Party added to case"
  - "Case sealed by court order"
  - "Disposition entered"
  - "Filing docketed"

### Event Block

```xml
<tyler:Event>
  <tyler:ID>EVT-2024-12-04-0001</tyler:ID>
  <tyler:EventType>CaseParty</tyler:EventType>
</tyler:Event>
```

**Event ID:**
- Unique identifier for this event instance
- **For most EventTypes:** Can be any unique value (CMS-generated or arbitrary)
- **For specific EventTypes:** Must be a business object CMSID - see table below
- Useful for tracking/logging

**EventID Requirements by EventType:**

| EventType | EventID Must Be | How re:Search Uses It |
|-----------|----------------|----------------------|
| **CaseFiling** | **Filing CMSID** | Passed to GetCase as `DocketEntryTypeCodeFilterText` - filters to this filing only |
| **DocumentSecurity** | **Document CMSID** | Passed to GetDocument as `DocumentFileControlID` |
| **DocumentFiling** | **Document CMSID** | Passed to GetDocument as `DocumentFileControlID` |
| **CaseHearing** | **Hearing CMSID** | Passed to GetCase as `DocketEntryTypeCodeFilterText` - filters to this hearing only |
| All others | Any unique value | Typically ignored by re:Search |

**Format examples:**
- General: `EVT-[DATE]-[SEQUENCE]` → `EVT-20241204-0001`
- General: `[UUID]` → `550e8400-e29b-41d4-a716-446655440000`
- CaseFiling: Use your filing ID → `FIL-20241204-001`
- DocumentSecurity: Use your document ID → `DOC-789`
- CaseParty: Any unique value → `PARTY-UPDATE-123`

**EventType:**
- Must be a supported event type
- Case-sensitive
- See [Event Types Guide](./event-types.md) for complete list

---

## Multiple Events

You can include multiple events in a single NotifyCaseEvent:

```xml
<tyler:NotifyCaseEventMessage>
  <nc:CaseTrackingID>CV-2024-00123</nc:CaseTrackingID>
  <tyler:NotificationReason>Multiple case updates</tyler:NotificationReason>
  
  <tyler:Event>
    <tyler:ID>EVT-001</tyler:ID>
    <tyler:EventType>CaseParty</tyler:EventType>
  </tyler:Event>
  
  <tyler:Event>
    <tyler:ID>EVT-002</tyler:ID>
    <tyler:EventType>CaseFiling</tyler:EventType>
  </tyler:Event>
  
  <tyler:Event>
    <tyler:ID>EVT-003</tyler:ID>
    <tyler:EventType>CaseSecurity</tyler:EventType>
  </tyler:Event>
</tyler:NotifyCaseEventMessage>
```

**Result:** re:Search will call GetCase multiple times (once per EventType) to process each change.

**Alternative:** Use `CaseInitiated` EventType to process everything in one GetCase call.

---

## Error Handling

### HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Event received, GetCase will be called |
| 400 | Bad Request | Fix request format, check required fields |
| 401 | Unauthorized | Check mTLS certificate |
| 500 | Internal Server Error | re:Search error, retry with backoff |
| 503 | Service Unavailable | re:Search temporarily down, retry |

### Common Errors

**Invalid EventType:**
```xml
<faultstring>Invalid EventType</faultstring>
<ErrorMessage>EventType 'BadType' is not supported</ErrorMessage>
```

**Solution:** Use a supported EventType from the [Event Types Guide](./event-types.md)

**Missing Required Field:**
```xml
<faultstring>Missing required field</faultstring>
<ErrorMessage>CaseTrackingID is required</ErrorMessage>
```

**Solution:** Include all required fields

**Case Not Found:**
```xml
<faultstring>Case not found</faultstring>
<ErrorMessage>Case CV-2024-99999 does not exist in court Hopkins:cc</ErrorMessage>
```

**Solution:** Verify CaseTrackingID is correct and case exists

**Certificate Error:**
```xml
<faultstring>Authentication failed</faultstring>
<ErrorMessage>Invalid or expired mTLS certificate</ErrorMessage>
```

**Solution:** Check certificate validity and renewal

### Retry Logic

**Recommended retry pattern:**

```
Attempt 1: Immediate
   ↓ (failure)
Wait 1 second
   ↓
Attempt 2: +1 second
   ↓ (failure)
Wait 2 seconds
   ↓
Attempt 3: +2 seconds
   ↓ (failure)
Wait 4 seconds
   ↓
Attempt 4: +4 seconds
   ↓ (failure)
Alert operations team
```

**Exponential backoff:** 1s → 2s → 4s → 8s → 16s (max)

**When to retry:**
- HTTP 500 (Internal Server Error)
- HTTP 503 (Service Unavailable)
- Network timeout
- Connection refused

**When NOT to retry:**
- HTTP 400 (Bad Request) - fix the request first
- HTTP 401 (Unauthorized) - fix authentication first
- SOAP Fault with client error - fix the data first

---

## Complete Examples

### Example 1: Party Added

**Scenario:** Clerk adds new party to existing case

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:wsdl:WebServiceMessagingProfile-Definitions-4.0"
                  xmlns:tyler="urn:tyler:ecf:extensions:CaseNotifyMessage"
                  xmlns:nc="http://niem.gov/niem/niem-core/2.0"
                  xmlns:jxdm="http://niem.gov/niem/domains/jxdm/4.0">
  <soapenv:Header/>
  <soapenv:Body>
    <ecf:NotifyCaseEvent>
      <ecf:NotifyCaseEventRequest>
        <tyler:NotifyCaseEventMessage>
          <jxdm:CaseCourt>
            <nc:OrganizationIdentification>
              <nc:IdentificationID>Hopkins:cc</nc:IdentificationID>
            </nc:OrganizationIdentification>
          </jxdm:CaseCourt>
          <nc:CaseTrackingID>CV-2024-00123</nc:CaseTrackingID>
          <tyler:NotificationReason>Defendant Jane Doe added to case</tyler:NotificationReason>
          <tyler:Event>
            <tyler:ID>PARTY-ADD-2024-12-04-001</tyler:ID>
            <tyler:EventType>CaseParty</tyler:EventType>
          </tyler:Event>
        </tyler:NotifyCaseEventMessage>
      </ecf:NotifyCaseEventRequest>
    </ecf:NotifyCaseEvent>
  </soapenv:Body>
</soapenv:Envelope>
```

[Complete example →](./examples/case-party.xml)

---

### Example 2: Case Sealed (Security)

**Scenario:** Judge seals a case

```xml
<tyler:NotifyCaseEventMessage>
  <jxdm:CaseCourt>
    <nc:OrganizationIdentification>
      <nc:IdentificationID>Travis:dc</nc:IdentificationID>
    </nc:OrganizationIdentification>
  </jxdm:CaseCourt>
  <nc:CaseTrackingID>CR-2024-00456</nc:CaseTrackingID>
  <tyler:NotificationReason>Case sealed by court order #2024-789</tyler:NotificationReason>
  <tyler:Event>
    <tyler:ID>SEAL-2024-12-04-001</tyler:ID>
    <tyler:EventType>CaseSecurity</tyler:EventType>
  </tyler:Event>
</tyler:NotifyCaseEventMessage>
```

**Critical:** This must be sent **immediately** (synchronously), never queued or batched.

[Complete example →](./examples/case-security.xml)

---

### Example 3: Filing Docketed

**Scenario:** Clerk dockets an e-filed document

```xml
<tyler:NotifyCaseEventMessage>
  <jxdm:CaseCourt>
    <nc:OrganizationIdentification>
      <nc:IdentificationID>Bexar:cc</nc:IdentificationID>
    </nc:OrganizationIdentification>
  </jxdm:CaseCourt>
  <nc:CaseTrackingID>CV-2024-00789</nc:CaseTrackingID>
  <tyler:NotificationReason>Motion to Dismiss docketed</tyler:NotificationReason>
  <tyler:Event>
    <tyler:ID>FIL-20241204-MTD-001</tyler:ID> ← MUST be Filing CMSID!
    <tyler:EventType>CaseFiling</tyler:EventType>
  </tyler:Event>
</tyler:NotifyCaseEventMessage>
```

**Critical:** EventID MUST be the Filing CMSID - re:Search uses this to filter GetCase response.

[Complete example →](./examples/case-filing.xml)

---

### Example 4: Multiple Changes

**Scenario:** Bulk case update (party + disposition)

```xml
<tyler:NotifyCaseEventMessage>
  <jxdm:CaseCourt>
    <nc:OrganizationIdentification>
      <nc:IdentificationID>Ellis:cc</nc:IdentificationID>
    </nc:OrganizationIdentification>
  </jxdm:CaseCourt>
  <nc:CaseTrackingID>CV-2024-01000</nc:CaseTrackingID>
  <tyler:NotificationReason>Case closed with disposition</tyler:NotificationReason>
  <tyler:Event>
    <tyler:ID>DISP-2024-12-04-001</tyler:ID>
    <tyler:EventType>CaseDisposition</tyler:EventType>
  </tyler:Event>
  <tyler:Event>
    <tyler:ID>STATUS-2024-12-04-001</tyler:ID>
    <tyler:EventType>CaseStatus</tyler:EventType>
  </tyler:Event>
</tyler:NotifyCaseEventMessage>
```

**Alternative:** Use `CaseInitiated` to process all changes at once.

[Complete example →](./examples/generic-example.xml)

---

### Example 5: New Case Created

**Scenario:** Brand new case filed

```xml
<tyler:NotifyCaseEventMessage>
  <jxdm:CaseCourt>
    <nc:OrganizationIdentification>
      <nc:IdentificationID>Tarrant:dc</nc:IdentificationID>
    </nc:OrganizationIdentification>
  </jxdm:CaseCourt>
  <nc:CaseTrackingID>FM-2024-02000</nc:CaseTrackingID>
  <tyler:NotificationReason>New family law case filed</tyler:NotificationReason>
  <tyler:Event>
    <tyler:ID>NEW-CASE-2024-12-04-001</tyler:ID>
    <tyler:EventType>CaseInitiated</tyler:EventType>
  </tyler:Event>
</tyler:NotifyCaseEventMessage>
```

**EventType:** `CaseInitiated` processes the complete case (all blocks in GetCase response).

---

### Additional XML Examples

The [examples directory](./examples/) contains complete NotifyCaseEvent XML files for all EventTypes:

| File | EventType | Key Notes |
|------|-----------|-----------|
| [case-filing.xml](./examples/case-filing.xml) | CaseFiling | EventID = Filing CMSID (required) |
| [case-party.xml](./examples/case-party.xml) | CaseParty | EventID can be any unique value |
| [case-party-attorney.xml](./examples/case-party-attorney.xml) | CasePartyAttorney | EventID can be any unique value |
| [case-security.xml](./examples/case-security.xml) | CaseSecurity | Must be sent immediately |
| [case-disposition.xml](./examples/case-disposition.xml) | CaseDisposition | EventID can be any unique value |
| [case-reassigned.xml](./examples/case-reassigned.xml) | CaseReassigned | EventID can be any unique value |
| [case-delete.xml](./examples/case-delete.xml) | CaseDelete | EventID can be any unique value |
| [case-expunge.xml](./examples/case-expunge.xml) | CaseExpunge | EventID can be any unique value |
| [document-filing.xml](./examples/document-filing.xml) | DocumentFiling | EventID = Document CMSID (required) |
| [document-security.xml](./examples/document-security.xml) | DocumentSecurity | EventID = Document CMSID, send immediately |
| [generic-example.xml](./examples/generic-example.xml) | Multiple events | Shows multiple events in one request |

**Use these as templates** for your NotifyCaseEvent implementation.

---

## Implementation Examples

### C# Example

```csharp
public async Task SendNotifyCaseEventAsync(
    string courtCode, 
    string caseTrackingId, 
    string eventType, 
    string reason)
{
    var client = new ResearchClient();
    var request = new NotifyCaseEventRequest
    {
        CaseCourt = new CaseCourt
        {
            OrganizationIdentification = new OrganizationIdentification
            {
                IdentificationID = courtCode
            }
        },
        CaseTrackingID = caseTrackingId,
        NotificationReason = reason,
        Event = new Event
        {
            ID = $"EVT-{DateTime.UtcNow:yyyyMMdd-HHmmss}",
            EventType = eventType
        }
    };
    
    try
    {
        var response = await client.NotifyCaseEventAsync(request);
        _logger.LogInformation(
            $"Event sent successfully: {caseTrackingId} - {eventType}");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, 
            $"Failed to send event: {caseTrackingId} - {eventType}");
        // Queue for retry
        await _retryQueue.EnqueueAsync(request);
    }
}
```

### Java Example

```java
public void sendNotifyCaseEvent(
    String courtCode, 
    String caseTrackingId, 
    String eventType, 
    String reason) {
    
    NotifyCaseEventMessage message = new NotifyCaseEventMessage();
    
    CaseCourt court = new CaseCourt();
    OrganizationIdentification orgId = new OrganizationIdentification();
    orgId.setIdentificationID(courtCode);
    court.setOrganizationIdentification(orgId);
    message.setCaseCourt(court);
    
    message.setCaseTrackingID(caseTrackingId);
    message.setNotificationReason(reason);
    
    Event event = new Event();
    event.setID("EVT-" + System.currentTimeMillis());
    event.setEventType(eventType);
    message.setEvent(event);
    
    try {
        researchClient.notifyCaseEvent(message);
        logger.info("Event sent: {} - {}", caseTrackingId, eventType);
    } catch (Exception e) {
        logger.error("Failed to send event: {} - {}", 
            caseTrackingId, eventType, e);
        retryQueue.add(message);
    }
}
```

---

## Best Practices

### Event ID Generation

**Good patterns:**
- `EVT-{DATE}-{SEQUENCE}` - `EVT-20241204-0001`
- `{UUID}` - `550e8400-e29b-41d4-a716-446655440000`
- `{TYPE}-{DATE}-{SEQUENCE}` - `PARTY-20241204-001`

**Avoid:**
- Non-unique IDs
- Special characters that need escaping
- Extremely long IDs (> 100 characters)

### Notification Reason Guidelines

**Good:**
- "Party Jane Doe added to case"
- "Case sealed by court order #2024-789"
- "Disposition entered: Dismissed with prejudice"
- "Motion to Dismiss filed and docketed"

**Avoid:**
- Too vague: "Update"
- Too long: 500-character essays
- Technical jargon: "CaseParticipant entity inserted into DB table"

### Message Size

**Keep messages small:**
- Target: < 5 KB
- Maximum: < 50 KB
- Don't include case data (that's what GetCase is for)

---

## Testing Checklist

Before certification:

- [ ] All EventTypes tested
- [ ] Required fields present in all requests
- [ ] Court codes formatted correctly
- [ ] Event IDs unique per event
- [ ] Security events sent immediately (not queued)
- [ ] Retry logic working correctly
- [ ] Logging comprehensive
- [ ] mTLS certificate valid
- [ ] GetCase triggered after event
- [ ] Multiple events in single request tested
- [ ] Error handling working correctly

---

## Next Steps

- **Learn all event types** → Read [Event Types Guide](./event-types.md)
- **See integration context** → Read [Workflows](./workflows.md)
- **Review examples** → Browse [examples directory](./examples/)
- **Return to overview** → [NotifyCaseEvent Overview](./README.md)

---

**Last Updated:** December 2025