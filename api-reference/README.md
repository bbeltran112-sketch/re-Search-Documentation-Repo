# API Reference

**Navigation:** [Home](../README.md) › API Reference

Complete technical reference for all re:Search APIs. Each API includes detailed specifications, implementation guides, and annotated XML examples.

---

## Overview

re:Search integrations use SOAP/XML APIs to exchange case data between your CMS and the re:Search platform. The APIs you'll work with depend on your [integration mode](../integration-guide/integration-modes/README.md).

**Integration Mode → APIs Used:**

| Mode | APIs Required |
|------|---------------|
| **ECF Mode** | GetCase, GetDocument, NotifyCaseEvent |
| **CIP Mode** | GetCase, GetDocument (via CIP), NotifyCaseEvent (via CIP) |
| **Batch Mode** | None - uses file exports instead |
| **Non-Integrated** | None - manual portal entry |
| **E-Filing** | RecordFiling (separate from re:Search) |

---

## Quick Navigation

| API | Direction | Purpose | Status |
|-----|-----------|---------|--------|
| **[GetCase](#getcase)** | re:Search → CMS | Retrieve case data | ✅ Complete |
| **[GetDocument](#getdocument)** | re:Search → CMS | Retrieve document binaries | ✅ Complete |
| **[NotifyCaseEvent](#notifycaseevent)** | CMS → re:Search | Notify of case changes | ✅ Complete |
| **[RecordFiling](#recordfiling)** | EFM → CMS | Receive e-filing submissions | 📝 In Progress |
| **[NotifyDocketingComplete](#notifydocketingcomplete)** | CMS → re:Search | Confirm docketing complete | 📝 In Progress |

---

## Core re:Search APIs

These are the primary APIs used for re:Search integration in ECF Mode.

### GetCase

**Direction:** re:Search → CMS  
**Purpose:** Retrieve complete case data on-demand

re:Search calls GetCase after receiving NotifyCaseEvent to fetch the latest case information.

**📁 Documentation Structure:**
```
api-reference-getcase/
├── README.md              ⭐ Quick reference - START HERE
├── request-response.md    📋 Request/response formats, fields, examples
├── behavior-guide.md      ⚙️ How re:Search processes responses
├── workflows.md           🔄 Integration patterns and troubleshooting
└── examples/              📂 Annotated XML files
```

**Key Pages:**
- [**Quick Reference**](./getcase/README.md) - What is GetCase, when it's called, key requirements
- [**Request & Response**](./getcase/request-response.md) - Complete technical specification
- [**Behavior Guide**](./getcase/behavior-guide.md) - How re:Search processes your response
- [**Workflows**](./getcase/workflows.md) - Sequence diagrams and integration patterns
- [**XML Examples**](./getcase/examples/) - Annotated request/response samples

**What You'll Return:**
- Case metadata (number, type, status, filing date)
- All parties and attorneys
- All filing events (docket entries)
- Document metadata with CMSIDs
- Security settings (case and document level)

**Performance Target:** < 3 seconds response time

---

### GetDocument

**Direction:** re:Search → CMS  
**Purpose:** Retrieve document binaries (PDFs, TIFFs) for viewing

re:Search calls GetDocument when users click to view or download documents.

**📁 Documentation Structure:**
```
api-reference-getdocument/
├── README.md              ⭐ Quick reference - START HERE
├── request-response.md    📋 Request/response formats, fields, examples
├── security-guide.md      🔒 Security enforcement rules
├── workflows.md           🔄 User-triggered vs event-triggered retrieval
└── examples/              📂 Annotated XML files
```

**Key Pages:**
- [**Quick Reference**](./getdocument/README.md) - What is GetDocument, when it's called
- [**Request & Response**](./getdocument/request-response.md) - Complete technical specification
- [**Security Guide**](./getdocument/security-guide.md) - How to enforce document security
- [**Workflows**](./getdocument/workflows.md) - Caching, performance, troubleshooting
- [**XML Examples**](./getdocument/examples/) - Request/response samples

**What You'll Return:**
- Base64-encoded document binary (PDF/TIFF/image)
- Document metadata (filing code, description, post date)
- File information (name, size, format)

**Critical:** Must enforce case and document security - never return sealed/restricted documents to unauthorized users.

**Performance Target:** < 2 seconds for typical files, < 5 seconds for large files

---

### NotifyCaseEvent

**Direction:** CMS → re:Search  
**Purpose:** Notify re:Search when case data changes

Your CMS sends NotifyCaseEvent to trigger re:Search to call GetCase and refresh case data.

**📁 Documentation Structure:**
```
api-reference-notifycaseevent/
├── README.md              ⭐ Quick reference - START HERE
├── request-response.md    📋 Request/response formats, fields, examples
├── event-types.md         📑 All supported EventTypes and when to use them
├── workflows.md           🔄 When and how to send events
└── examples/              📂 XML examples for each EventType
```

**Key Pages:**
- [**Quick Reference**](./notifycaseevent/README.md) - What is NotifyCaseEvent, when to send
- [**Request & Response**](./notifycaseevent/request-response.md) - Complete technical specification
- [**Event Types**](./notifycaseevent/event-types.md) - All 11 EventTypes explained
- [**Workflows**](./notifycaseevent/workflows.md) - Integration patterns and timing rules
- [**XML Examples**](./notifycaseevent/examples/) - Sample events for each type

**When to Send:**
- Party added/changed → `CaseParty`
- Filing docketed → `CaseFiling`
- Case sealed → `CaseSecurity` (immediate!)
- Disposition entered → `CaseDisposition`
- Case reassigned → `CaseReassigned`
- New case created → `CaseInitiated`
- Document security changed → `DocumentSecurity` (immediate!)

**Critical:** Security events (CaseSecurity, DocumentSecurity) must be sent **synchronously** - never queue or batch!

**Performance Target:** < 1 second delivery time

---

## E-Filing API

This API is used for e-filing integration, separate from re:Search integration. Many courts implement both.

### RecordFiling

**Direction:** EFM → CMS  
**Purpose:** Receive e-filing submissions from Tyler's Electronic Filing Manager (EFM)

When a clerk approves an e-filing in EFM, EFM calls RecordFiling on your CMS to deliver the filing package.

**Status:** 📝 Documentation in progress

**What You'll Receive:**
- Complete filing package from EFM
- Lead document and all attachments
- Filer information
- Case identification
- Filing codes and metadata

**What You Must Do:**
1. Accept and validate the filing
2. Docket the filing in your CMS
3. Generate case and document CMSIDs
4. **Send NotifyCaseEvent** to trigger re:Search update
5. Return success/failure response to EFM

**Integration with re:Search:**
```
EFM → RecordFiling → CMS dockets filing → NotifyCaseEvent → re:Search → GetCase
```

**Key Concept:** RecordFiling is the entry point for e-filings into your CMS. After docketing, you send NotifyCaseEvent to make the filing visible in re:Search.

**Documentation:** Coming soon - check back for complete implementation guide

---

## Optional APIs

### NotifyDocketingComplete

**Direction:** CMS → re:Search (optional callback)  
**Purpose:** Receive confirmation after re:Search successfully indexes a case

**Status:** 📝 Documentation in progress

This is an **optional callback** that re:Search can send to your CMS after successfully processing a GetCase response and updating its index.

**When re:Search Sends This:**
- After successful GetCase processing
- After case data indexed and searchable
- Includes confirmation of what was indexed

**Typical Use Cases:**
- CMS wants confirmation that re:Search received the update
- Audit logging
- Troubleshooting synchronization issues

**Note:** Most implementations don't use this callback. Only implement if you have a specific need for indexing confirmation.

**Documentation:** Coming soon - check back for complete specification

---

## API Comparison

### Quick Decision Guide

**Need to update case data in re:Search?**
→ Send [NotifyCaseEvent](#notifycaseevent), re:Search will call [GetCase](#getcase)

**Need to provide document for viewing?**
→ Implement [GetDocument](#getdocument) endpoint

**Accepting e-filings from EFM?**
→ Implement [RecordFiling](#recordfiling) endpoint

**Want confirmation re:Search indexed your case?**
→ Optional: Implement [NotifyDocketingComplete](#notifydocketingcomplete) endpoint

---

## API Architecture

### Data Flow for ECF Mode

```
┌─────────────────────────────────────────────────────────────┐
│                      Case Update Flow                        │
└─────────────────────────────────────────────────────────────┘

CMS                        re:Search
 │                             │
 │  1. Case data changes       │
 │                             │
 │─────NotifyCaseEvent────────>│
 │     (lightweight event)     │
 │                             │
 │<────────GetCase─────────────│
 │    (retrieve full data)     │
 │                             │
 │─────GetCase Response───────>│
 │    (complete case data)     │
 │                             │
 │                             │  2. User searches
 │                             │     case appears
 │                             │
 │                             │  3. User clicks document
 │                             │
 │<─────GetDocument────────────│
 │    (request PDF)            │
 │                             │
 │───GetDocument Response─────>│
 │   (Base64 PDF content)      │
 │                             │
 │                             │  4. User views document
```

### E-Filing Flow (with re:Search)

```
┌─────────────────────────────────────────────────────────────┐
│                    E-Filing Integration                      │
└─────────────────────────────────────────────────────────────┘

EFM                CMS                    re:Search
 │                  │                         │
 │  Filing approved │                         │
 │                  │                         │
 │─RecordFiling────>│                         │
 │  (filing pkg)    │                         │
 │                  │  CMS dockets filing     │
 │                  │                         │
 │<─Success─────────│                         │
 │                  │                         │
 │                  │──NotifyCaseEvent───────>│
 │                  │   (CaseFiling)          │
 │                  │                         │
 │                  │<────GetCase─────────────│
 │                  │                         │
 │                  │──GetCase Response──────>│
 │                  │  (includes new filing)  │
 │                  │                         │
 │                  │                         │  Filing now searchable
```

---

## Common Implementation Patterns

### Pattern 1: New Case Filed

```
1. Clerk creates new case in CMS
2. CMS sends NotifyCaseEvent (EventType: CaseInitiated)
3. re:Search calls GetCase
4. CMS returns complete case data
5. re:Search indexes case
6. Case appears in search results
```

**Timeline:** 5-15 seconds from case creation to searchable

---

### Pattern 2: Party Added to Case

```
1. Clerk adds defendant to case in CMS
2. CMS sends NotifyCaseEvent (EventType: CaseParty)
3. re:Search calls GetCase
4. CMS returns updated party list
5. re:Search updates case index
6. New party visible in search results
```

**Timeline:** 5-10 seconds

---

### Pattern 3: Case Sealed

```
1. Judge seals case in CMS
2. CMS sends NotifyCaseEvent (EventType: CaseSecurity) ← IMMEDIATE!
3. re:Search calls GetCase
4. CMS returns case with SealedCase security
5. re:Search removes case from public search
6. Case hidden from public users
```

**Timeline:** 5 seconds (high priority)

**Critical:** Security events must be sent **immediately and synchronously** - never queue!

---

### Pattern 4: E-Filing Docketed

```
1. Attorney e-files motion in portal
2. Clerk reviews and approves in EFM
3. EFM calls RecordFiling on CMS
4. CMS dockets filing, generates CMSIDs
5. CMS sends NotifyCaseEvent (EventType: CaseFiling)
6. re:Search calls GetCase
7. CMS returns filing with document metadata
8. re:Search indexes new filing
9. Filing searchable, documents viewable
```

**Timeline:** Minutes to hours (depends on clerk review time)

---

## Performance Expectations

### Response Time Targets

| API | Target | Maximum | Notes |
|-----|--------|---------|-------|
| **NotifyCaseEvent** (delivery) | < 1s | < 2s | CMS → re:Search |
| **GetCase** (simple case) | < 1s | < 3s | Small case, few parties |
| **GetCase** (average case) | < 2s | < 5s | Typical case complexity |
| **GetCase** (complex case) | < 3s | < 5s | Many parties/filings |
| **GetDocument** (< 1 MB) | < 1s | < 2s | Small PDF |
| **GetDocument** (1-10 MB) | < 2s | < 5s | Average PDF |
| **GetDocument** (10-50 MB) | < 5s | < 10s | Large PDF |
| **RecordFiling** | < 5s | < 10s | EFM → CMS |

### API Call Frequency

**Typical court (500 e-filings/month):**
- NotifyCaseEvent: ~1,500 events/month
- GetCase: ~1,500 calls/month
- GetDocument: ~1,000 calls/month (70% cache hit rate)

**Large court (5,000 e-filings/month):**
- NotifyCaseEvent: ~15,000 events/month
- GetCase: ~15,000 calls/month
- GetDocument: ~10,000 calls/month (75% cache hit rate)

---

## Security & Authentication

All re:Search APIs use **mutual TLS (mTLS)** authentication.

### Certificate Requirements

- Valid mTLS certificate issued by Tyler Technologies
- Certificate must not be expired
- Proper certificate chain validation
- Secure certificate storage (HSM recommended)
- Certificate rotation every 12-18 months

### SOAP Security Headers

All requests include WS-Security headers with:
- X.509 certificate token
- Timestamp validation
- Message signing

**Documentation:** Contact your Tyler Technical Project Manager for authentication setup guide

---

## Testing Your Implementation

### Development Testing

**GetCase:**
- [ ] Returns case metadata correctly
- [ ] All parties and attorneys included
- [ ] Filing events in correct order
- [ ] Document CMSIDs match
- [ ] Security settings correct
- [ ] Response time < 5 seconds

**GetDocument:**
- [ ] Returns valid Base64-encoded PDF
- [ ] Security enforced correctly
- [ ] 403 returned for unauthorized access
- [ ] 404 returned for missing documents
- [ ] Response time < 5 seconds

**NotifyCaseEvent:**
- [ ] Events sent after database commit
- [ ] Security events sent immediately
- [ ] EventType matches change type
- [ ] EventID correct for CaseFiling/DocumentSecurity
- [ ] Delivery time < 2 seconds

### Certification Testing

Work with your Tyler Technical Project Manager (TPM) to:
1. Test all EventTypes
2. Verify GetCase data accuracy
3. Test GetDocument security enforcement
4. Performance testing under load
5. Error handling validation
6. End-to-end workflow testing

---

## Troubleshooting

### Common Issues

**NotifyCaseEvent not triggering updates:**
- Check event sent after database commit
- Verify EventType is correct
- Check RabbitMQ message delivery
- Review re:Search logs for errors

**GetCase returning incomplete data:**
- Verify all required fields populated
- Check for missing CMSIDs on documents
- Validate ActivityDate on all filing events
- Review PageCount on documents

**GetDocument returning errors:**
- Verify CMSID matches GetCase response
- Check document file exists in storage
- Validate security enforcement logic
- Test Base64 encoding

**Performance issues:**
- Add database indexes on CMSIDs
- Optimize GetCase query
- Check file storage speed
- Review network latency

---

## Implementation Workflow

### Recommended Order

**Phase 1: GetCase (Week 1-2)**
1. Implement GetCase endpoint
2. Return case metadata
3. Add parties and filing events
4. Include document metadata
5. Test with Tyler team

**Phase 2: GetDocument (Week 3)**
1. Implement GetDocument endpoint
2. Add security enforcement
3. Test document retrieval
4. Validate Base64 encoding

**Phase 3: NotifyCaseEvent (Week 4)**
1. Identify trigger points in CMS
2. Implement event sending
3. Map EventTypes to changes
4. Test all event scenarios

**Phase 4: Integration Testing (Week 5)**
1. End-to-end testing
2. Performance validation
3. Security testing
4. Error handling verification

**Phase 5: Certification (Week 6)**
1. Work with Tyler TPM
2. Fix any issues
3. Production deployment
4. Go-live

---

## Related Documentation

### For Integration Process
→ [Integration Guide](../integration-guide/README.md) - Step-by-step onboarding process

### For Implementation Best Practices
→ [Implementation Playbook](../implementation-playbook/README.md) - Architecture patterns and coding standards

### For Market-Specific Rules
→ [Market Standards](../market-standards/README.md) - Texas JCIT, Illinois AOIC, etc.

### For Problem Solving
→ [Troubleshooting Guide](../troubleshooting/README.md) - Common issues and solutions

---

## Support & Resources

**Technical Questions:**
Contact your assigned Tyler Technical Project Manager (TPM)

**XML Schema Files:**
Available from your TPM

**Sample Data:**
XML examples available in each API's `examples/` directory:
- [GetCase examples](./getcase/examples/)
- [GetDocument examples](./getdocument/examples/)
- [NotifyCaseEvent examples](./notifycaseevent/examples/)

**Best Practices:**
Review the [Implementation Playbook](../implementation-playbook/README.md)

---

## Document Status

| API | Status | Last Updated |
|-----|--------|--------------|
| GetCase | ✅ Complete | December 2024 |
| GetDocument | ✅ Complete | December 2024 |
| NotifyCaseEvent | ✅ Complete | December 2024 |
| RecordFiling | 📝 In Progress | TBD |
| NotifyDocketingComplete | 📝 In Progress | TBD |

---

**Last Updated:** December 2024