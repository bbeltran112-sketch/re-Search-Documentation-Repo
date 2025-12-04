# GetDocument API

**Navigation:** [Home](../../../README.md) › [Technical Documentation](../../README.md) › [API Reference](../README.md) › GetDocument

## Quick Navigation
- [Request & Response Specification](./request-response.md) - Technical details and examples
- [Security Guide](./security-guide.md) - Document security enforcement rules
- [Workflows](./workflows.md) - When and how documents are retrieved
- [XML Examples](./examples/) - Sample request/response files

---

## What is GetDocument?

GetDocument is the **document retrieval API** that returns binary document content (PDFs, TIFFs, etc.) when users view or download documents in re:Search. It's called on-demand and must enforce document-level security.

**Think of it as:** re:Search asking "Give me the PDF for document ID 12345"

---

## At a Glance

| Property | Value |
|---------|--------|
| **Direction** | re:Search → CMS |
| **Protocol** | SOAP 1.2 over HTTPS |
| **Security** | Mutual TLS (mTLS) |
| **Standard** | OASIS ECF 4.x |
| **Message Type** | DocumentQueryMessage / DocumentResponseMessage |
| **Required?** | Yes (ECF/CIP modes) - Optional if using Batch Mode |
| **Returns** | Base64-encoded document binary + metadata |

---

## When GetDocument is Called

GetDocument is triggered when:

1. **User clicks document in UI** - User wants to view or download
2. **Document preview needed** - Thumbnail or preview generation
3. **NotifyCaseEvent received** - EventTypes: CaseFiling, DocumentSecurity, DocumentFiling
4. **Background validation** - re:Search verifies document availability

**Frequency:** On-demand per user action (not bulk retrieval)

---

## What GetDocument Returns

**Document binary and metadata:**
- Base64-encoded PDF/TIFF/image content
- Document description and filing code
- File name and size
- Document type and category (LEAD/ATTACHMENT/EXHIBIT)
- Post date

**Critical:** Returns binary ONLY - no case metadata, parties, or filings (use GetCase for that)

---

## Quick Example

### Request (What re:Search Sends)

```xml
<DocumentQueryMessage>
  <!-- Which court? -->
  <j:CaseCourt>
    <nc:OrganizationIdentification>
      <nc:IdentificationID>harris:dcccv</nc:IdentificationID>
    </nc:OrganizationIdentification>
  </j:CaseCourt>
  
  <!-- Which case? -->
  <nc:CaseTrackingID>054Rti3-3252-4510-acdc-1205ca5aa5a2</nc:CaseTrackingID>
  
  <!-- Which document? -->
  <nc:CaseDocketID>9Ghif57gh-1617-4bc7-83e0-6971c8cb2712</nc:CaseDocketID>
</DocumentQueryMessage>
```

### Response (What CMS Returns)

```xml
<DocumentResponseMessage>
  <Error>
    <ErrorCode>0</ErrorCode>
    <ErrorText>No error</ErrorText>
  </Error>
  <Document>
    <DocumentIdentification>
      <IdentificationID>13763456</IdentificationID>
      <IdentificationCategoryText>CMSID</IdentificationCategoryText>
    </DocumentIdentification>
    <DocumentRendition>
      <DocumentDescriptionText>APPLICATION</DocumentDescriptionText>
      <DocumentRenditionMetadata>
        <DocumentAttachment>
          <!-- Base64-encoded PDF content -->
          <BinaryBase64Object>JVBERi0xLjQKJw...</BinaryBase64Object>
          <BinaryLocationURI>1376273.pdf</BinaryLocationURI>
          <BinarySizeValue>87217</BinarySizeValue>
          <BinaryFormatStandardName>PDF</BinaryFormatStandardName>
        </DocumentAttachment>
      </DocumentRenditionMetadata>
    </DocumentRendition>
  </Document>
</DocumentResponseMessage>
```

