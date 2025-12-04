# GetCase API

**Navigation:** [Home](../../../README.md) › [Technical Documentation](../../README.md) › [API Reference](../README.md) › GetCase

## Quick Navigation
- [Request & Response Specification](./request-response.md) - Technical details and examples
- [Behavior Guide](./behavior-guide.md) - How re:Search processes GetCase
- [Workflows](./workflows.md) - Sequence diagrams and integration patterns
- [XML Examples](./examples/) - Sample request/response files

---

## What is GetCase?

GetCase is the **data retrieval API** that returns complete, authoritative case information from your CMS to re:Search. It's called in response to events and provides the full case dataset needed to update the re:Search index.

**Think of it as:** re:Search asking "Show me the current state of this case"

---

## At a Glance

| Property | Value |
|---------|--------|
| **Direction** | re:Search → CMS |
| **Protocol** | SOAP 1.2 over HTTPS |
| **Security** | Mutual TLS (mTLS) |
| **Standard** | OASIS ECF 4.x |
| **Message Type** | GetCaseRequest / GetCaseResponse |
| **Required?** | Yes (all ECF/CIP integrations) |

---

## When GetCase is Called

GetCase is triggered immediately after:

1. **NotifyCaseEvent received** - CMS notifies re:Search of a case change
2. **NotifyDocketingComplete received** - Filing has been docketed
3. **System reindex operation** - Manual or scheduled refresh

**Example flow:**
```
CMS change happens → NotifyCaseEvent sent → re:Search calls GetCase → Case updated
```

---

## What GetCase Returns

**Complete case dataset including:**
- Case metadata (number, title, type, status, dates)
- All filing events and docket entries
- All parties with roles and contact information
- All attorneys with party associations
- All documents with metadata (no binaries)
- Current security settings (case and document level)
- Judge assignments and court information

**Critical:** GetCase must return the **current, complete state** of the case - not cached, not partial.

---

## Quick Example

### Request
```xml
<ecf:GetCaseRequest>
  <tyler:GetCaseRequestMessage>
    <tyler:MessageID>unique-request-id</tyler:MessageID>
    <tyler:RequestDateTime>2025-12-04T10:30:00Z</tyler:RequestDateTime>
    
    <!-- Which court? -->
    <tyler:CourtIdentification>
      <nc:IdentificationID>Hopkins:cc</nc:IdentificationID>
    </tyler:CourtIdentification>
    
    <!-- Which case? -->
    <tyler:CaseTrackingID>CASE-12345678</tyler:CaseTrackingID>
    
    <!-- Include documents and parties? -->
    <tyler:IncludeDocumentMetadata>true</tyler:IncludeDocumentMetadata>
    <tyler:IncludePartyInformation>true</tyler:IncludePartyInformation>
  </tyler:GetCaseRequestMessage>
</ecf:GetCaseRequest>
```

### Response (Abbreviated)
```xml
<ecf:GetCaseResponse>
  <tyler:GetCaseResponseMessage>
    <ecf:Case>
      <ecf:CaseTrackingID>CASE-12345678</ecf:CaseTrackingID>
      <ecf:CaseTitleText>Smith v. Jones</ecf:CaseTitleText>
      <tyler:CaseSecurity>PublicFilingPublicView</tyler:CaseSecurity>
      
      <!-- Parties, filings, documents... -->
    </ecf:Case>
  </tyler:GetCaseResponseMessage>
</ecf:GetCaseResponse>
```

