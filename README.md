<p align="center">
  <img src="./assets/images/V3.png" alt="BIS Team Documentation Logo" width="260">
</p>

<h1 align="center" style="margin-bottom: 0;">
  re:Search Documentation
</h1>

<p align="center">
  Comprehensive Client, Technical, API, and Integration Documentation for the re:Search Platform  
  <br>
  Maintained by the BIS Team – Tyler Technologies
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Documentation-Active-blue?style=flat-square">
  <img src="https://img.shields.io/badge/Status-Maintained-brightgreen?style=flat-square">
  <img src="https://img.shields.io/badge/Platform-re:Search-red?style=flat-square">
  <img src="https://img.shields.io/badge/Team-BIS-darkorange?style=flat-square">
</p>

---

## Overview

This repository contains all business, technical, API, integration, and onboarding documentation for  
**Tyler Technologies' re:Search platform**.

The documentation is designed for:

- CMS vendors and integrators  
- Court IT partners  
- Tyler BIS support analysts, TPMs, and developers  
- Internal engineering and product teams  

---

## Quick Start

If you're new to re:Search, begin here:

➡ **[Integration Modes Overview](./client-documentation/integration-modes/README.md)**  
➡ **[API Reference Index](./technical-documentation/api-reference/README.md)**  
➡ **[Client Onboarding Guide](./client-documentation/onboarding/README.md)**  

Or explore the full directory structure below.

---

## Client Documentation

<details>
<summary><strong>Integration Modes</strong></summary>

| Section | Description |
|--------|-------------|
| [Integration Modes Overview](./client-documentation/integration-modes/README.md) | Summary of all supported integration modes |
| [Batch Mode Overview](./client-documentation/integration-modes/batch-mode-overview.md) | JSONL + S3 ingestion model |
| [ECF Mode Overview](./client-documentation/integration-modes/ecf-mode-overview.md) | Event-driven EFM → CMS → re:Search architecture |
| [CIP Mode Overview](./client-documentation/integration-modes/cip-mode-overview.md) | Legacy EJ → re:Search REST model |
| [Non-Integrated Mode Overview](./client-documentation/integration-modes/non-integrated-mode-overview.md) | Counties without CMS integration |
| [Integration Mode Decision Tree](./client-documentation/integration-modes/selection-decision-tree.md) | Helps determine the correct integration mode |

</details>

---

<details>
<summary><strong>Onboarding</strong></summary>

| Section | Description |
|--------|-------------|
| [Onboarding Overview](./client-documentation/onboarding/README.md) | Full onboarding lifecycle, expectations, and milestones |

</details>

---

## Technical Documentation

<details>
<summary><strong>API Reference (Core)</strong></summary>

### API Index & Shared Artifacts

| Section | Description |
|--------|-------------|
| [API Reference Overview](./technical-documentation/api-reference/README.md) | Central index for all APIs |
| [Common Headers & Auth](./technical-documentation/api-reference/common-headers-and-auth.md) | SOAP headers and authentication |
| [Error Codes](./technical-documentation/api-reference/error-codes.md) | Standardized error schema |

### Core APIs

| API | Description |
|-----|-------------|
| [RecordFiling](./technical-documentation/api-reference/recordfiling/README.md) | EFM → CMS filing submission |
| [NotifyDocketingComplete](./technical-documentation/api-reference/notifydocketingcomplete/README.md) | CMS docketing acknowledgment |
| [NotifyCaseEvent](./technical-documentation/api-reference/notifycaseevent/README.md) | CMS-originated case events |
| [GetCase](./technical-documentation/api-reference/getcase/README.md) | Authoritative case metadata |
| [GetDocument](./technical-documentation/api-reference/getdocument/README.md) | Retrieves document binaries |

### API Examples

| API | Examples Folder |
|-----|-----------------|
| RecordFiling | [Examples](./technical-documentation/api-reference/recordfiling/examples/) |
| NotifyDocketingComplete | [Examples](./technical-documentation/api-reference/notifydocketingcomplete/examples/) |
| NotifyCaseEvent | [Examples](./technical-documentation/api-reference/notifycaseevent/examples/) |
| GetCase | [Examples](./technical-documentation/api-reference/getcase/examples/) |
| GetDocument | [Examples](./technical-documentation/api-reference/getdocument/examples/) |

</details>

---

## Integration Guides

<details>
<summary><strong>Guides</strong></summary>

| Section | Description |
|---------|-------------|
| Batch Mode Guide *(coming soon)* | JSONL schema, manifest rules, ingestion, troubleshooting |
| ECF Mode Guide *(coming soon)* | WSDL, event flow, security, behavior mapping |

</details>

---

## System Behavior

<details>
<summary><strong>Behavior Guides</strong></summary>

| Section | Description |
|---------|-------------|
| System Behavior Guide *(coming soon)* | How API data maps to UI and indexing rules |

</details>

---

## Runbooks

<details>
<summary><strong>Operational Playbooks</strong></summary>

| Section | Description |
|---------|-------------|
| Batch Troubleshooting *(coming soon)* | S3 ingestion, JSONL validation issues |
| ECF Troubleshooting *(coming soon)* | NCE/NDC issues, GetCase mismatches, visibility conflicts |

</details>

---

## Dark Mode Friendly

This README is fully compatible with GitHub dark and light modes, including images and tables.

---

## Contribution Standards

- Use **lowercase dashed filenames**  
- Use `README.md` for directory-level indexes  
- Avoid MkDocs syntax in the repo  
- Diagrams should be mermaid or labeled placeholders  
- PRs must describe scope and changes  

---

## Support

For internal BIS/Tyler contributors, contact:

**BIS – Bryce Beltran**
