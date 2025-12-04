# Event Types Guide

[← Back to NotifyCaseEvent Overview](./README.md)

Complete reference of all supported EventTypes for NotifyCaseEvent.

## Quick Navigation
- [EventType Mapping](#eventtype-mapping)
- [All Supported EventTypes](#all-supported-eventtypes)
- [When to Use Each Type](#when-to-use-each-type)
- [EventType Decision Tree](#eventtype-decision-tree)

---

## EventID Requirements

**Critical:** The EventID value you send depends on the EventType. Some EventTypes require specific business object identifiers.

### EventTypes That Use EventID

| EventType | EventID Must Contain | re:Search Uses It For |
|-----------|---------------------|----------------------|
| `CaseFiling` | **Filing CMSID** (unique identifier for this filing) | Filters GetCase to return ONLY this filing |
| `DocumentSecurity` | **Document CMSID** | Calls GetDocument with this ID |
| `DocumentFiling` | **Document CMSID** | Calls GetDocument with this ID |
| `CaseHearing` | **Hearing CMSID** | Filters GetCase to return ONLY this hearing |

### EventTypes That Ignore EventID

| EventType | EventID Can Be | re:Search Behavior |
|-----------|---------------|-------------------|
| `CaseParty` | Any unique value (case CMSID typical) | Ignored - GetCase returns all parties |
| `CasePartyAttorney` | Any unique value | Ignored - GetCase returns all attorneys |
| `CaseSecurity` | Any unique value (case CMSID typical) | Ignored - GetCase returns case security |
| `CaseDisposition` | Any unique value (case CMSID typical) | Ignored - GetCase returns disposition |
| `CaseReassigned` | Any unique value (case CMSID typical) | Ignored - GetCase returns court/judge |
| `CaseStatus` | Any unique value (case CMSID typical) | Ignored - GetCase returns status |
| `CaseDelete` | Any unique value (case CMSID typical) | Ignored - GetCase returns case |
| `CaseExpunge` | Any unique value (case CMSID typical) | Ignored - GetCase returns case |

### Critical EventID Rules

**CaseFiling - MUST be Filing CMSID:**
```xml
<Event>
  <ID>FIL-20241204-001</ID> ← Filing CMSID from your CMS
  <EventType>CaseFiling</EventType>
</Event>
```

**Why:** re:Search sends GetCase with `DocketEntryTypeCodeFilterText` set to this ID. GetCase response should contain **ONLY this filing** - any other filing data could prevent proper updates.

**DocumentSecurity / DocumentFiling - MUST be Document CMSID:**
```xml
<Event>
  <ID>DOC-20241204-789</ID> ← Document CMSID from your CMS
  <EventType>DocumentSecurity</EventType>
</Event>
```

**Why:** re:Search calls GetDocument with `DocumentFileControlID` set to this value.

**CaseHearing - MUST be Hearing CMSID:**
```xml
<Event>
  <ID>HRG-20241204-456</ID> ← Hearing CMSID from your CMS
  <EventType>CaseHearing</EventType>
</Event>
```

**Why:** re:Search sends GetCase with `DocketEntryTypeCodeFilterText` set to this ID. GetCase response should contain **ONLY this hearing** - any other hearing data could prevent proper updates.

---

## EventType Mapping

**Critical concept:** The EventType you send determines which blocks re:Search processes in the GetCase response.

| EventType | What GetCase Processes | What Gets Ignored |
|-----------|------------------------|-------------------|
| `CaseFiling` | FilingEvent, DocumentMetadata | Parties, Security, Disposition |
| `CaseParty` | CaseParticipant blocks | Filings, Documents, Security |
| `CasePartyAttorney` | CaseAttorney blocks | Filings, Parties, Security |
| `CaseSecurity` | tyler:CaseSecurity **only** | Everything else |
| `DocumentSecurity` | tyler:DocumentSecurity **only** | Everything else |
| `CaseDisposition` | Disposition block | Everything else |
| `CaseReassigned` | Court/judge header | Everything else |
| `CaseStatus` | CaseStatus block | Everything else |
| `CaseInitiated` | **Everything** | Nothing (processes complete case) |

---

## All Supported EventTypes

### Case-Level Events

| EventType | When to Send | Priority |
|-----------|--------------|----------|
| `CaseInitiated` | New case created | Normal |
| `CaseFiling` | Filing added/docketed | Normal |
| `CaseParty` | Party added/changed/removed | Normal |
| `CasePartyAttorney` | Attorney added/changed/removed | Normal |
| `CaseSecurity` | Case sealed/unsealed | **Critical** |
| `CaseDisposition` | Disposition entered | Normal |
| `CaseReassigned` | Judge/division changed | Normal |
| `CaseStatus` | Case status changed | Normal |
| `CaseDelete` | Case deleted/restored | Normal |

### Document-Level Events

| EventType | When to Send | Priority |
|-----------|--------------|----------|
| `DocumentFiling` | Document added (non-e-filing) | Normal |
| `DocumentSecurity` | Document sealed/restricted | **Critical** |

### Not Implemented

⚠️ **The following are NOT supported:**
- `LoadAllCaseParties` - Use `CaseParty` instead
- `LoadAllCaseEvents` - Use `CaseInitiated` instead

---

## When to Use Each Type

### CaseInitiated
**Use when:** New case created or bulk updates to entire case

**Example scenarios:**
- Brand new case filed
- Case migrated from old system
- Multiple changes at once (party + filing + disposition)

**What happens:**
- re:Search processes entire GetCase response
- All blocks updated (parties, filings, security, etc.)

---

### CaseFiling
**Use when:** New filing added to case

**Example scenarios:**
- E-filing docketed
- Manual filing entered by clerk
- Document added to existing filing

**EventID requirement:** **MUST be the Filing CMSID** (unique identifier for this specific filing in your CMS)

**Example:**
```xml
<Event>
  <ID>FIL-20241204-001</ID> ← Your CMS's filing identifier
  <EventType>CaseFiling</EventType>
</Event>
```

**What happens:**
- re:Search processes only FilingEvent and DocumentMetadata blocks
- re:Search filters GetCase to return **ONLY this filing**
- Any other filing data in GetCase response could prevent proper updates
- Parties, security, etc. ignored

**Critical:** The EventID is used as `DocketEntryTypeCodeFilterText` in GetCase request.

---

### CaseParty
**Use when:** Party information changes

**Example scenarios:**
- New party added
- Party name/address updated
- Party removed

**What happens:**
- re:Search processes only CaseParticipant blocks
- Filings, documents ignored

---

### CasePartyAttorney
**Use when:** Attorney information changes

**Example scenarios:**
- Attorney assigned to party
- Attorney contact info updated
- Attorney withdrawn

**What happens:**
- re:Search processes only CaseAttorney blocks
- Parties, filings ignored

---

### CaseSecurity
**Use when:** Case-level security changes

**Example scenarios:**
- Case sealed by court order
- Sealed case unsealed
- Security level changed

**What happens:**
- re:Search processes ONLY tyler:CaseSecurity
- Everything else ignored
- **Processed immediately** (high priority)

**⚠️ CRITICAL:** Never batch or queue security events!

---

### DocumentSecurity
**Use when:** Individual document security changes

**Example scenarios:**
- Document sealed within public case
- Document unsealed
- Document access restricted

**EventID requirement:** **MUST be the Document CMSID** (unique identifier for this specific document in your CMS)

**Example:**
```xml
<Event>
  <ID>DOC-20241204-789</ID> ← Your CMS's document identifier
  <EventType>DocumentSecurity</EventType>
</Event>
```

**What happens:**
- re:Search calls GetDocument with `DocumentFileControlID` set to this EventID
- re:Search processes ONLY tyler:DocumentSecurity
- Everything else ignored
- **Processed immediately** (high priority)

**⚠️ CRITICAL:** Never batch or queue DocumentSecurity events!

---

### CaseDisposition
**Use when:** Disposition entered or changed

**Example scenarios:**
- Case dismissed
- Judgment entered
- Disposition type changed

**What happens:**
- re:Search processes only disposition block
- Filings, parties, security ignored

---

### CaseReassigned
**Use when:** Judge or division assignment changes

**Example scenarios:**
- Judge reassigned
- Case moved to different division
- Court location changed

**What happens:**
- re:Search processes court/judge header
- Everything else ignored

---

### CaseStatus
**Use when:** Case status changes

**Example scenarios:**
- Case opened → active
- Active → closed
- Reopened

**What happens:**
- re:Search processes status block
- Everything else ignored

---

## EventType Decision Tree

```
What changed in the case?

├─ New case created?
│  └─ Use: CaseInitiated
│
├─ Filing/document added?
│  └─ Use: CaseFiling
│
├─ Party information?
│  ├─ Party itself changed? → Use: CaseParty
│  └─ Attorney changed? → Use: CasePartyAttorney
│
├─ Security changed?
│  ├─ Entire case? → Use: CaseSecurity (IMMEDIATE!)
│  └─ Just one document? → Use: DocumentSecurity (IMMEDIATE!)
│
├─ Disposition entered?
│  └─ Use: CaseDisposition
│
├─ Judge/division changed?
│  └─ Use: CaseReassigned
│
├─ Status changed?
│  └─ Use: CaseStatus
│
└─ Multiple things changed?
   └─ Use: CaseInitiated (processes everything)
```

---

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Wrong EventType | Changes ignored | Use correct type or CaseInitiated |
| Using CaseFiling for party | Party not updated | Use CaseParty |
| Using CaseParty for filing | Filing not updated | Use CaseFiling |
| Queuing security events | Delay in hiding sealed cases | Send immediately |
| Using unsupported type | Event rejected | Check supported types |

---

[View request/response details →](./request-response.md)
[Return to overview →](./README.md)

---

## XML Examples

The [examples directory](./examples/) contains complete NotifyCaseEvent XML files for each EventType:

| File | EventType | Shows |
|------|-----------|-------|
| [case-filing.xml](./examples/case-filing.xml) | CaseFiling | Filing docketed, EventID = Filing CMSID |
| [case-party.xml](./examples/case-party.xml) | CaseParty | Party added to case |
| [case-party-attorney.xml](./examples/case-party-attorney.xml) | CasePartyAttorney | Attorney assigned |
| [case-security.xml](./examples/case-security.xml) | CaseSecurity | Case sealed (critical/immediate) |
| [case-disposition.xml](./examples/case-disposition.xml) | CaseDisposition | Disposition entered |
| [case-reassigned.xml](./examples/case-reassigned.xml) | CaseReassigned | Judge changed |
| [case-delete.xml](./examples/case-delete.xml) | CaseDelete | Case deleted |
| [case-expunge.xml](./examples/case-expunge.xml) | CaseExpunge | Case expunged |
| [document-filing.xml](./examples/document-filing.xml) | DocumentFiling | Document filed, EventID = Document CMSID |
| [document-security.xml](./examples/document-security.xml) | DocumentSecurity | Document sealed, EventID = Document CMSID |
| [generic-example.xml](./examples/generic-example.xml) | Multiple events | Shows multiple events in one request |

**Use these as templates** for your NotifyCaseEvent implementation.

---