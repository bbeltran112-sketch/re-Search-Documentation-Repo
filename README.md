# re:Search Documentation
<p align="center">
  <img src="assets/images/V3.png" alt="re:Search Logo" width="240">
</p>



This repository contains the complete business, technical, onboarding, and integration documentation for **Tyler Technologies’ re:Search platform**.  

The structure is organized for:
- CMS vendors and integrators  
- Court partners and IT staff  
- Tyler BIS support analysts, TPMs, and developers  

All content is written in GitHub-native Markdown, using consistent naming and folder patterns.

---

## Client Documentation

### Integration Modes

| Section | Description |
|--------|-------------|
| [Integration Modes Overview](./client-documentation/integration-modes/README.md) | Summary of all supported integration models |
| [Batch Mode Overview](./client-documentation/integration-modes/batch-mode-overview.md) | Standard modern mode (JSONL + S3) |
| [ECF Mode Overview](./client-documentation/integration-modes/ecf-mode-overview.md) | Event-driven EFM → CMS → re:Search |
| [CIP Mode Overview](./client-documentation/integration-modes/cip-mode-overview.md) | Legacy EJ → re:Search REST flow |
| [Non-Integrated Mode Overview](./client-documentation/integration-modes/non-integrated-mode-overview.md) | Courts with no automated CMS integration |
| [Selection Decision Tree](./client-documentation/integration-modes/selection-decision-tree.md) | Determine which mode applies to each market |

---

### Client Onboarding

| Section | Description |
|--------|-------------|
| [Client Onboarding Guide](./client-documentation/onboarding/README.md) | End-to-end onboarding workflow for CMS vendors and courts |

---

## Technical Documentation

### API Reference – Index & Shared Artifacts

| Section | Description |
|--------|-------------|
| [API Reference Index](./technical-documentation/api-reference/README.md) | All re:Search APIs and structure |
| [Common Headers & Auth](./technical-documentation/api-reference/common-headers-and-auth.md) | Required SOAP headers, mTLS, envelope structure |
| [Error Codes](./technical-documentation/api-reference/error-codes.md) | ECF-aligned fault codes and error rules |

---

### Core API Specifications

| API | Description |
|-----|-------------|
| [RecordFiling](./technical-documentation/api-reference/recordfiling/README.md) | Deliver accepted filings from EFM → CMS |
| [NotifyDocketingComplete](./technical-documentation/api-reference/notifydocketingcomplete/README.md) | CMS → EFM docket confirmation |
| [NotifyCaseEvent](./technical-documentation/api-reference/notifycaseevent/README.md) | CMS-originated updates to re:Search |
| [GetCase](./technical-documentation/api-reference/getcase/README.md) | Retrieve authoritative case record |
| [GetDocument](./technical-documentation/api-reference/getdocument/README.md) | Retrieve document metadata & binaries |

---

### API Example Collections

| API | Examples Folder |
|-----|-----------------|
| RecordFiling | [Examples](./technical-documentation/api-reference/recordfiling/examples/) |
| NotifyDocketingComplete | [Examples](./technical-documentation/api-reference/notifydocketingcomplete/examples/) |
| NotifyCaseEvent | [Examples](./technical-documentation/api-reference/notifycaseevent/examples/) |
| GetCase | [Examples](./technical-documentation/api-reference/getcase/examples/) |
| GetDocument | [Examples](./technical-documentation/api-reference/getdocument/examples/) |

---

## Integration Guides

| Section | Description |
|---------|-------------|
| Batch Mode Guide *(coming soon)* | Schema details, manifest rules, ingestion behavior |
| ECF Mode Guide *(coming soon)* | Endpoint configuration, WSDLs, EFM routing |

---

## System Behavior

| Section | Description |
|---------|-------------|
| System Behavior Guide *(future)* | Explains UI behavior, indexing logic, event relationships |

---

## Runbooks

| Section | Description |
|---------|-------------|
| Batch Troubleshooting *(future)* | Debug ingestion, validation, and S3 upload issues |
| ECF Troubleshooting *(future)* | Debug NCE, NDC, RecordFiling, GetCase failures |

---

## Contribution Standards

- Use **lowercase dashed filenames** (e.g., `getcase-response-annotated.xml`)  
- Use **README.md** for all folder index pages  
- Avoid MkDocs-specific formatting  
- Keep diagrams as **“diagram placeholder”** unless Mermaid is required  
- Submit PRs to the `main` branch with clear change descriptions  

---

## Questions or Support

For internal support, contact:  
**BIS – Bryce Beltran**