[See complete examples →](./request-response.md#complete-examples)

---

## Key Requirements

### Must Include (Required)
- ✅ Complete case payload (not partial updates)
- ✅ Case security settings (`tyler:CaseSecurity`)
- ✅ All filing events with timestamps (`ecf:ActivityDate`)
- ✅ Document CMSIDs (`nc:IdentificationID`) for all documents
- ✅ Document page counts (`tyler:PageCount`) for all documents
- ✅ Valid ISO 8601 timestamps (UTC)

### Must Not Include (Forbidden)
- ❌ Embedded binary document content (use GetDocument instead)
- ❌ Cached or stale data (always return current state)
- ❌ Empty `<Case>` blocks (return SOAP fault if case not found)
- ❌ Partial case data (must be complete for EventType)

### Performance Expectations
- **Simple cases:** < 1 second response time
- **Average cases:** < 2 seconds response time
- **Complex cases:** < 3 seconds response time
- **Maximum timeout:** 5 seconds

---

## Common Issues

| Issue | Cause | Impact |
|-------|-------|--------|
| **Missing timestamps** | `ActivityDate` not provided | Filings appear out of order or missing |
| **Missing CMSIDs** | Document `IdentificationID` not provided | Documents can't be retrieved |
| **Stale data** | Returning cached response | Changes don't appear in re:Search |
| **Wrong EventType mapping** | Including irrelevant data blocks | Updates ignored by re:Search |
| **Missing security** | `CaseSecurity` not provided | Case visibility incorrect |

[Complete troubleshooting guide →](./behavior-guide.md#known-vendor-pitfalls)

---

## Understanding EventType Mapping

**Critical concept:** re:Search only processes XML blocks that match the EventType that triggered GetCase.

| EventType | What Gets Processed | What Gets Ignored |
|-----------|---------------------|-------------------|
| `CaseFiling` | Filing events, documents | Parties, security changes |
| `CaseParty` | Party and attorney data | Filings, documents |
| `CaseSecurity` | Security settings only | Everything else |
| `CaseDisposition` | Disposition data | Everything else |

**Why this matters:** If you send a party change but used `CaseFiling` event type, the party change will be ignored.

[Complete EventType mapping →](./behavior-guide.md#eventtype-mapping)

---

## Quick Start Checklist

### For Developers Implementing GetCase

- [ ] Review [Request & Response Specification](./request-response.md)
- [ ] Understand [EventType mapping rules](./behavior-guide.md#eventtype-mapping)
- [ ] Review [complete XML examples](./request-response.md#complete-examples)
- [ ] Test with all EventTypes
- [ ] Verify security settings correct
- [ ] Confirm timestamps are ISO 8601 UTC
- [ ] Ensure CMSIDs stable and persistent
- [ ] Test response times meet targets (< 3s)
- [ ] Validate with [certification checklist](./behavior-guide.md#certification-checklist)

### For Operations Teams

- [ ] Monitor GetCase response times
- [ ] Alert on timeouts or errors
- [ ] Review error logs regularly
- [ ] Ensure mTLS certificates valid
- [ ] Verify data freshness (not cached)

---

## Related APIs

GetCase works in coordination with these APIs:

**Triggers GetCase:**
- [NotifyCaseEvent](../notifycaseevent/) - Event notification from CMS
- [NotifyDocketingComplete](../notifydocketingcomplete/) - Docketing confirmation
- [RecordFiling](../recordfiling/) - Filing recorded (via EFM)

**Called After GetCase:**
- [GetDocument](../getdocument/) - Retrieve document binaries

**Workflow:**
```
NotifyCaseEvent → GetCase → [GetDocument if needed]
```

---

## Authentication

GetCase requires mutual TLS (mTLS) authentication.

**Certificate requirements:**
- Valid mTLS certificate issued by Tyler
- Certificate must not be expired
- Proper certificate chain validation
- Secure certificate storage

[Authentication details →](../common-headers-and-auth.md)

---

## Next Steps

### New to GetCase?
1. Read [Request & Response Specification](./request-response.md) to understand the API structure
2. Review [Behavior Guide](./behavior-guide.md) to understand how re:Search processes responses
3. Study [Workflows](./workflows.md) to see GetCase in context

### Implementing GetCase?
1. Start with [Request & Response Specification](./request-response.md)
2. Download [example XML files](./examples/)
3. Test with simple cases first
4. Use [Behavior Guide](./behavior-guide.md) for validation rules
5. Complete [certification checklist](./behavior-guide.md#certification-checklist)

### Troubleshooting Issues?
1. Check [common issues](#common-issues) above
2. Review [Behavior Guide pitfalls](./behavior-guide.md#known-vendor-pitfalls)
3. Verify [EventType mapping](./behavior-guide.md#eventtype-mapping)
4. Contact your Tyler Technical Project Manager

---

## Additional Resources

- [Security Logic](../../support-playbook/security-logic.md) - How security rules work
- [Registered User Matrix](../../support-playbook/registered-user-matrix.md) - Access control
- [Troubleshooting Guide](../../support-playbook/troubleshooting.md) - Common problems
- [Full XML Library](../../xml-library/xml-library.md) - All XML examples

---

**Last Updated:** December 2025