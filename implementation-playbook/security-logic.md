# Security Logic

> **Breadcrumb:** [Home](../README.md) > [Implementation Playbook](./README.md) > Security Logic

Understand how re:Search enforces case and document security to protect confidential information while providing appropriate public access.

---

## Security Principles

### Layered Security Model

re:Search uses a multi-layer security approach:
```
Layer 1: Case-Level Security (CMS defines)
   ↓
Layer 2: Document-Level Security (CMS defines)
   ↓
Layer 3: Market-Specific Rules (re:Search applies)
   ↓
Layer 4: User Role Check (at search time)
   ↓
Final: Access Granted or Denied
```

**Key Principle:** Security can become MORE restrictive at each layer, but never LESS restrictive.

---

## Case-Level Security

### Security Levels

Your CMS sets case-level security in the GetCase response. re:Search recognizes three primary levels:

#### Public (Default)
**Visibility:** Available to all users

**When to use:**
- Standard civil cases
- Public criminal proceedings
- Cases with no confidentiality concerns
- Default for most case types

**GetCase XML:**
```xml
<Case>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <CaseSecurity>
    <SecurityLevel>Public</SecurityLevel>
  </CaseSecurity>
</Case>
```

**User Access:**
- ✅ Public users can search and view
- ✅ Attorneys can view
- ✅ Registered users can view
- ✅ Court staff can view

---

#### Sealed
**Visibility:** Restricted access, not visible to public

**When to use:**
- Court order to seal case
- Confidential proceedings
- Cases with privacy concerns
- Cases under protective order

**GetCase XML:**
```xml
<Case>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <CaseSecurity>
    <SecurityLevel>Sealed</SecurityLevel>
    <SealDate>2024-01-15</SealDate>
  </CaseSecurity>
</Case>
```

**User Access:**
- ❌ Public users cannot view
- ⚠️ Attorneys may view if they represent a party
- ⚠️ Registered users subject to market rules
- ✅ Court staff can view

**Publication Delays:**
Some markets apply delays to sealed cases (e.g., Texas 180-day rule)

---

#### Confidential
**Visibility:** Highly restricted, limited to specific roles

**When to use:**
- Juvenile cases
- Mental health proceedings
- Adoption cases
- Cases with statutory confidentiality

**GetCase XML:**
```xml
<Case>
  <CaseTrackingID>JV-2024-000567</CaseTrackingID>
  <CaseSecurity>
    <SecurityLevel>Confidential</SecurityLevel>
  </CaseSecurity>
</Case>
```

**User Access:**
- ❌ Public users cannot view
- ❌ General attorneys cannot view
- ⚠️ Attorneys of record may view
- ✅ Court staff can view

---

### Case Security Inheritance

**Rule:** All documents in a case inherit the case's security level by default.

**Example:**
```
Case: CV-2024-001234
  CaseSecurity: Public
  
  Document 1: Complaint
    (inherits Public from case)
    
  Document 2: Motion to Seal
    (inherits Public from case)
    
  Document 3: Sealed Exhibit
    DocumentSecurity: Sealed (overrides case level)
```

**Important:** Document security can be MORE restrictive, never LESS restrictive than case security.

---

## Document-Level Security

### Overriding Case Security

Documents can have their own security level that overrides the case-level security.

#### Sealed Document in Public Case

**GetCase XML:**
```xml
<Case>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <CaseSecurity>
    <SecurityLevel>Public</SecurityLevel>
  </CaseSecurity>
  <CaseDocuments>
    <Document>
      <DocumentID>DOC-001</DocumentID>
      <DocumentTitle>Motion for Protective Order</DocumentTitle>
      <!-- This document is public (inherits from case) -->
    </Document>
    <Document>
      <DocumentID>DOC-002</DocumentID>
      <DocumentTitle>Confidential Medical Records</DocumentTitle>
      <DocumentSecurity>
        <SecurityLevel>Sealed</SecurityLevel>
      </DocumentSecurity>
      <!-- This document is sealed (overrides case security) -->
    </Document>
  </CaseDocuments>
</Case>
```

**Result:**
- Case is searchable by public
- DOC-001 is viewable by public
- DOC-002 is NOT viewable by public (sealed)

---

#### Invalid: Public Document in Sealed Case

**This is NOT allowed:**
```xml
<Case>
  <CaseSecurity>
    <SecurityLevel>Sealed</SecurityLevel>
  </CaseSecurity>
  <CaseDocuments>
    <Document>
      <DocumentSecurity>
        <SecurityLevel>Public</SecurityLevel>  <!-- ❌ INVALID -->
      </DocumentSecurity>
    </Document>
  </CaseDocuments>
</Case>
```

**Why:** You cannot make documents MORE visible than the case. If case is sealed, all documents must be at least sealed.

