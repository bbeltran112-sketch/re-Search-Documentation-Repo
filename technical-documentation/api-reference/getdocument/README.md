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
