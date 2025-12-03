# API Reference

> **Breadcrumb:** [Home](../README.md) > API Reference

Complete technical reference for all re:Search APIs. Each endpoint includes detailed specifications, behavior guides, and annotated XML examples.

---

## Overview

re:Search integrations use SOAP/XML APIs to exchange case data between your CMS and the re:Search platform. The APIs you'll work with depend on your [integration mode](../integration-guide/integration-modes/README.md).

**Integration Mode → APIs Used:**

| Mode | APIs Required |
|------|---------------|
| **ECF Mode** | NotifyCaseEvent (send), GetCase (receive), GetDocument (receive) |
| **CIP Mode** | EJ specific |
| **Batch Mode** | None - uses file exports instead |
| **Non-Integrated** | None - manual portal entry |

---

## Core APIs

### [NotifyCaseEvent](./notifycaseevent/README.md)
**Direction:** CMS → re:Search  
**Purpose:** Notify re:Search when case data changes

Send this event whenever:
- A new case is filed
- Case information is updated
- Parties or attorneys are added/changed
- Documents are filed
- Case security changes
- Case is disposed or reassigned

**Key Resources:**
- [Event Types Reference](./notifycaseevent/event-types.md) - Complete list of event types
- [Implementation Guide](./notifycaseevent/implementation-guide.md) - Best practices
- [XML Examples](./notifycaseevent/examples/) - Annotated examples for each event type

**When to use:** ECF and CIP modes only

---

### [GetCase](./getcase/README.md)
**Direction:** re:Search → CMS  
**Purpose:** Return current case data when requested

re:Search calls this endpoint on your CMS after receiving a NotifyCaseEvent. You must return:
- Case metadata (number, type, status, filing date)
- All parties and attorneys
- All docket entries
- Document metadata
- Security settings

**Key Resources:**
- [Behavior Guide](./getcase/behavior-guide.md) - Response requirements and edge cases
- [XML Examples](./getcase/examples/) - Complete request/response examples

**When to use:** ECF and CIP modes only

---

### [GetDocument](./getdocument/README.md)
**Direction:** re:Search → CMS  
**Purpose:** Return document PDFs when requested

re:Search calls this endpoint when users view documents. You must return:
- PDF file as base64-encoded content
- Appropriate security enforcement
- Error responses for sealed/unavailable documents

