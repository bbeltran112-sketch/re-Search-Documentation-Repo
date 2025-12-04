# GetDocument Security Guide

[← Back to GetDocument Overview](./README.md)

This guide defines how CMS must enforce security when returning documents via GetDocument.

## Quick Navigation
- [Security Enforcement Rules](#security-enforcement-rules)
- [Case vs Document Security](#case-vs-document-security)
- [User Authorization](#user-authorization)
- [Security Decision Tree](#security-decision-tree)
- [Common Security Mistakes](#common-security-mistakes)
- [Certification Checklist](#certification-checklist)

---

## Core Security Principle

> **GetDocument MUST enforce security - never return documents the user shouldn't access.**

Even if a document appears in search results, GetDocument must verify authorization before returning the binary content.

---

## Security Enforcement Rules

### Rule 1: Most Restrictive Security Wins

When case security and document security conflict, **apply the most restrictive rule**.

**Example:**
- Case Security: `PublicFilingPublicView` (public)
- Document Security: `RestrictedView` (restricted)
- **Effective Security:** `RestrictedView` (more restrictive)

### Rule 2: Case Security Overrides Document Security

If the entire case is sealed, **all documents are sealed** regardless of document-level settings.

**Example:**
- Case Security: `SealedCase`
- Document Security: `PublicView`
- **Effective Security:** `SealedCase` (case security wins)

### Rule 3: Never Skip Security Checks

**Always validate authorization** even if:
- Document appears in public search results
- Document was previously accessible
- User accessed similar documents

Security settings can change at any time!

---

## Case vs Document Security

### Case-Level Security

Set in GetCase response as `tyler:CaseSecurity`:

| Value | Meaning | Default Document Access |
|-------|---------|-------------------------|
| `PublicFilingPublicView` | Public case, public filings | All documents public (unless document security restricts) |
| `PublicFilingRestrictedView` | Public case, some filings restricted | Documents vary by document security |
| `SealedCase` | Entire case sealed | **All documents hidden from public** |
| `ExpungedCase` | Case expunged | **All documents hidden from everyone** |

### Document-Level Security

Set in GetCase response as `tyler:DocumentSecurity`:

| Value | Meaning | Who Can Access |
|-------|---------|----------------|
| `PublicView` | Public document | Everyone (unless case is sealed) |
| `RestrictedView` | Restricted document | Attorneys of record, court staff |
| `NoAccess` | No public access | Court staff only |

---

## Security Conflict Resolution

### Resolution Matrix

| Case Security | Document Security | Effective Security | Public Access? |
|---------------|-------------------|-------------------|----------------|
| `PublicFilingPublicView` | `PublicView` | **Public** | ✅ Yes |
| `PublicFilingPublicView` | `RestrictedView` | **Restricted** | ❌ No (attorneys only) |
| `PublicFilingPublicView` | `NoAccess` | **NoAccess** | ❌ No (staff only) |
| `PublicFilingRestrictedView` | `PublicView` | **Public** | ✅ Yes |
| `PublicFilingRestrictedView` | `RestrictedView` | **Restricted** | ❌ No (attorneys only) |
| `SealedCase` | Any | **Sealed** | ❌ No (case sealed) |
| `ExpungedCase` | Any | **Expunged** | ❌ No (removed) |

### Decision Logic

```
Step 1: Is case sealed or expunged?
        YES → Deny access (unless user is court staff)
        NO → Continue to Step 2

Step 2: What is document security?
        - PublicView → Allow (if user authorized)
        - RestrictedView → Check if user is attorney of record
        - NoAccess → Deny (unless court staff)

Step 3: Combine case + document security
        Apply most restrictive rule
```

---

## User Authorization

### User Roles

| Role | Typical Access |
|------|---------------|
| **Public** | Public documents only |
| **Attorney** | Public + their cases' restricted docs |
| **Court Staff** | All documents (including sealed) |
| **Judge** | All documents in their cases |

### Authorization Checks

**For Public Users:**
```
1. Case must be public (not sealed/expunged)
2. Document must have PublicView security
3. No additional restrictions
```

**For Attorneys:**
```
1. If document is public → Allow
2. If document is restricted → Check attorney of record
3. If case is sealed → Check attorney of record
4. If no attorney relationship → Deny
```

**For Court Staff:**
```
1. Allow access to all documents
2. Including sealed and expunged cases
3. No additional checks needed
```

---

## Security Decision Tree

```
GetDocument request received
    ↓
Retrieve document metadata
    ↓
Get case data for security context
    ↓
Determine effective security
    ↓
Is case sealed or expunged?
├─ YES → Is user court staff?
│         ├─ YES → Allow access
│         └─ NO → DENY ACCESS (403)
│
└─ NO → What is document security?
         ├─ PublicView → Is case public?
         │              ├─ YES → Allow access
         │              └─ NO → DENY ACCESS (403)
         │
         ├─ RestrictedView → Is user attorney of record?
         │                   ├─ YES → Allow access
         │                   └─ NO → DENY ACCESS (403)
         │
         └─ NoAccess → Is user court staff?
                       ├─ YES → Allow access
                       └─ NO → DENY ACCESS (403)
```

---

## Implementation Example

### Complete Security Check

```csharp
public bool UserCanAccessDocument(
    User user,
    Case caseData,
    Document document)
{
    // 1. Court staff can access everything
    if (user.Role == UserRole.CourtStaff || user.Role == UserRole.Judge)
    {
        return true;
    }
    
    // 2. Check case-level security first
    if (caseData.Security == CaseSecurity.SealedCase)
    {
        // Only attorneys of record can access sealed cases
        if (user.Role == UserRole.Attorney)
        {
            return IsAttorneyOfRecord(user.Id, caseData.Id);
        }
        return false;
    }
    
    if (caseData.Security == CaseSecurity.ExpungedCase)
    {
        // Expunged cases - very limited access
        return false; // Even attorneys typically can't access
    }
    
    // 3. Check document-level security
    var docSecurity = document.Security ?? DocumentSecurity.PublicView;
    
    switch (docSecurity)
    {
        case DocumentSecurity.PublicView:
            // Public documents accessible if case is public
            return caseData.Security == CaseSecurity.PublicFilingPublicView ||
                   caseData.Security == CaseSecurity.PublicFilingRestrictedView;
        
        case DocumentSecurity.RestrictedView:
            // Only attorneys of record can access
            if (user.Role == UserRole.Attorney)
            {
                return IsAttorneyOfRecord(user.Id, caseData.Id);
            }
            return false;
        
        case DocumentSecurity.NoAccess:
            // No public access at all
            return false;
        
        default:
            // Unknown security level - deny by default
            return false;
    }
}

private bool IsAttorneyOfRecord(int attorneyId, int caseId)
{
    return _repository.IsAttorneyAssignedToCase(attorneyId, caseId);
}
```

### Python Example

```python
def user_can_access_document(user, case_data, document):
    """Check if user can access document based on security"""
    
    # 1. Court staff can access everything
    if user.role in ['CourtStaff', 'Judge']:
        return True
    
    # 2. Check case-level security
    if case_data.security == 'SealedCase':
        # Only attorneys of record
        if user.role == 'Attorney':
            return is_attorney_of_record(user.id, case_data.id)
        return False
    
    if case_data.security == 'ExpungedCase':
        # Expunged - no access
        return False
    
    # 3. Check document-level security
    doc_security = document.security or 'PublicView'
    
    if doc_security == 'PublicView':
        # Public if case is public
        return case_data.security in [
            'PublicFilingPublicView',
            'PublicFilingRestrictedView'
        ]
    
    elif doc_security == 'RestrictedView':
        # Attorneys of record only
        if user.role == 'Attorney':
            return is_attorney_of_record(user.id, case_data.id)
        return False
    
    elif doc_security == 'NoAccess':
        # No public access
        return False
    
    # Unknown security - deny by default
    return False
```

---

## Common Security Mistakes

### Mistake 1: Not Checking Case Security

**❌ Wrong:**
```csharp
// Only checking document security
if (document.Security == "PublicView")
{
    return document; // WRONG! Ignores case security
}
```

**✅ Correct:**
```csharp
// Check case security first
if (caseData.Security == "SealedCase")
{
    // Check authorization even if doc is "public"
    if (!UserCanAccess(user, caseData))
    {
        throw new AccessDeniedException();
    }
}
```

### Mistake 2: Assuming Public Search = Public Access

**❌ Wrong:**
```csharp
// Document appears in search, so must be public
return GetDocument(documentId); // WRONG!
```

**✅ Correct:**
```csharp
// Always validate security in GetDocument
var caseData = GetCase(document.CaseId);
if (!UserCanAccessDocument(user, caseData, document))
{
    throw new AccessDeniedException();
}
return GetDocument(documentId);
```

### Mistake 3: Caching Security Decisions

**❌ Wrong:**
```csharp
// Cache user's access decision
if (_cache.HasAccess(userId, documentId))
{
    return document; // WRONG! Security can change
}
```

**✅ Correct:**
```csharp
// Always check security fresh on each request
var caseData = GetCurrentCaseData(document.CaseId);
if (!UserCanAccessDocument(user, caseData, document))
{
    throw new AccessDeniedException();
}
```

### Mistake 4: Returning Empty Content Instead of Error

**❌ Wrong:**
```csharp
if (!authorized)
{
    return new Document { BinaryContent = null }; // WRONG!
}
```

**✅ Correct:**
```csharp
if (!authorized)
{
    throw new AccessDeniedException(
        "User does not have permission to access this document");
}
```

### Mistake 5: Not Logging Access Denials

**❌ Wrong:**
```csharp
if (!authorized)
{
    return 403; // WRONG! No audit trail
}
```

**✅ Correct:**
```csharp
if (!authorized)
{
    _logger.LogWarning(
        "Access denied: User {UserId} attempted to access " +
        "sealed document {DocumentId} in case {CaseId}",
        user.Id, document.Id, caseData.Id);
    
    throw new AccessDeniedException();
}
```

---

## Testing Security

### Test Scenarios

**1. Public Document in Public Case**
- User: Public
- Case: PublicFilingPublicView
- Document: PublicView
- **Expected:** Access granted

**2. Restricted Document in Public Case**
- User: Public
- Case: PublicFilingPublicView
- Document: RestrictedView
- **Expected:** Access denied (403)

**3. Public Document in Sealed Case**
- User: Public
- Case: SealedCase
- Document: PublicView
- **Expected:** Access denied (case sealed)

**4. Attorney Accessing Their Case**
- User: Attorney (attorney of record)
- Case: SealedCase
- Document: RestrictedView
- **Expected:** Access granted

**5. Attorney Accessing Other Case**
- User: Attorney (NOT attorney of record)
- Case: SealedCase
- Document: RestrictedView
- **Expected:** Access denied (not their case)

**6. Court Staff Accessing Sealed Case**
- User: Court Staff
- Case: SealedCase
- Document: Any
- **Expected:** Access granted (staff can see all)

**7. NoAccess Document**
- User: Public
- Case: PublicFilingPublicView
- Document: NoAccess
- **Expected:** Access denied

**8. Expunged Case**
- User: Any (except possibly court staff)
- Case: ExpungedCase
- Document: Any
- **Expected:** Access denied

---

## Certification Checklist

Before production deployment:

### Security Implementation
- [ ] Case security retrieved from database
- [ ] Document security retrieved from database
- [ ] Most restrictive rule applied
- [ ] User role validated
- [ ] Attorney of record check implemented (for attorneys)
- [ ] Court staff bypass implemented
- [ ] Access denied returns 403 SOAP Fault
- [ ] No documents returned without authorization

### Security Testing
- [ ] Public user can access public documents
- [ ] Public user cannot access restricted documents
- [ ] Public user cannot access sealed cases
- [ ] Attorney can access their cases' restricted docs
- [ ] Attorney cannot access other cases' restricted docs
- [ ] Court staff can access all documents
- [ ] NoAccess documents blocked for public/attorneys
- [ ] Expunged cases blocked appropriately

### Logging and Monitoring
- [ ] All access attempts logged
- [ ] Access denials logged with reason
- [ ] Security level included in logs
- [ ] User role included in logs
- [ ] Audit trail maintained
- [ ] Alerts configured for unusual access patterns

### Error Handling
- [ ] 403 returned for access denied
- [ ] Clear error messages provided
- [ ] No sensitive info exposed in errors
- [ ] SOAP Faults properly formatted

---

## Security Audit Questions

**For CMS Vendors:**

1. Where is case security stored?
2. Where is document security stored?
3. How do you determine attorney of record?
4. Do you cache security decisions? (Should be NO)
5. How do you handle security changes?
6. What happens if security data is missing?
7. Do you log all access denials?
8. How do court staff bypass security?

**For Re:Search Integration:**

1. Does GetDocument check case security?
2. Does GetDocument check document security?
3. Does GetDocument apply most restrictive rule?
4. Does GetDocument validate user authorization?
5. Does GetDocument return proper 403 for denials?
6. Are all access attempts logged?
7. Is there an audit trail?

---

## Next Steps

- **Review request/response format** → Read [Request & Response Specification](./request-response.md)
- **See integration context** → Read [Workflows](./workflows.md)
- **Return to overview** → [GetDocument Overview](./README.md)

---

**Last Updated:** December 2025