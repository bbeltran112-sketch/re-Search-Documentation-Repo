# GetDocument

**Navigation:**  
[Home](../../../README.md) › [Technical Documentation](../../README.md) › [API Reference](../README.md) › GetDocument

GetDocument retrieves a document binary for viewing in re:SearchTX.  
Access is controlled by case security, document security, and JCIT visibility rules.

---

## Purpose

GetDocument is used to:

- Retrieve the binary PDF/TIFF document  
- Enforce JCIT visibility and security  
- Display documents in re:Search’s viewer  
- Validate document-level metadata accuracy  

---

## Transport and Protocol

| Property | Value |
|---------|--------|
| Direction | re:Search → CMS |
| Protocol | SOAP 1.2 over HTTPS |
| Security | Mutual TLS |
| Standard | OASIS ECF |
| Message Type | GetDocumentRequest / GetDocumentResponse |

Authentication details:  
[Common Headers & Auth](../common-headers-and-auth.md)

---

## When This API Is Used

re:Search calls GetDocument:

- When a user clicks a document in the UI  
- When indexing requires document metadata validation  
- When the system detects a missing or stale binary  

---

## Behavior Summary

re:Search:

- Evaluates CaseSecurity and DocumentSecurity  
- Confirms JCIT role-based access  
- Requests the document binary  
- Displays or denies access accordingly  

Incorrect security values = locked documents or incorrect visibility.

---
## Processing Workflow
<!-- XML Placeholder: Insert sample GetDocumentResponse here -->

---
## XML Example
```xml
<DocumentResponseMessage
    xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:DocumentResponseMessage-4.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CommonTypes-4.0"
    xmlns:tyler="urn:tyler:ecf:extensions:Common"
    xmlns:nc="http://niem.gov/niem/niem-core/2.0"
    xmlns:s="http://niem.gov/niem/structures/2.0"
    xmlns:j="http://niem.gov/niem/domains/jxdm/4.0"
    xmlns:courtNS="urn:tyler:ecf:extensions:CourtCase"
    xmlns:urn="urn:oasis:names:tc:legalxml-courtfiling:wsdl:WebServiceMessagingProfile-Definitions-4.0"
    xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
    xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
    <!-- [EXPECTED] echo of sender -->
    <SendingMDELocationID xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CommonTypes-4.0">
        <IdentificationID xmlns="http://niem.gov/niem/niem-core/2.0">
            https://www.idocket.com/
        </IdentificationID>
    </SendingMDELocationID>
    <!-- [REQUIRED] success when 0 -->
    <Error xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CommonTypes-4.0">
        <ErrorCode>0</ErrorCode>
        <ErrorText>No error</ErrorText>
    </Error>
    <Document xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CommonTypes-4.0">
        <!-- [REQUIRED] authoritative document identifiers -->
        <DocumentIdentification xmlns="http://niem.gov/niem/niem-core/2.0">
            <IdentificationID>13763456</IdentificationID>
            <IdentificationCategoryText>CMSID</IdentificationCategoryText>
        </DocumentIdentification>
        <!-- [EXPECTED] optional metadata for audit/UI -->
        <DocumentMetadata>
            <RegisterActionDescriptionText xmlns="http://niem.gov/niem/domains/jxdm/4.0">
                evtapp
            </RegisterActionDescriptionText>
        </DocumentMetadata>
        <DocumentRendition>
            <!-- [EXPECTED] high-level description -->
            <DocumentDescriptionText xmlns="http://niem.gov/niem/niem-core/2.0">
                APPLICATION
            </DocumentDescriptionText>
            <!-- [EXPECTED] post date/time -->
            <DocumentPostDate xmlns="http://niem.gov/niem/niem-core/2.0">
                <DateTime>2025-01-29T00:00:00</DateTime>
            </DocumentPostDate>
            <DocumentRenditionMetadata>
                <DocumentAttachment>
                    <!-- [REQUIRED] inline binary payload -->
                    <BinaryBase64Object xmlns="http://niem.gov/niem/niem-core/2.0">
                        rt556gfdyKGphaW1lKS9DcmVhdGlvbkRhdGUoRDoyMDA5MTAxMzE0NTEzMykvQ3JlYXRvcihQU2NyaXB0NS5kbGwgVmV+GX0y38QNfN1BzCmVuZHN0cmVhbQ1lbmRvYmoNc3RhcnR4cmVmDTg2Njk0DSUlRU9GCg==
                    </BinaryBase64Object>
                    <!-- [EXPECTED] rendition metadata -->
                    <BinaryDescriptionText xmlns="http://niem.gov/niem/niem-core/2.0">APPLICATION</BinaryDescriptionText>
                    <BinaryFormatStandardName xmlns="http://niem.gov/niem/niem-core/2.0">PUB</BinaryFormatStandardName>
                    <BinaryLocationURI xmlns="http://niem.gov/niem/niem-core/2.0">1376273.pdf</BinaryLocationURI>
                    <BinarySizeValue xmlns="http://niem.gov/niem/niem-core/2.0">87217</BinarySizeValue>
                    <BinaryCategoryText xmlns="http://niem.gov/niem/niem-core/2.0">LEAD</BinaryCategoryText>

                    <!-- [NEW][OPTIONAL] integrity hints -->
                    <!-- <Hash xmlns="http://niem.gov/niem/niem-core/2.0">sha256:...</Hash> -->
                </DocumentAttachment>
            </DocumentRenditionMetadata>
        </DocumentRendition>
    </Document>
</DocumentResponseMessage>
```