**Key Resources:**
- [Implementation Guide](./getdocument/README.md#implementation-guidelines)
- [XML Examples](./getdocument/examples/) - Request/response with security handling

**When to use:** ECF and CIP modes only

---

### [RecordFiling](./recordfiling/README.md)
**Direction:** EFM → CMS  
**Purpose:** Receive e-filing submissions from EFM

**Note:** This API is used for **e-filing integration**, not re:Search integration. It's included here because many courts implement both simultaneously.

The EFM calls this endpoint to deliver approved e-filings to your CMS. You must:
- Accept the filing package
- Docket the filing
- Generate case/document IDs
- Send NotifyCaseEvent to trigger re:Search update

**Key Resources:**
- [Behavior Guide](./recordfiling/behavior-guide.md) - Acceptance criteria and error handling
- [XML Examples](./recordfiling/examples/) - Filing submission examples

**When to use:** Courts accepting e-filings through EFM

---

### [NotifyDocketingComplete](./notifydocketingcomplete/README.md)
**Direction:** re:Search → CMS  
**Purpose:** Receive confirmation after re:Search indexes a case (optional)

This is an **optional callback** that re:Search can send to your CMS after successfully indexing a case. Most implementations don't use this.

**Key Resources:**
- [Implementation Guide](./notifydocketingcomplete/README.md)
- [XML Examples](./notifydocketingcomplete/examples/) - Callback message structure

**When to use:** Optional - only if you need confirmation of indexing

---

## Authentication & Headers

All re:Search APIs use SOAP with WS-Security authentication.

### [Authentication Guide](./authentication.md)

**Topics covered:**
- WS-Security SOAP headers
- X.509 certificate requirements
- Username/password tokens
- Timestamp validation
- Security best practices

**Required reading before implementation**

---

## Error Handling

### [Error Codes Reference](./error-codes.md)

**Topics covered:**
- Standard SOAP fault codes
- re:Search-specific error codes
- Error response formats
- Retry strategies
- Logging recommendations

**Use this when:** Implementing error handling or troubleshooting API failures

---

## Implementation Workflow

### For ECF Mode:
```
1. Implement GetCase endpoint
   ↓
2. Test with re:Search team
   ↓
3. Implement GetDocument endpoint
   ↓
4. Test document retrieval
   ↓
5. Implement NotifyCaseEvent sending
   ↓
6. End-to-end testing
```

### For CIP Mode:
```
1. Work with CIP team to expose GetCase
   ↓
2. Validate CIP-generated responses
   ↓
3. Work with CIP team to expose GetDocument
   ↓
4. Ensure CIP sends NotifyCaseEvent
   ↓
5. End-to-end testing
```

[See detailed implementation guides →](../integration-guide/integration-modes/README.md)

---

## API Quick Reference

| API | Direction | Required? | Purpose |
|-----|-----------|-----------|---------|
| [NotifyCaseEvent](./notifycaseevent/README.md) | CMS → re:Search | ECF | Trigger case updates |
| [GetCase](./getcase/README.md) | re:Search → CMS | ECF | Retrieve case data |
| [GetDocument](./getdocument/README.md) | re:Search → CMS | ECF | Retrieve documents |
| [RecordFiling](./recordfiling/README.md) | EFM → CMS | E-filing only | Accept e-filings |
| [NotifyDocketingComplete](./notifydocketingcomplete/README.md) | re:Search → CMS | Optional | Indexing confirmation |

---

## Common Implementation Patterns

### Event Generation Strategy
**When to send NotifyCaseEvent:**
- Immediately after case data changes in CMS
- After successful transaction commit
- Include all relevant event types for the change

[See Event Type Logic →](../implementation-playbook/event-type-logic.md)

### GetCase Response Optimization
**Performance considerations:**
- Cache responses with short TTL (5-10 minutes)
- Optimize database queries
- Include only necessary data
- Handle large case files efficiently

[See Best Practices →](../implementation-playbook/best-practices.md)

### Security Enforcement
**Critical requirements:**
- GetDocument must enforce same security as CMS
- Check both case-level and document-level security
- Return appropriate error codes for denied access
- Never expose sealed/confidential content

[See Security Logic →](../implementation-playbook/security-logic.md)

---

## Testing Your Implementation

### Functional Testing
- [ ] All event types trigger correctly
- [ ] GetCase returns accurate data
- [ ] GetDocument returns correct PDFs
- [ ] Security properly enforced
- [ ] Error handling works correctly

### Data Validation
- [ ] Case metadata matches CMS
- [ ] All parties and attorneys included
- [ ] Docket entries complete and accurate
- [ ] Document metadata correct
- [ ] Security levels match CMS

### Performance Testing
- [ ] GetCase response < 5 seconds
- [ ] GetDocument response < 10 seconds
- [ ] System handles expected load
- [ ] No timeouts under normal conditions

[Complete testing guide →](../integration-guide/onboarding/06-testing-and-certification.md)

---

## Troubleshooting API Issues

**Common problems and solutions:**

### NotifyCaseEvent not triggering updates
→ [Event Handling Troubleshooting](../troubleshooting/event-handling.md)

### GetCase timeouts or errors
→ [Performance Optimization](../troubleshooting/performance-optimization.md)

### GetDocument access issues
→ [Security Troubleshooting](../troubleshooting/security-and-authentication.md)

### Authentication failures
→ [Authentication Guide](./authentication.md#troubleshooting)

[Complete troubleshooting guide →](../troubleshooting/README.md)

---

## Market-Specific Considerations

Some markets have additional API requirements or restrictions.

**Check your market standards:**
- [Texas (JCIT)](../market-standards/texas/README.md) - Event type requirements, case type restrictions
- [Illinois (AOIC)](../market-standards/illinois/README.md) - Security rules, data requirements
- [Other Markets](../market-standards/other-markets/README.md) - Market-specific variations

[Review all market standards →](../market-standards/README.md)

---

## Related Documentation

### For Integration Process
→ [Integration Guide](../integration-guide/README.md) - Step-by-step onboarding

### For Implementation Patterns
→ [Implementation Playbook](../implementation-playbook/README.md) - Best practices and architecture

### For Market Rules
→ [Market Standards](../market-standards/README.md) - Jurisdiction-specific requirements

### For Problem Solving
→ [Troubleshooting Guide](../troubleshooting/README.md) - Issue resolution

---

## Additional Resources

**XML Schema Files:**
Contact your Tyler Technical Project Manager (TPM) for XSD schema files

**Test Data:**
Sample XML files available in each API's `/examples/` folder

**Support:**
For API questions, contact your assigned TPM

---

**Last Updated:** January 2025