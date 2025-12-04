# GetCase Request & Response Specification

[← Back to GetCase Overview](./README.md)

## Quick Navigation
- [Request Format](#request-format)
- [Response Format](#response-format)
- [Required Fields](#required-fields)
- [Optional Fields](#optional-fields)
- [Error Handling](#error-handling)
- [Complete Examples](#complete-examples)

---

## Request Format

### Request Structure

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:wsdl:WebServiceMessagingProfile-Definitions-4.0"
                  xmlns:tyler="urn:tyler:ecf:extensions:GetCaseMessage"
                  xmlns:nc="http://niem.gov/niem/niem-core/2.0">
  <soapenv:Header/>
  <soapenv:Body>
    <ecf:GetCaseRequest>
      <tyler:GetCaseRequestMessage>
        
        <!-- Request identification -->
        <tyler:MessageID>[unique-request-id]</tyler:MessageID>
        <tyler:RequestDateTime>[ISO8601-timestamp]</tyler:RequestDateTime>
        
        <!-- Court identification -->
        <tyler:CourtIdentification>
          <nc:IdentificationID>[court-code]</nc:IdentificationID>
        </tyler:CourtIdentification>
        
        <!-- Case identification -->
        <tyler:CaseTrackingID>[case-id]</tyler:CaseTrackingID>
        
        <!-- Optional: Control what's included -->
        <tyler:IncludeDocumentMetadata>true|false</tyler:IncludeDocumentMetadata>
        <tyler:IncludePartyInformation>true|false</tyler:IncludePartyInformation>
        
        <!-- Optional: Delta updates (if supported) -->
        <tyler:SinceDateTime>[ISO8601-timestamp]</tyler:SinceDateTime>
        
      </tyler:GetCaseRequestMessage>
    </ecf:GetCaseRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

### Request Fields

| Field | Type | Required? | Description |
|-------|------|-----------|-------------|
| `MessageID` | string | **Required** | Unique identifier for this request (UUID recommended) |
| `RequestDateTime` | ISO 8601 | **Required** | UTC timestamp of request |
| `CourtIdentification/IdentificationID` | string | **Required** | Court code (e.g., "Hopkins:cc") |
| `CaseTrackingID` | string | **Required** | Authoritative CMS case identifier |
| `IncludeDocumentMetadata` | boolean | Optional | Include document details (default: true, **recommended**) |
| `IncludePartyInformation` | boolean | Optional | Include party/attorney data (default: true, **recommended**) |
| `SinceDateTime` | ISO 8601 | Optional | Return only changes since timestamp (if CMS supports delta) |

### Request Example (Annotated)

```xml
<ecf:GetCaseRequest>
  <tyler:GetCaseRequestMessage>
  
    <!-- [REQUIRED] Unique per request - use UUID or GUID -->
    <tyler:MessageID>0f1c68b3-8a55-4f7c-a41a-123456789abc</tyler:MessageID>
    
    <!-- [REQUIRED] Must be UTC, ISO 8601 format -->
    <tyler:RequestDateTime>2025-12-04T10:30:00Z</tyler:RequestDateTime>
    
    <!-- [REQUIRED] Mapped court/jurisdiction code -->
    <tyler:CourtIdentification>
      <nc:IdentificationID>Hopkins:cc</nc:IdentificationID>
    </tyler:CourtIdentification>
    
    <!-- [REQUIRED] Authoritative CMS case ID -->
    <tyler:CaseTrackingID>CASE-12345678</tyler:CaseTrackingID>
    
    <!-- [OPTIONAL][RECOMMENDED] Include document metadata for correlation -->
    <tyler:IncludeDocumentMetadata>true</tyler:IncludeDocumentMetadata>
    
    <!-- [OPTIONAL][RECOMMENDED] Include parties and attorneys -->
    <tyler:IncludePartyInformation>true</tyler:IncludePartyInformation>
    
    <!-- [OPTIONAL][NEW] Delta updates - return only changes since date -->
    <!-- Only use if your CMS supports incremental updates -->
    <!-- <tyler:SinceDateTime>2025-10-01T00:00:00Z</tyler:SinceDateTime> -->
    
  </tyler:GetCaseRequestMessage>
</ecf:GetCaseRequest>
```

---

## Response Format

### Response Structure (High-Level)

```xml
<soapenv:Envelope>
  <soapenv:Body>
    <ecf:GetCaseResponse>
      <tyler:GetCaseResponseMessage>
        <ecf:Case>
        
          <!-- Case header -->
          <ecf:CaseTrackingID/>
          <ecf:CaseTitleText/>
          <ecf:CaseCategoryText/>
          <tyler:CaseSecurity/>
          
          <!-- Filing events -->
          <tyler:FilingEvent>
            <ecf:ActivityDate/>
            <j:RegisterActionDescriptionText/>
            <ecf:DocumentMetadata/>
            <ecf:DocumentRendition/>
          </tyler:FilingEvent>
          
          <!-- Parties -->
          <ecf:CaseParticipant>
            <ecf:EntityPerson/>
            <ecf:CasePartyRole/>
          </ecf:CaseParticipant>
          
          <!-- Attorneys -->
          <ecf:CaseAttorney>
            <ecf:AttorneyID/>
            <ecf:PersonName/>
            <ecf:BarNumber/>
            <ecf:RepresentsParty/>
          </ecf:CaseAttorney>
          
        </ecf:Case>
      </tyler:GetCaseResponseMessage>
    </ecf:GetCaseResponse>
  </soapenv:Body>
</soapenv:Envelope>
```

---

## Required Fields

### Case Header (Always Required)

| Field | Description | Example |
|-------|-------------|---------|
| `ecf:CaseTrackingID` | Authoritative CMS case ID | `"CASE-12345678"` |
| `ecf:CaseTitleText` | Case title/caption | `"Smith v. Jones"` |
| `tyler:CaseSecurity` | Case-level security | `"PublicFilingPublicView"` |
| `ecf:CaseCategoryText` | Case category | `"Civil"` |
| `ecf:CaseTypeText` | Case type | `"Contract Dispute"` |

### Filing Events (Required for CaseFiling EventType)

| Field | Description | Notes |
|-------|-------------|-------|
| `ecf:ActivityDate/nc:DateTime` | **Required** filing timestamp | ISO 8601 UTC - **must be present** |
| `j:RegisterActionDescriptionText` | Filing/docket description | Human-readable action |
| `ecf:DocumentMetadata` | Filing-level metadata | Links to documents |
| `tyler:DocumentAttachmentIdentification` | Document CMSID mapping | **Required** for document retrieval |

### Documents (Required When Included)

| Field | Description | Notes |
|-------|-------------|-------|
| `nc:IdentificationID` (CMSID) | Document identifier | **Required** - used in GetDocument |
| `tyler:PageCount` | Number of pages | **Required** for indexing and UI |
| `tyler:DocumentSecurity` | Document-level security | Controls individual document access |
| `nc:BinaryFormatStandardName` | Document format | Usually "PDF" |
| `structures:id` | Cross-reference ID | Links metadata to attachments |

### Parties (Required for CaseParty EventType)

| Field | Description |
|-------|-------------|
| `ecf:EntityPerson/PersonName` | Party name |
| `ecf:CasePartyRole` | Role (Plaintiff, Defendant, etc.) |
| `nc:ContactInformation` | Contact details (optional but recommended) |

---

## Optional Fields

### Nice to Have (Improves Quality)

| Field | Purpose |
|-------|---------|
| `ecf:CaseSubTypeText` | Deeper case classification |
| `ecf:ActivityStatus` | Filing status details |
| `nc:BinaryLocationURI` | Direct document link |
| `nc:BinarySizeValue` | Document file size |
| `nc:DocumentDescriptionText` | Document description |
| `j:CaseCourtEvent` | Hearing/event information |
| `ecf:CaseDisposition` | Disposition details |

### Future/Enhancement Fields

| Field | Purpose | Status |
|-------|---------|--------|
| `tyler:SinceDateTime` (request) | Delta updates | Optional - if CMS supports |
| `ecf:CaseAugmentation` | Extended metadata | Future enhancement |

---

## Error Handling

### Case Not Found

**Return a SOAP Fault - DO NOT return empty `<Case>` block**

```xml
<soapenv:Envelope>
  <soapenv:Body>
    <soapenv:Fault>
      <faultcode>soapenv:Server</faultcode>
      <faultstring>Case not found</faultstring>
      <detail>
        <tyler:ErrorDetail>
          <tyler:ErrorCode>CASE_NOT_FOUND</tyler:ErrorCode>
          <tyler:ErrorMessage>Case CASE-12345678 does not exist in court Hopkins:cc</tyler:ErrorMessage>
          <tyler:CaseTrackingID>CASE-12345678</tyler:CaseTrackingID>
        </tyler:ErrorDetail>
      </detail>
    </soapenv:Fault>
  </soapenv:Body>
</soapenv:Envelope>
```

### Invalid Request

```xml
<soapenv:Fault>
  <faultcode>soapenv:Client</faultcode>
  <faultstring>Invalid request</faultstring>
  <detail>
    <tyler:ErrorDetail>
      <tyler:ErrorCode>INVALID_REQUEST</tyler:ErrorCode>
      <tyler:ErrorMessage>CaseTrackingID is required</tyler:ErrorMessage>
    </tyler:ErrorDetail>
  </detail>
</soapenv:Fault>
```

### System Error

```xml
<soapenv:Fault>
  <faultcode>soapenv:Server</faultcode>
  <faultstring>Internal server error</faultstring>
  <detail>
    <tyler:ErrorDetail>
      <tyler:ErrorCode>INTERNAL_ERROR</tyler:ErrorCode>
      <tyler:ErrorMessage>Database connection failed</tyler:ErrorMessage>
    </tyler:ErrorDetail>
  </detail>
</soapenv:Fault>
```

### HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | Success | Valid response returned |
| 400 | Bad Request | Invalid request format |
| 401 | Unauthorized | Certificate authentication failed |
| 404 | Not Found | Case does not exist |
| 500 | Internal Server Error | CMS error processing request |
| 503 | Service Unavailable | CMS temporarily unavailable |
| 504 | Gateway Timeout | Request exceeded timeout (> 5s) |

---

## Complete Examples

### Example 1: Simple Case Request

**Request:**
[View: getcase-request-annotated.xml](./examples/getcase-request-annotated.xml)

Basic request with minimal options - suitable for simple case lookups.

**Response:**
[View: getcase-response-annotated.xml](./examples/getcase-response-annotated.xml)

Basic response structure showing core case elements.

---

### Example 2: Request with Parties and Documents

**Request:**
[View: getcase-request-withpartiesanddocuments.xml](./examples/getcase-request-withpartiesanddocuments.xml)

Full request including both party information and document metadata flags set to `true`.

Shows how to request complete case data with all related entities.

---

### Example 3: Request for Party Information Only

**Request:**
[View: getcase-request-partys.xml](./examples/getcase-request-partys.xml)

Request focused on retrieving party and attorney information.

Useful when responding to CaseParty EventType.

---

### Additional Examples

The [examples directory](./examples/) contains additional XML files showing:
- Complete request/response pairs
- Various EventType scenarios
- Security variations
- Complex case structures

Browse the examples folder for more reference implementations.

---

## Field Reference Tables

### Case Security Values

| Value | Meaning | Public Visibility |
|-------|---------|-------------------|
| `PublicFilingPublicView` | Public case, public filings | Yes - all content visible |
| `PublicFilingRestrictedView` | Public case, restricted filings | Case visible, some filings hidden |
| `SealedCase` | Entire case sealed | No - case completely hidden |
| `ExpungedCase` | Case expunged | No - case removed from public view |

### Document Security Values

| Value | Meaning | Visibility |
|-------|---------|------------|
| `PublicView` | Public document | Yes - anyone can view |
| `RestrictedView` | Restricted document | Limited metadata shown, content hidden |
| `NoAccess` | No public access | Document completely hidden |

### Binary Category Values

| Value | Meaning | Display Order |
|-------|---------|---------------|
| `LEAD` | Lead/primary document | First in filing |
| `ATTACHMENT` | Supporting attachment | After lead document |
| `EXHIBIT` | Exhibit | After attachments |

---

## Performance Guidelines

### Response Time Targets

| Case Complexity | Target | Maximum |
|----------------|--------|---------|
| **Simple** (1-5 parties, 1-10 docs) | < 1 second | 2 seconds |
| **Average** (5-15 parties, 10-50 docs) | < 2 seconds | 3 seconds |
| **Complex** (15+ parties, 50+ docs) | < 3 seconds | 5 seconds |

### Optimization Tips

**Database Queries:**
- Use single query with joins (avoid N+1 problem)
- Add indexes on case_tracking_id
- Eager load related data (parties, documents)

**Response Generation:**
- Don't cache GetCase responses (must be current)
- Stream large responses
- Compress if possible

**Testing:**
- Test with production-sized cases
- Profile slow queries
- Monitor response times

---

## Validation Checklist

Before submitting for certification:

### Request Validation
- [ ] MessageID is unique per request
- [ ] RequestDateTime is valid ISO 8601 UTC
- [ ] CourtIdentification matches expected format
- [ ] CaseTrackingID matches CMS format

### Response Validation
- [ ] All required fields present
- [ ] ActivityDate present and valid for all filings
- [ ] PageCount present for all documents
- [ ] CMSIDs stable and persistent
- [ ] CaseSecurity uses exact enum values
- [ ] Timestamps are ISO 8601 UTC
- [ ] No embedded binaries
- [ ] structures:id and s:ref resolve correctly
- [ ] Response time < 3 seconds for typical cases
- [ ] SOAP Fault returned for missing cases

---

## Next Steps

- **Understand behavior rules** → Read [Behavior Guide](./behavior-guide.md)
- **See integration context** → Read [Workflows](./workflows.md)
- **Review examples** → Browse [examples directory](./examples/)
- **Return to overview** → [GetCase Overview](./README.md)

---

**Last Updated:** December 2025