**re:Search Behavior:** Will treat document as sealed (uses most restrictive level)

---

### Document Security Use Cases

#### Scenario 1: Trade Secret Protection

**Situation:** Civil case with confidential business documents
```xml
<Case>
  <CaseSecurity>
    <SecurityLevel>Public</SecurityLevel>
  </CaseSecurity>
  <CaseDocuments>
    <Document>
      <DocumentTitle>Complaint</DocumentTitle>
      <!-- Public -->
    </Document>
    <Document>
      <DocumentTitle>Protective Order</DocumentTitle>
      <!-- Public -->
    </Document>
    <Document>
      <DocumentTitle>Confidential Financial Records</DocumentTitle>
      <DocumentSecurity>
        <SecurityLevel>Sealed</SecurityLevel>
      </DocumentSecurity>
      <!-- Sealed -->
    </Document>
  </CaseDocuments>
</Case>
```

**Result:** Public can see case and most documents, but not confidential records.

---

#### Scenario 2: Victim Protection

**Situation:** Criminal case with victim information to protect
```xml
<Case>
  <CaseSecurity>
    <SecurityLevel>Public</SecurityLevel>
  </CaseSecurity>
  <CaseDocuments>
    <Document>
      <DocumentTitle>Indictment</DocumentTitle>
      <!-- Public -->
    </Document>
    <Document>
      <DocumentTitle>Victim Impact Statement</DocumentTitle>
      <DocumentSecurity>
        <SecurityLevel>Confidential</SecurityLevel>
      </DocumentSecurity>
      <!-- Confidential - highly restricted -->
    </Document>
  </CaseDocuments>
</Case>
```

**Result:** Public can see case and indictment, but victim statement is confidential.

---

## Market-Specific Security Rules

Different jurisdictions have additional security requirements beyond basic sealed/public settings.

### Texas (JCIT) Security Rules

#### Publication Delays

**31-Day Delay (Criminal Cases):**
```
Criminal case filed → 31-day delay period → Then visible to public
```

**Implementation:**
```xml
<Case>
  <CaseType>CR</CaseType>  <!-- Criminal -->
  <CaseSecurity>
    <SecurityLevel>Public</SecurityLevel>
  </CaseSecurity>
  <FilingDate>2024-01-15</FilingDate>
</Case>
```

**re:Search Processing:**
- Case indexed immediately
- Public access delayed until 2024-02-15 (31 days)
- Court staff can see immediately
- Registered users may see earlier (per JCIT rules)

---

**180-Day Delay (Sealed Documents):**
```
Document sealed → 180-day delay period → Then may become available
```

**Implementation:**
```xml
<Document>
  <DocumentSecurity>
    <SecurityLevel>Sealed</SecurityLevel>
    <SealDate>2024-01-15</SealDate>
  </DocumentSecurity>
</Document>
```

**re:Search Processing:**
- Document sealed immediately
- Public access restricted
- After 180 days, may become available (if seal lifted)
- Court order required to lift seal

---

#### JCIT Roles and Visibility

**Four Primary Roles:**

1. **Public** - General public users
2. **Attorney** - Licensed attorneys (may have case-specific access)
3. **Registered User** - County/court-registered users
4. **Court Staff** - Court employees

**Visibility Matrix Example:**

| Case Type | Public | Attorney | Registered User | Court Staff |
|-----------|--------|----------|-----------------|-------------|
| Criminal (< 31 days) | ❌ | ❌ | ⚠️ (County only) | ✅ |
| Criminal (> 31 days) | ✅ | ✅ | ✅ | ✅ |
| Family | ⚠️ Limited | ✅ (if involved) | ⚠️ (County only) | ✅ |
| Civil | ✅ | ✅ | ✅ | ✅ |
| Juvenile | ❌ | ⚠️ (if appointed) | ❌ | ✅ |

[Complete Texas security requirements →](../market-standards/texas/security/README.md)

---

### Illinois (AOIC) Security Rules

Illinois has its own security standards defined by AOIC.

**Key Differences:**
- Different publication delay rules (if any)
- Different role definitions
- State-specific access matrix

[Complete Illinois security requirements →](../market-standards/illinois/security/README.md)

---

### Other Markets

Markets without specific standards bodies follow universal re:Search security:
- Case-level security (Public/Sealed/Confidential)
- Document-level overrides
- Basic role-based access (Public/Attorney/Court Staff)

[Other market security →](../market-standards/other-markets/README.md)

---

## Implementing Security in Your CMS

### GetCase Security Implementation

**Checklist for GetCase responses:**
```
✅ Include CaseSecurity element for every case
✅ Set SecurityLevel based on CMS status
✅ Include SealDate if case is sealed
✅ Set DocumentSecurity for restricted documents
✅ Ensure document security never less restrictive than case
✅ Include all security metadata (dates, reasons, etc.)
```