---

## Example XML Files

| File | Description |
|------|-------------|
| [getdocument-request-annotated.xml](./examples/getdocument-request-annotated.xml) | Annotated request |
| [getdocument-request-clean.xml](./examples/getdocument-request-clean.xml) | Clean request |
| [getdocument-response-annotated.xml](./examples/getdocument-response-annotated.xml) | Annotated response |

---

## Related API Pages

- [GetCase](../getcase/README.md)  
- [RecordFiling](../recordfiling/README.md)  
- [NotifyCaseEvent](../notifycaseevent/README.md)  
- [NotifyDocketingComplete](../notifydocketingcomplete/README.md)  

---

## Related Topics

- [Security Logic](../../support-playbook/security-logic.md)
- [Registered User Matrix](../../support-playbook/registered-user-matrix.md)
- [Troubleshooting Guide](../../support-playbook/troubleshooting.md)
- [Full XML Library](../../xml-library/xml-library.md)




























# GetDocument – re:Search Integration API

`GetDocument` is the API re:Search uses to retrieve **document binaries** from the CMS.  
Every time a user opens a document in the re:Search UI, this operation is called.

This API does **not** return case metadata or filing information — only the requested binary stream (PDF, TIFF, JPEG, etc.) and minimal metadata required to fulfill the file request.

GetDocument is required in all **ECF Mode** and **Hybrid** integrations unless documents are pre-staged via Batch Mode (rare).

---

## Purpose

GetDocument allows re:Search to:

- Retrieve document binaries securely on demand  
- Enforce document-level security from CMS rules  
- Maintain a single authoritative source for document content  
- Fetch lead documents, attachments, exhibits, and supplemental filings  

GetDocument is **never used** to return metadata about filings, parties, or cases — that information must come from `GetCase`.

---

## Transport & Protocol

| Property | Value |
|---------|--------|
| Direction | **re:Search → CMS** |
| Protocol | SOAP 1.2 over HTTPS |
| Security | Mutual TLS (mTLS) |
| Standard | OASIS ECF 4.x DocumentQuery/DocumentResponse |
| Messages | `GetDocumentRequest` / `GetDocumentResponse` |

Header requirements:  
➡️ `../../common/common-headers-and-auth.md`

---

## When re:Search Calls GetDocument

re:Search calls GetDocument when:

1. A user clicks *View Document* in the UI  
2. A PDF thumbnail or preview is required  
3. Background validation jobs confirm document availability  
4. A document ID appears in GetCaseResponse and re:Search needs to verify accessibility  

