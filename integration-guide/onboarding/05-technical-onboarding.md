# Technical Onboarding – Client Onboarding

**Navigation:**  
[Home](../../../README.md) › [Client Documentation](../README.md) › [Client Onboarding](./README.md) › Technical Onboarding

This page outlines the technical steps required for vendors and courts to begin building and validating their re:Search integration.  

---

# 📌 Purpose

Technical onboarding prepares your CMS, development team, and infrastructure for successful integration with re:Search.  
This includes environment setup, account provisioning, initial configuration, and understanding the APIs or delivery mechanisms used in your chosen integration mode.

---

# 🧰 Technical Setup Requirements

## 1. Environment Access
Before coding begins, confirm:

- S3 or SFTP credentials (Batch Mode)  
- SOAP endpoints + mTLS certs (ECF Mode)  
- Stage environment endpoint + firewall allowlists  
- Optional user accounts for UI verification  

Environment details:  
➡ **[Environment Access →](./02-environment-access.md)**

---

## 2. Integration Mode Configuration

Your CMS must be configured according to the selected integration mode:

| Mode | Required Configuration |
|------|-------------------------|
| **Batch Mode** | JSONL generation, manifest creation, S3/SFTP upload automation |
| **ECF Mode** | SOAP client setup, mTLS cert install, WSDL binding, ECF event handling |
| **Non-Integrated** | BIS-assisted publishing, limited implementation requirements |

Integration Mode details:  
➡ **[Integration Modes Overview →](../integration-modes/README.md)**

---

## 3. Data Mapping & Schema Alignment

CMS vendors must ensure:

- Case identifiers map consistently to court codes  
- CaseCategory, CaseType, DocumentType map to valid market-level values  
- Security classifications are accurate (public, confidential, sealed)  
- JSONL or XML structures conform to schema expectations  

Validate data samples before certification.

---

# 📡 API & Delivery Enablement

Depending on your integration mode, specific APIs or delivery channels must be enabled.

## Batch Mode
- JSONL file generation process  
- Manifest builder  
- S3 or SFTP delivery automation  
- Internal validation pipeline (recommended)

## ECF Mode
- SOAP client configured for:  
  - **RecordFiling**  
  - **NotifyDocketingComplete**  
  - **NotifyCaseEvent**  
  - **GetCase**  
  - **GetDocument** (optional)
- WSDL binding & event routing through EFM  
- Strict adherence to ECF 4.x schemas  

API reference:  
➡ **[API Reference Index →](../../technical-documentation/api-reference/README.md)**

---

# 🧪 Preparing for Testing

Before functional testing begins:

### ✔ Development Teams Should
- Build and deploy the integration into Stage  
- Validate that transport (S3, SOAP, mTLS) is working  
- Produce initial data samples for review  

### ✔ Vendors Should Prepare
- A list of test cases (public, confidential, sealed)  
- Filings that represent typical workflows  
- Document metadata samples  
- Party/attorney/role changes  

### ✔ Courts Should Prepare
- Court SMEs to validate UI output  
- Internal acceptance criteria  
- Known case variations or special workflows  

---

# 🛠 Optional Tools (Recommended)

Although optional, the following internal workflows improve reliability:

- Internal schema validator for JSONL or XML  
- Pre-ingestion QA automation  
- Logging dashboards (CloudWatch, ELK, or similar)  
- Automated retry and alerting for S3/SFTP delivery  

---

# ➡ Next Steps

- **[Testing & Certification →](./06-testing-and-certification.md)**  
- **[Client Onboarding Home →](./README.md)**  
- **[Client Documentation Home →](../README.md)**  