**Example GetCase Response:**
```xml
<GetCaseResponse>
  <Case>
    <CaseTrackingID>FM-2024-003456</CaseTrackingID>
    <CaseType>FM</CaseType>
    <FilingDate>2024-01-15</FilingDate>
    
    <CaseSecurity>
      <SecurityLevel>Public</SecurityLevel>
      <!-- Family case, public but with restricted documents -->
    </CaseSecurity>
    
    <CaseParties>
      <Party>
        <PartyName>John Smith</PartyName>
        <!-- Party info visible with case -->
      </Party>
    </CaseParties>
    
    <CaseDocuments>
      <Document>
        <DocumentID>DOC-001</DocumentID>
        <DocumentTitle>Petition for Divorce</DocumentTitle>
        <!-- Public (inherits from case) -->
      </Document>
      
      <Document>
        <DocumentID>DOC-002</DocumentID>
        <DocumentTitle>Child Custody Evaluation</DocumentTitle>
        <DocumentSecurity>
          <SecurityLevel>Confidential</SecurityLevel>
        </DocumentSecurity>
        <!-- Confidential (protects child information) -->
      </Document>
      
      <Document>
        <DocumentID>DOC-003</DocumentID>
        <DocumentTitle>Financial Affidavit</DocumentTitle>
        <DocumentSecurity>
          <SecurityLevel>Sealed</SecurityLevel>
        </DocumentSecurity>
        <!-- Sealed (protects financial information) -->
      </Document>
    </CaseDocuments>
  </Case>
</GetCaseResponse>
```

---

### GetDocument Security Implementation

**Critical:** GetDocument must enforce the same security as GetCase.

**Security Check Flow:**
```
1. re:Search calls GetDocument for DOC-002
   ↓
2. CMS checks document security
   ↓
3. Document is Confidential
   ↓
4. CMS checks requesting user role
   ↓
5. User is Public (not authorized)
   ↓
6. CMS returns SOAP Fault (access denied)
```

**Example GetDocument Security Check:**
```python
def get_document(document_id, user_role):
    # Retrieve document metadata
    doc = db.get_document(document_id)
    
    # Check document security
    if doc.security_level == 'Confidential':
        if user_role not in ['CourtStaff', 'AuthorizedAttorney']:
            raise AccessDeniedError("Document is confidential")
    
    if doc.security_level == 'Sealed':
        if user_role not in ['CourtStaff', 'AttorneyOfRecord']:
            raise AccessDeniedError("Document is sealed")
    
    # Security passed, return document
    return doc.get_pdf()
```

**SOAP Fault Response:**
```xml
<soap:Fault>
  <faultcode>soap:Client</faultcode>
  <faultstring>Access Denied</faultstring>
  <detail>
    <ErrorCode>403</ErrorCode>
    <ErrorMessage>Document is sealed and not accessible to public users</ErrorMessage>
  </detail>
</soap:Fault>
```

---

## Security Event Handling

### When Security Changes

**Always send security events immediately:**

#### Case Sealed
```
1. Judge orders case sealed
   ↓
2. CMS updates case security to Sealed
   ↓
3. CMS sends CaseSecurity event IMMEDIATELY
   ↓
4. re:Search calls GetCase
   ↓
5. re:Search applies Sealed security
   ↓
6. Case hidden from public immediately
```

**Event XML:**
```xml
<NotifyCaseEvent>
  <EventType>CaseSecurity</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
</NotifyCaseEvent>
```

**Timing Critical:** Do NOT delay security events. They must be sent immediately.

---

#### Document Sealed
```
1. Court orders document sealed
   ↓
2. CMS updates document security to Sealed
   ↓
3. CMS sends DocumentSecurity event IMMEDIATELY
   ↓
4. re:Search calls GetCase
   ↓
5. re:Search updates document security
   ↓
6. Document hidden from public immediately
```

**Event XML:**
```xml
<NotifyCaseEvent>
  <EventType>DocumentSecurity</EventType>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
</NotifyCaseEvent>
```

---

#### Case Unsealed
```
1. Seal period expires or court order lifts seal
   ↓
2. CMS updates case security to Public
   ↓
3. CMS sends CaseSecurity event
   ↓
4. re:Search calls GetCase
   ↓
5. re:Search applies Public security
   ↓
6. Case visible to public
```

**Important:** Check market rules for unsealing requirements. Some jurisdictions require court order even after delay period.

---

## Security Testing

### Test Scenarios

#### Test 1: Public Case Visibility
```
Setup:
  - Create public case
  - Send CaseFiling event
  
Expected:
  ✅ Public users can search and find case
  ✅ All users can view case details
  ✅ Public users can view documents
```