**Important:**  
If GetCaseResponse provides incorrect or missing `CMSID` / `DocumentSecurity`, GetDocument may fail — these failures must be fixed in GetCase, not in GetDocument.

---

## Required & Expected Fields

### Request Fields

| Element | Requirement | Notes |
|---------|------------|-------|
| `nc:IdentificationID` | **REQUIRED** | Must match the CMS document ID in GetCase |
| `CourtIdentifier` | EXPECTED | Needed in multi-court CMS implementations |
| Authentication headers | **REQUIRED** | mTLS + Auth header chain |

---

### Response Fields

| Element | Requirement | Notes |
|---------|------------|-------|
| Binary stream (`nc:Base64BinaryObject`) | **REQUIRED** | Must be valid Base64 PDF/TIFF/etc. |
| `nc:DocumentDescriptionText` | EXPECTED | Used in UI display |
| `nc:BinaryFormatStandardName` | EXPECTED | PDF, TIFF, JPEG |
| `tyler:DocumentSecurity` | EXPECTED | Should match what is in GetCase |
| `nc:BinarySizeValue` | EXPECTED | Helps UI pre-allocation |
| `tyler:PageCount` | EXPECTED | If available; improves UI |

---

## High-Level Workflow

**Diagram placeholder**

(Insert mermaid diagram or image later.)

### Steps

1. User requests document  
2. re:Search issues `GetDocumentRequest`  
3. CMS authenticates request via mTLS  
4. CMS returns Base64-encoded binary  
5. re:Search delivers document to UI  
6. Security is validated per CMS response + local rules  

---

## Document Mapping Rules

GetDocument relies on the `nc:IdentificationID` provided in **GetCaseResponse**, specifically within:

- `ecf:DocumentRendition`
- `ecf:DocumentAttachment`
- `tyler:DocumentAttachmentIdentification`

If the CMS changes document IDs or fails to reference attachments correctly:

| Problem | Result |
|---------|--------|
| Missing CMSID | Document fails to load |
| CMSID changes | Duplicate or “missing document” errors |
| Incorrect `DocumentSecurity` | Document appears in UI when it shouldn’t (or vice versa) |
| Wrong BinaryLocationURI (Batch Mode) | Document retrieval fails |

---

## File Types & Expectations

| File Type | Supported | Notes |
|-----------|-----------|-------|
| PDF | ✔ | Most common |
| TIFF | ✔ | Multi-page allowed |
| JPEG/PNG | ✔ | Usually exhibits or attachments |
| Word/Excel | ✔ | CMS must Base64-encode raw binary |
| Audio/Video | ⚠ | Allowed but not recommended |

---

## Error Handling

Typical CMS-side validation must include:

- Document not found → SOAP Fault  
- Unauthorized (document security) → SOAP Fault  
- CMS connectivity failure → SOAP Fault  
- Unsupported format → SOAP Fault  
- Invalid Base64 → SOAP Fault  

On the re:Search side, these appear as:

- "Document unavailable"  
- "Security restrictions apply"  
- "CMS returned an invalid or empty response"  

---

## Example Files

| Example | File |
|---------|------|
| Annotated Request | [examples/getdocument-request-annotated](./examples/getdocument-request-annotated) |
| Clean Request | [examples/getdocument-request-clean](./examples/getdocument-request-clean) |
| Annotated Response | [examples/getdocument-response-annotated](./examples/getdocument-response-annotated) |

---

## Related API Pages

| API | Link | Purpose |
|------|------|---------|
| **GetCase** | [../getcase/README.md](../getcase/README.md) | Provides CMSID + document metadata |
| **NotifyCaseEvent** | [../notifycaseevent/README.md](../notifycaseevent/README.md) | Triggers case updates |
| **RecordFiling** | [../recordfiling/README.md](../recordfiling/README.md) | Introduces new documents |
| **NotifyDocketingComplete** | [../notifydocketingcomplete/README.md](../notifydocketingcomplete/README.md) | Provides CMS document IDs |

---

Back to API Reference Index:  
➡️ [../README.md](../README.md)
