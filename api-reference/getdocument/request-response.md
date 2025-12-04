# GetDocument Request & Response Specification

[← Back to GetDocument Overview](./README.md)

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
                  xmlns:doc="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:DocumentQueryMessage-4.0"
                  xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CommonTypes-4.0"
                  xmlns:nc="http://niem.gov/niem/niem-core/2.0"
                  xmlns:j="http://niem.gov/niem/domains/jxdm/4.0">
  <soapenv:Header/>
  <soapenv:Body>
    <doc:DocumentQueryMessage>
      
      <!-- Sender identification -->
      <ecf:SendingMDELocationID>
        <nc:IdentificationID>[sender-url]</nc:IdentificationID>
      </ecf:SendingMDELocationID>
      
      <!-- Profile -->
      <ecf:SendingMDEProfileCode>
        urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:WebServicesMessaging-2.0
      </ecf:SendingMDEProfileCode>
      
      <!-- Query submitter -->
      <ecf:QuerySubmitter>
        <ecf:EntityPerson />
      </ecf:QuerySubmitter>
      
      <!-- Court identification -->
      <j:CaseCourt>
        <nc:OrganizationIdentification>
          <nc:IdentificationID>[court-code]</nc:IdentificationID>
        </nc:OrganizationIdentification>
      </j:CaseCourt>
      
      <!-- Case identification -->
      <nc:CaseTrackingID>[case-id]</nc:CaseTrackingID>
      
      <!-- Document identification (CMSID from GetCase) -->
      <nc:CaseDocketID>[document-cmsid]</nc:CaseDocketID>
      
    </doc:DocumentQueryMessage>
  </soapenv:Body>
</soapenv:Envelope>
```

### Request Fields

| Field | Type | Required? | Description |
|-------|------|-----------|-------------|
| `SendingMDELocationID/IdentificationID` | string | **Required** | Sender URL (typically re:Search) |
| `SendingMDEProfileCode` | string | **Required** | Profile code for web services messaging |
| `j:CaseCourt/OrganizationIdentification/IdentificationID` | string | **Required** | Court code (e.g., "harris:dcccv") |
| `nc:CaseTrackingID` | string | **Required** | Case tracking ID from GetCase |
| `nc:CaseDocketID` | string | **Required** | Document CMSID from GetCase response |

### Critical: Document CMSID Mapping

The `nc:CaseDocketID` in the request **must match** the document CMSID from GetCase:

**In GetCase Response:**
```xml
<DocumentAttachment>
  <tyler:DocumentAttachmentIdentification>
    <nc:IdentificationID>DOC-12345</nc:IdentificationID>
  </tyler:DocumentAttachmentIdentification>
</DocumentAttachment>
```

**In GetDocument Request:**
```xml
<nc:CaseDocketID>DOC-12345</nc:CaseDocketID> ← Must match!
```

**If mismatch:** Document will not be found.

---

## Response Format

### Response Structure (High-Level)

```xml
<soapenv:Envelope>
  <soapenv:Body>
    <DocumentResponseMessage>
      
      <!-- Sender echo -->
      <SendingMDELocationID>
        <IdentificationID>[cms-url]</IdentificationID>
      </SendingMDELocationID>
      
      <!-- Error/success indicator -->
      <Error>
        <ErrorCode>0</ErrorCode>
        <ErrorText>No error</ErrorText>
      </Error>
      
      <!-- Document payload -->
      <Document>
        <DocumentIdentification>
          <IdentificationID>[document-cmsid]</IdentificationID>
          <IdentificationCategoryText>CMSID</IdentificationCategoryText>
        </DocumentIdentification>
        
        <DocumentMetadata>
          <RegisterActionDescriptionText>[filing-code]</RegisterActionDescriptionText>
        </DocumentMetadata>
        
        <DocumentRendition>
          <DocumentDescriptionText>[description]</DocumentDescriptionText>
          <DocumentPostDate>
            <DateTime>[post-date]</DateTime>
          </DocumentPostDate>
          
          <DocumentRenditionMetadata>
            <DocumentAttachment>
              <BinaryBase64Object>[base64-encoded-pdf]</BinaryBase64Object>
              <BinaryDescriptionText>[description]</BinaryDescriptionText>
              <BinaryFormatStandardName>[PDF/TIFF/etc]</BinaryFormatStandardName>
              <BinaryLocationURI>[filename.pdf]</BinaryLocationURI>
              <BinarySizeValue>[bytes]</BinarySizeValue>
              <BinaryCategoryText>[LEAD/ATTACHMENT/EXHIBIT]</BinaryCategoryText>
            </DocumentAttachment>
          </DocumentRenditionMetadata>
        </DocumentRendition>
      </Document>
      
    </DocumentResponseMessage>
  </soapenv:Body>