---

#### Test 2: Sealed Case Restriction
```
Setup:
  - Create sealed case
  - Send CaseFiling event
  
Expected:
  ❌ Public users cannot find case in search
  ✅ Court staff can view case
  ⚠️ Attorneys of record can view (if market allows)
```

---

#### Test 3: Mixed Document Security
```
Setup:
  - Create public case
  - Add public document (DOC-001)
  - Add sealed document (DOC-002)
  - Send DocumentFiling event
  
Expected:
  ✅ Public users can view case
  ✅ Public users can view DOC-001
  ❌ Public users cannot view DOC-002
  ✅ Court staff can view all documents
```

---

#### Test 4: Case Sealing Event
```
Setup:
  - Create public case
  - Case is searchable
  - Send CaseSecurity event (seal case)
  
Expected:
  ❌ Case immediately hidden from public
  ✅ Case still visible to court staff
  ✅ Previously visible case now restricted
```

---

#### Test 5: Publication Delay (Texas)
```
Setup:
  - Create criminal case (Texas)
  - Set FilingDate = today
  - Send CaseFiling event
  
Expected:
  ❌ Public users cannot see case (31-day delay)
  ✅ Court staff can see case immediately
  ⚠️ Registered users in county can see case
  ✅ After 31 days, public users can see case
```

---

## Common Security Mistakes

### ❌ Mistake 1: Not Including Security Metadata
**Problem:** GetCase response omits CaseSecurity element

**Result:** re:Search may treat case as public by default

**Solution:** Always include CaseSecurity in GetCase response

---

### ❌ Mistake 2: Document Security Less Restrictive Than Case
**Problem:** Sealed case with public documents

**Result:** re:Search applies most restrictive level (ignores invalid public setting)

**Solution:** Ensure document security >= case security

---

### ❌ Mistake 3: GetDocument Doesn't Check Security
**Problem:** GetDocument returns sealed documents to public users

**Result:** Confidential information exposed inappropriately

**Solution:** Always check security before returning document

---

### ❌ Mistake 4: Delayed Security Events
**Problem:** Case sealed but CaseSecurity event sent hours later

**Result:** Sealed case visible to public during delay

**Solution:** Send security events immediately when security changes

---

### ❌ Mistake 5: Inconsistent Security Between GetCase and GetDocument
**Problem:** GetCase says document is sealed, but GetDocument returns it to public

**Result:** Security bypass, confidential information exposed

**Solution:** Ensure GetDocument enforces same security as GetCase

---

### ❌ Mistake 6: Not Handling Market-Specific Rules
**Problem:** Texas criminal case visible immediately (ignoring 31-day delay)

**Result:** Non-compliance with JCIT standards

**Solution:** Understand and implement market-specific security rules

---

## Security Monitoring

### What to Monitor

**Case Security Events:**
- Number of cases sealed per day/week
- Cases unsealed (track seal period compliance)
- Security events sent
- Security event processing time

**Document Security:**
- Sealed documents per case type
- Document access denials
- Documents with security overrides
- Confidential document access patterns

**Access Violations:**
- Attempts to access sealed cases
- GetDocument access denials
- Invalid security configurations
- Security event failures

**Market Compliance:**
- Cases subject to publication delays
- Delay period adherence
- Cases unsealed appropriately
- Role-based access patterns

---

## Security Audit Trail

### Recommended Logging

**Security Changes:**
```
Log:
  - Timestamp of security change
  - User who made change
  - Old security level
  - New security level
  - Reason for change (if available)
  - CaseSecurity/DocumentSecurity event sent
```

**Access Attempts:**
```
Log:
  - Timestamp of access attempt
  - User role attempting access
  - Case/Document accessed
  - Security level of case/document
  - Access granted or denied
  - Reason for denial (if applicable)
```

**Event Transmission:**
```
Log:
  - Timestamp of event sent
  - Event type (CaseSecurity/DocumentSecurity)
  - Case tracking ID
  - Success or failure
  - Response received
```

---

## Related Documentation

### For API Implementation
→ [GetCase API](../api-reference/getcase/README.md)  
→ [GetDocument API](../api-reference/getdocument/README.md)  
→ [NotifyCaseEvent API](../api-reference/notifycaseevent/README.md)

### For Market Rules
→ [Texas Security](../market-standards/texas/security/README.md)  
→ [Illinois Security](../market-standards/illinois/security/README.md)

### For Implementation Patterns
→ [Best Practices](./best-practices.md)  
→ [Event Type Logic](./event-type-logic.md)

### For Troubleshooting
→ [Security Troubleshooting](../troubleshooting/security-and-authentication.md)

---

**Last Updated:** January 2025