[See complete examples →](./request-response.md#complete-examples)

---

## Key Requirements

### Must Include (Required)
- ✅ Base64-encoded binary content (`BinaryBase64Object`)
- ✅ Document CMSID (`DocumentIdentification/IdentificationID`)
- ✅ File name (`BinaryLocationURI`)
- ✅ File size in bytes (`BinarySizeValue`)
- ✅ Document format (`BinaryFormatStandardName` - PDF, TIFF, etc.)
- ✅ Filing code (`RegisterActionDescriptionText`)
- ✅ Error code and text (0 = success)

### Must Enforce (Security)
- ✅ Case-level security (most restrictive wins)
- ✅ Document-level security settings
- ✅ User authorization checks
- ✅ Return SOAP Fault if access denied

### Performance Expectations
- **Target:** < 2 seconds response time
- **Maximum:** < 5 seconds
- **File size:** Up to 50 MB supported

---

## Common Issues

| Issue | Cause | Impact |
|-------|-------|--------|
| **Empty BinaryBase64Object** | Document file not found or not encoded | User gets 0 KB file or error |
| **Missing CMSID** | Document ID from GetCase doesn't match | Document fails to load |
| **Wrong security settings** | Security not enforced | Sealed docs visible to public |
| **Timeout** | Large file or slow database | User sees "Document unavailable" |
| **Invalid Base64** | Encoding error | Document won't open |

[Complete troubleshooting →](./workflows.md#troubleshooting)

---

## Document Security Enforcement

**Critical:** GetDocument MUST enforce security - never return documents the user shouldn't access.

### Security Hierarchy (Most Restrictive Wins)

```
Case Security + Document Security = Effective Security
```

| Case Security | Document Security | Can User Access? |
|---------------|-------------------|------------------|
| Sealed | Any | **NO** - Entire case sealed |
| Public | Public | **YES** - Public can view |
| Public | Restricted | **LIMITED** - Attorneys of record only |
| Public | NoAccess | **NO** - Document completely hidden |

**Example:**
- Case = Public
- Document = Restricted
- User = Public
- **Result:** Access denied

[Complete security rules →](./security-guide.md)

---

## Document CMSID Mapping

**Critical:** The document ID in GetDocument request comes from GetCase response.

**GetCase provides:**
```xml
<DocumentAttachment>
  <tyler:DocumentAttachmentIdentification>
    <nc:IdentificationID>DOC-12345</nc:IdentificationID> ← This CMSID
  </tyler:DocumentAttachmentIdentification>
</DocumentAttachment>
```

**GetDocument receives:**
```xml
<nc:CaseDocketID>DOC-12345</nc:CaseDocketID> ← Must match!
```

**If CMSID changes or is incorrect:**
- Document fails to load
- "Document not found" errors
- Broken links in UI

---

## Supported File Types

| File Type | Supported | Common Use | Notes |
|-----------|-----------|------------|-------|
| **PDF** | ✅ | Most documents | Preferred format |
| **TIFF** | ✅ | Scanned documents | Multi-page supported |
| **JPEG/PNG** | ✅ | Exhibits, photos | Single images |
| **Word/Excel** | ✅ | Raw office docs | Must Base64-encode |
| **Audio/Video** | ⚠️ | Rare | Allowed but not recommended |

**CMS must:**
- Read file from storage
- Base64 encode the binary content
- Set correct `BinaryFormatStandardName`

---

## Caching Behavior

**re:Search caches documents** to reduce calls to CMS:

- **Cache duration:** Configurable (typically 30-90 days)
- **When cache expires:** GetDocument called again
- **When cache bypassed:** DocumentSecurity events trigger refresh

**CMS should always return current document** - re:Search handles caching.

---

## Quick Start Checklist

### For Developers Implementing GetDocument

- [ ] Review [Request & Response Specification](./request-response.md)
- [ ] Understand [security enforcement rules](./security-guide.md)
- [ ] Review [workflows](./workflows.md) to see when GetDocument is called
- [ ] Implement document retrieval from storage
- [ ] Add Base64 encoding
- [ ] Implement security checks (case + document level)
- [ ] Add error handling (document not found, access denied)
- [ ] Test with all file types (PDF, TIFF, images)
- [ ] Verify CMSIDs match GetCase response
- [ ] Test response times (< 2 seconds target)
- [ ] Test with sealed/restricted documents
- [ ] Validate with [certification checklist](./security-guide.md#certification-checklist)

### For Operations Teams

- [ ] Monitor GetDocument response times
- [ ] Alert on timeouts (> 5 seconds)
- [ ] Monitor error rates
- [ ] Verify document storage accessible
- [ ] Ensure mTLS certificates valid
- [ ] Review security denial logs

---

## Related APIs

GetDocument works in coordination with:

**Provides Document IDs:**
- [GetCase API](../getcase/) - Returns document CMSIDs in response
- [NotifyDocketingComplete](../notifydocketingcomplete/) - Provides document IDs after docketing

**Triggers GetDocument:**
- [NotifyCaseEvent](../notifycaseevent/) - CaseFiling, DocumentSecurity, DocumentFiling events

**Workflow:**
```
GetCase returns CMSID → User clicks document → GetDocument retrieves binary
```

---

## Authentication

GetDocument requires mutual TLS (mTLS) authentication.

**Certificate requirements:**
- Valid mTLS certificate issued by Tyler
- Certificate must not be expired
- Proper certificate chain validation
- Secure certificate storage

[Authentication details →](../common-headers-and-auth.md)

---

## Next Steps

### New to GetDocument?
1. Read [Request & Response Specification](./request-response.md) to understand the API structure
2. Review [Security Guide](./security-guide.md) to understand enforcement rules
3. Study [Workflows](./workflows.md) to see GetDocument in context

### Implementing GetDocument?
1. Start with [Request & Response Specification](./request-response.md)
2. Download [example XML files](./examples/)
3. Implement document retrieval logic
4. Add Base64 encoding
5. Implement security enforcement using [Security Guide](./security-guide.md)
6. Test with different file types
7. Complete [certification checklist](./security-guide.md#certification-checklist)

### Troubleshooting Issues?
1. Check [common issues](#common-issues) above
2. Review [Workflows troubleshooting](./workflows.md#troubleshooting)
3. Verify CMSIDs match GetCase response
4. Check security settings
5. Contact your Tyler Technical Project Manager

---

## Additional Resources

- [GetCase API](../getcase/) - Provides document CMSIDs
- [NotifyCaseEvent API](../notifycaseevent/) - Events that trigger GetDocument
- [Security Logic](../../support-playbook/security-logic.md) - Overall security rules
- [Troubleshooting Guide](../../support-playbook/troubleshooting.md) - Common problems

---

**Last Updated:** December 2025