</soapenv:Envelope>
```

---

## Required Fields

### Error Block (Always Required)

| Field | Type | Description | Values |
|-------|------|-------------|--------|
| `ErrorCode` | integer | Error code | 0 = success, non-zero = error |
| `ErrorText` | string | Human-readable status | "No error" for success |

**Success example:**
```xml
<Error>
  <ErrorCode>0</ErrorCode>
  <ErrorText>No error</ErrorText>
</Error>
```

### Document Identification (Required)

| Field | Type | Description |
|-------|------|-------------|
| `DocumentIdentification/IdentificationID` | string | Document CMSID |
| `DocumentIdentification/IdentificationCategoryText` | string | Should be "CMSID" |

### Document Metadata (Required)

| Field | Type | Description |
|-------|------|-------------|
| `RegisterActionDescriptionText` | string | Filing code from CMS |

### Document Binary (Required)

| Field | Type | Description |
|-------|------|-------------|
| `BinaryBase64Object` | Base64 | **Base64-encoded document content** |
| `BinaryDescriptionText` | string | Document description |
| `BinaryFormatStandardName` | string | File format (PDF, TIFF, JPEG, etc.) |
| `BinaryLocationURI` | string | File name (e.g., "12345.pdf") |
| `BinarySizeValue` | integer | File size in bytes |
| `BinaryCategoryText` | string | LEAD, ATTACHMENT, or EXHIBIT |

### Document Post Date (Required)

| Field | Type | Description |
|-------|------|-------------|
| `DocumentPostDate/DateTime` | ISO 8601 | Date/time document was filed |

---

## Optional Fields

| Field | Purpose |
|-------|---------|
| `DocumentDescriptionText` | Additional description beyond filing code |
| `Hash` | SHA-256 or other integrity hash |
| `PageCount` | Number of pages (if known) |

---

## Error Handling

### Success Response

```xml
<Error>
  <ErrorCode>0</ErrorCode>
  <ErrorText>No error</ErrorText>
</Error>
```

**HTTP 200** with ErrorCode 0 indicates success.

---

### Document Not Found

```xml
<soapenv:Fault>
  <faultcode>soapenv:Server</faultcode>
  <faultstring>Document not found</faultstring>
  <detail>
    <ErrorDetail>
      <ErrorCode>404</ErrorCode>
      <ErrorMessage>Document DOC-12345 does not exist in case CV-2024-00123</ErrorMessage>
      <CaseDocketID>DOC-12345</CaseDocketID>
    </ErrorDetail>
  </detail>
</soapenv:Fault>
```

**When to return:**
- Document ID doesn't exist in CMS
- Document file missing from storage
- CMSID from request doesn't match any document

---

### Access Denied (Security)

```xml
<soapenv:Fault>
  <faultcode>soapenv:Client</faultcode>
  <faultstring>Access denied</faultstring>
  <detail>
    <ErrorDetail>
      <ErrorCode>403</ErrorCode>
      <ErrorMessage>User does not have permission to access sealed document</ErrorMessage>
      <CaseDocketID>DOC-12345</CaseDocketID>
      <SecurityLevel>Sealed</SecurityLevel>
    </ErrorDetail>
  </detail>
</soapenv:Fault>
```

**When to return:**
- Case is sealed and user is not authorized
- Document is restricted and user is not attorney of record
- Document has NoAccess security setting

---

### File Read Error

```xml
<soapenv:Fault>
  <faultcode>soapenv:Server</faultcode>
  <faultstring>Unable to retrieve document</faultstring>
  <detail>
    <ErrorDetail>
      <ErrorCode>500</ErrorCode>
      <ErrorMessage>Document file exists in database but cannot be read from storage</ErrorMessage>
      <CaseDocketID>DOC-12345</CaseDocketID>
    </ErrorDetail>
  </detail>
