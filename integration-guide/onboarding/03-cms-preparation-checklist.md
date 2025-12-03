# CMS Preparation Checklist – Client Onboarding

**Navigation:**  
[Home](../../../README.md) › [Client Documentation](../README.md) › [Client Onboarding](./README.md) › CMS Preparation Checklist

This checklist outlines what each CMS vendor must complete before beginning integration work with re:Search.  
Completing these steps ensures smoother onboarding, faster testing, and accurate data delivery.

---

# 📌 Summary

Vendors should finalize environment access, system configuration, security requirements, and data preparation **before** development begins.

This page serves as a readiness checklist for CMS, development, and technical operations teams.

---

# 🧰 CMS Technical Readiness

## Integration Mode Requirements

| Mode | Requirements |
|------|-------------|
| **Batch** | Ability to generate JSONL + manifest; S3 or SFTP delivery |
| **ECF** | SOAP/XML, mTLS, ECF 4.x messaging, EFM connectivity |
| **Non-Integrated** | Limited—BIS-assisted publishing workflows |

Verify your CMS supports the correct mode before proceeding.  
Use: **[Integration Modes Overview](../integration-modes/README.md)**

---

## Data Preparation

Before beginning integration, ensure the CMS can:

- Generate complete case, party, filing, and document metadata  
- Support stable case identifiers and consistent CourtLocation/CaseType mapping  
- Produce schema-compliant data for the selected integration mode  
- Provide accurate case security classifications  
- Provide reliable document metadata (filing date, document type, confidential flags, etc.)  

---

## Security & Visibility Rules

Your CMS must maintain a correct and consistent security model:

| Area | Expectation |
|-------|-------------|
| **Case Security** | Public, confidential, sealed logic is required |
| **Document Security** | Public vs. PrivateView visibility must be accurate |
| **Roles/Restrictions** | Must align with JCIT-level access expectations |
| **Internal Overrides** | Document how your system treats overrides or clerk-only fields |

These rules directly determine what re:Search can legally display.

---

## Logging & Error Handling

Prior to onboarding, vendors should confirm:

- Event logging is enabled  
- SOAP/transport logs (ECF) or delivery logs (Batch) are monitored  
- Errors can be surfaced to support teams  
- There is a clear internal escalation path  

This is essential for debugging ingestion issues.

---

# 📡 Environment Access

Before integration begins, ensure your team has:

- S3/SFTP credentials (Batch)  
- SOAP endpoints + mTLS certs (ECF)  
- Portal user accounts (if required for testing)  
- Stage environment access  

Environment URL reference:  
➡ **[Environment Access →](./02-environment-access.md)**

---

# 🛠 Pre-Development Checklist

This checklist should be completed internally before development starts:

### ✔ Technical Setup
- Development environment ready  
- Access credentials stored securely  
- Ability to test against Stage environment  

### ✔ CMS Capabilities
- Verified support for integration mode  
- Schema/format generation confirmed  
- Document metadata available and consistent  

### ✔ Court Preparedness
- Court contacts aligned  
- Case samples identified (public + restricted)  
- Test scenarios prepared  

### ✔ Vendor Workflows
- Logging enabled  
- Sandbox environment available  
- Developer assigned to case-security logic  

---

# ➡ Next Steps

- **[Integration Mode Selection →](./04-integration-mode-selection.md)**  
- **[Client Onboarding Home →](./README.md)**  
- **[Client Documentation Home →](../README.md)**  