</soapenv:Fault>
```

**When to return:**
- File path in database but file doesn't exist
- File permissions issue
- Storage system unavailable
- Corrupted file

---

### HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | Success | Document returned successfully |
| 400 | Bad Request | Invalid request format or missing required fields |
| 401 | Unauthorized | Certificate authentication failed |
| 403 | Forbidden | User not authorized to access document |
| 404 | Not Found | Document doesn't exist |
| 500 | Internal Server Error | CMS error retrieving document |
| 503 | Service Unavailable | Document storage temporarily unavailable |
| 504 | Gateway Timeout | Request exceeded timeout (> 5s) |

---

## Complete Examples

### Example 1: Simple PDF Request

**Scenario:** User clicks to view a PDF document

**Request:**
[View: examples/getdocument-request-clean.xml](./examples/getdocument-request-clean.xml)

```xml
<DocumentQueryMessage
    xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:DocumentQueryMessage-4.0"
    xmlns:j="http://niem.gov/niem/domains/jxdm/4.0"
    xmlns:nc="http://niem.gov/niem/niem-core/2.0"
    xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CommonTypes-4.0">
  <ecf:SendingMDELocationID>
    <nc:IdentificationID>https://courtrecordmde.com</nc:IdentificationID>
  </ecf:SendingMDELocationID>
  <ecf:SendingMDEProfileCode>
    urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:WebServicesMessaging-2.0
  </ecf:SendingMDEProfileCode>
  <ecf:QuerySubmitter>
    <ecf:EntityPerson />
  </ecf:QuerySubmitter>
  <j:CaseCourt>
    <nc:OrganizationIdentification>
      <nc:IdentificationID>harris:dcccv</nc:IdentificationID>
    </nc:OrganizationIdentification>
  </j:CaseCourt>
  <nc:CaseTrackingID>054Rti3-3252-4510-acdc-1205ca5aa5a2</nc:CaseTrackingID>
  <nc:CaseDocketID>9Ghif57gh-1617-4bc7-83e0-6971c8cb2712</nc:CaseDocketID>
</DocumentQueryMessage>
```

**Response:**
[View: examples/getdocument-response-annotated.xml](./examples/getdocument-response-annotated.xml)

**For annotated examples with field explanations, see:**
- [getdocument-request-annotated.xml](./examples/getdocument-request-annotated.xml) - Request with detailed comments
- [getdocument-response-annotated.xml](./examples/getdocument-response-annotated.xml) - Response with field explanations

---

### Example 2: Response with Full Metadata

```xml
<DocumentResponseMessage
    xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:DocumentResponseMessage-4.0"
    xmlns:nc="http://niem.gov/niem/niem-core/2.0"
    xmlns:j="http://niem.gov/niem/domains/jxdm/4.0"
    xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CommonTypes-4.0">
  
  <ecf:SendingMDELocationID>
    <nc:IdentificationID>https://www.idocket.com/</nc:IdentificationID>
  </ecf:SendingMDELocationID>
  
  <ecf:Error>
    <ecf:ErrorCode>0</ecf:ErrorCode>
    <ecf:ErrorText>No error</ecf:ErrorText>
  </ecf:Error>
  
  <ecf:Document>
    <nc:DocumentIdentification>
      <nc:IdentificationID>13763456</nc:IdentificationID>
      <nc:IdentificationCategoryText>CMSID</nc:IdentificationCategoryText>
    </nc:DocumentIdentification>
    
    <ecf:DocumentMetadata>
      <j:RegisterActionDescriptionText>evtapp</j:RegisterActionDescriptionText>
    </ecf:DocumentMetadata>
    
    <ecf:DocumentRendition>
      <nc:DocumentDescriptionText>APPLICATION</nc:DocumentDescriptionText>
      <nc:DocumentPostDate>
        <nc:DateTime>2025-01-29T00:00:00</nc:DateTime>
      </nc:DocumentPostDate>
      
      <ecf:DocumentRenditionMetadata>
        <ecf:DocumentAttachment>
          <nc:BinaryBase64Object>
            JVBERi0xLjQKJeLjz9MKMyAwIG9iago8PC9GaWx0ZXIvRmxhdGVEZWNv...
            [truncated for brevity - full base64 PDF content here]
          </nc:BinaryBase64Object>
          <nc:BinaryDescriptionText>APPLICATION</nc:BinaryDescriptionText>
          <nc:BinaryFormatStandardName>PDF</nc:BinaryFormatStandardName>
          <nc:BinaryLocationURI>1376273.pdf</nc:BinaryLocationURI>
          <nc:BinarySizeValue>87217</nc:BinarySizeValue>
          <nc:BinaryCategoryText>LEAD</nc:BinaryCategoryText>
        </ecf:DocumentAttachment>
      </ecf:DocumentRenditionMetadata>
    </ecf:DocumentRendition>
  </ecf:Document>
</DocumentResponseMessage>
```

---

### Example 3: TIFF Document

```xml
<DocumentAttachment>
  <nc:BinaryBase64Object>
    SUkqAAgAAAAPAP4ABAABAAAAAAAAEgEDAAEAAA...
    [base64-encoded TIFF content]
  </nc:BinaryBase64Object>
  <nc:BinaryDescriptionText>Scanned Exhibit A</nc:BinaryDescriptionText>
  <nc:BinaryFormatStandardName>TIFF</nc:BinaryFormatStandardName>
  <nc:BinaryLocationURI>exhibit-a-scan.tiff</nc:BinaryLocationURI>
  <nc:BinarySizeValue>245678</nc:BinarySizeValue>
  <nc:BinaryCategoryText>EXHIBIT</nc:BinaryCategoryText>
</DocumentAttachment>
```

---

### Example 4: Document Not Found Error

```xml
<soapenv:Fault>
  <faultcode>soapenv:Server</faultcode>
  <faultstring>Document not found</faultstring>
  <detail>
    <ErrorDetail>
      <ErrorCode>404</ErrorCode>
      <ErrorMessage>
        Document with CMSID 'DOC-99999' does not exist in case 'CV-2024-00123'
      </ErrorMessage>
      <CaseTrackingID>CV-2024-00123</CaseTrackingID>
      <CaseDocketID>DOC-99999</CaseDocketID>
    </ErrorDetail>
  </detail>
</soapenv:Fault>
```

---

### Example 5: Access Denied (Sealed Case)

```xml
<soapenv:Fault>
  <faultcode>soapenv:Client</faultcode>
  <faultstring>Access denied</faultstring>
  <detail>
    <ErrorDetail>
      <ErrorCode>403</ErrorCode>
      <ErrorMessage>
        Case is sealed. User 'public' does not have permission to access documents.
      </ErrorMessage>
      <CaseTrackingID>CR-2024-00456</CaseTrackingID>
      <CaseDocketID>DOC-12345</CaseDocketID>
      <SecurityLevel>SealedCase</SecurityLevel>
      <UserRole>Public</UserRole>
    </ErrorDetail>
  </detail>
</soapenv:Fault>
```

---

## Implementation Examples

### C# Example

```csharp
public async Task<DocumentResponse> GetDocumentAsync(
    string courtCode,
    string caseTrackingId,
    string documentCmsId)
{
    // 1. Retrieve document metadata
    var document = await _repository.GetDocumentAsync(documentCmsId);
    if (document == null)
    {
        throw new DocumentNotFoundException(
            $"Document {documentCmsId} not found");
    }
    
    // 2. Get case for security context
    var caseData = await _repository.GetCaseAsync(document.CaseId);
    
    // 3. Check security (most restrictive wins)
    var effectiveSecurity = DetermineEffectiveSecurity(
        caseData.Security, document.Security);
    
    // 4. Validate user access
    if (!UserCanAccess(_currentUser, effectiveSecurity, document))
    {
        throw new AccessDeniedException(
            $"User {_currentUser.Role} cannot access " +
            $"{effectiveSecurity} document");
    }
    
    // 5. Read file from storage
    byte[] fileBytes;
    try
    {
        fileBytes = await _storage.ReadFileAsync(document.FilePath);
    }
    catch (FileNotFoundException)
    {
        throw new DocumentStorageException(
            $"Document file not found at {document.FilePath}");
    }
    
    // 6. Base64 encode
    string base64Content = Convert.ToBase64String(fileBytes);
    
    // 7. Build response
    return new DocumentResponse
    {
        DocumentId = document.CmsId,
        FilingCode = document.FilingCode,
        Description = document.Description,
        BinaryContent = base64Content,
        FileName = document.FileName,
        FileSize = fileBytes.Length,
        FileFormat = document.Format,
        PostDate = document.PostDate
    };
}
```

### Java Example

```java
public DocumentResponse getDocument(
    String courtCode,
    String caseTrackingId,
    String documentCmsId) throws DocumentException {
    
    // 1. Get document metadata
    Document doc = documentRepository.findByCmsId(documentCmsId);
    if (doc == null) {
        throw new DocumentNotFoundException(
            "Document " + documentCmsId + " not found");
    }
    
    // 2. Get case for security
    Case caseData = caseRepository.findById(doc.getCaseId());
    
    // 3. Determine effective security
    SecurityLevel effectiveSecurity = SecurityHelper.getMostRestrictive(
        caseData.getSecurity(),
        doc.getSecurity()
    );
    
    // 4. Check user authorization
    if (!securityService.canAccess(currentUser, effectiveSecurity, doc)) {
        throw new AccessDeniedException(
            "User " + currentUser.getRole() + 
            " cannot access " + effectiveSecurity + " document");
    }
    
    // 5. Read file
    byte[] fileBytes;
    try {
        fileBytes = Files.readAllBytes(Paths.get(doc.getFilePath()));
    } catch (IOException e) {
        throw new DocumentStorageException(
            "Cannot read file: " + doc.getFilePath(), e);
    }
    
    // 6. Base64 encode
    String base64Content = Base64.getEncoder().encodeToString(fileBytes);
    
    // 7. Build response
    DocumentResponse response = new DocumentResponse();
    response.setDocumentId(doc.getCmsId());
    response.setFilingCode(doc.getFilingCode());
    response.setDescription(doc.getDescription());
    response.setBinaryContent(base64Content);
    response.setFileName(doc.getFileName());
    response.setFileSize(fileBytes.length);
    response.setFileFormat(doc.getFormat());
    response.setPostDate(doc.getPostDate());
    
    return response;
}
```

---

## Best Practices

### File Retrieval

**DO:**
- Check file exists before attempting to read
- Handle missing files gracefully with 404
- Log all file access attempts
- Validate file path before reading
- Use streaming for large files (> 10 MB)

**DON'T:**
- Return empty BinaryBase64Object (return error instead)
- Cache document content (re:Search handles caching)
- Skip security checks
- Read entire file into memory for huge files

### Base64 Encoding

**DO:**
- Use standard Base64 encoding (RFC 4648)
- Remove line breaks from encoded string
- Validate encoding succeeded before returning

**DON'T:**
- Use URL-safe Base64 encoding
- Include line breaks or whitespace
- Assume encoding will always succeed

### Security Enforcement

**DO:**
- Always check case security first
- Apply most restrictive security rule
- Log all access denials
- Return clear error messages

**DON'T:**
- Skip security checks for "public" documents
- Assume GetCase security is always correct
- Return documents without authorization

### Error Handling

**DO:**
- Return appropriate HTTP status codes
- Include helpful error messages
- Log errors for troubleshooting
- Distinguish between not found and access denied

**DON'T:**
- Return empty responses instead of errors
- Expose internal file paths in errors
- Return generic "error" messages

---

## Validation Checklist

Before certification:

### Request Handling
- [ ] Validates all required fields present
- [ ] Handles missing CaseDocketID gracefully
- [ ] Validates court code format
- [ ] Logs all incoming requests

### Document Retrieval
- [ ] Queries database for document metadata
- [ ] Verifies document file exists in storage
- [ ] Reads file from storage correctly
- [ ] Handles missing files with 404 error

### Security Enforcement
- [ ] Retrieves case security settings
- [ ] Retrieves document security settings
- [ ] Applies most restrictive security rule
- [ ] Validates user authorization
- [ ] Returns 403 for access denied

### Response Building
- [ ] Base64 encodes file content correctly
- [ ] Populates all required fields
- [ ] Sets correct error code (0 for success)
- [ ] Includes file size in bytes
- [ ] Sets correct file format name

### Performance
- [ ] Response time < 2 seconds for typical files
- [ ] Response time < 5 seconds for large files
- [ ] Handles concurrent requests
- [ ] No memory leaks with large files

### Error Cases
- [ ] Returns SOAP Fault for missing documents
- [ ] Returns SOAP Fault for access denied
- [ ] Returns SOAP Fault for storage errors
- [ ] Includes helpful error messages
- [ ] Logs all errors for troubleshooting

---

## Additional XML Examples

The [examples directory](./examples/) contains complete GetDocument XML files:

| File | Description |
|------|-------------|
| [getdocument-request-annotated.xml](./examples/getdocument-request-annotated.xml) | Annotated request with field explanations |
| [getdocument-request-clean.xml](./examples/getdocument-request-clean.xml) | Clean request template |
| [getdocument-response-annotated.xml](./examples/getdocument-response-annotated.xml) | Annotated response with full metadata |

---

## Next Steps

- **Understand security enforcement** → Read [Security Guide](./security-guide.md)
- **See integration context** → Read [Workflows](./workflows.md)
- **Review examples** → Browse [examples directory](./examples/)
- **Return to overview** → [GetDocument Overview](./README.md)

---

**Last Updated:** December 2025