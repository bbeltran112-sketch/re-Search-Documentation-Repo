# Testing & Certification – Client Onboarding

**Navigation:**  
[Home](../../../README.md) › [Client Documentation](../README.md) › [Client Onboarding](./README.md) › Testing & Certification

This page defines the testing phases, expectations, and certification criteria required before a court or CMS vendor is approved for go-live in re:Search.

---

# 🎯 Purpose

The testing and certification process ensures:

- Data accuracy  
- Security rule compliance  
- End-to-end workflow validation  
- Alignment between CMS behavior and re:Search UI behavior  
- Stable ingestion (Batch) or event routing (ECF)  

Testing is a collaboration between **CMS vendors**, **courts**, and **Tyler BIS**.

---

# 🧪 Testing Phases

## 1. **Technical Validation**
The initial verification that your integration *works at a transport and schema level*.

### Includes:
- Connectivity validation (S3/SFTP or SOAP/mTLS)  
- Schema conformance  
- Basic sample ingestion (Batch)  
- Event delivery and GetCase responses (ECF)  
- Confirming required fields and identifiers  
- Security alignment tests (public/confidential/sealed)  

**Outcome:** Vendor is cleared to begin functional testing.

---

## 2. **Functional Testing**
Ensures real-world workflows behave as expected.

### Required Functional Scenarios:
- Case creation / updates  
- Party and attorney changes  
- Case security updates (sealed, confidential, public)  
- Document metadata and document-level security  
- Disposition changes  
- Case deletions / expungements  
- Filing workflows (ECF vendors)  
- UI verification using court SMEs  

**Outcome:** Court and vendor confirm functional readiness.

---

## 3. **End-to-End Certification**
Final review performed by **Tyler BIS**.

### Review Focus:
- Full data consistency  
- No schema failures or ingestion defects  
- No missing identifiers  
- Correct application of JCIT security standards  
- UI behavior matches expected visibility rules  
- All event types routed properly (ECF)  
- Court SMEs sign off on case accuracy  

**Outcome:** Vendor is approved for production deployment.

---

# 📋 Required Test Cases

### Vendors must supply examples for:
- Public cases  
- Confidential cases  
- Sealed cases  
- Juvenile or restricted cases (if applicable)  
- Cases with multiple documents  
- Cases with docket corrections  
- Cases with party/attorney lifecycle updates  
- Filings with multi-document packages (ECF only)

### Courts must validate:
- Case list accuracy  
- Case header data  
- Party roles and visibility  
- Document visibility rules (padlock/gavel mapping)  
- Location/judge assignments  
- Docket detail correctness  

---

# 🧭 Certification Checklist

A vendor is eligible for certification when:

- ✔ All integrations run without schema errors  
- ✔ Security model matches JCIT-aligned rules  
- ✔ CMS identifiers (case numbers, IDs, locations) are stable  
- ✔ EventType usage is correct (ECF only)  
- ✔ No broken document metadata  
- ✔ No missing GetCase fields  
- ✔ Court SMEs validate 10–20 test cases end-to-end  
- ✔ Final regression test passes  

---

# 📅 What Happens After Certification?

Once BIS certifies the integration:

1. A production cutover window is scheduled  
2. Access credentials or S3/SFTP production paths are issued  
3. The vendor receives a go-live checklist  
4. BIS monitors initial ingestion or event activity  
5. Post-launch review is conducted during the first 1–3 weeks  

---

# 🔗 Related Documentation

**Integration**  
- [Integration Modes Overview](../integration-modes/README.md)  
- [Batch Mode Overview](../integration-modes/batch-mode-overview.md)  
- [ECF Mode Overview](../integration-modes/ecf-mode-overview.md)  
- [Non-Integrated Mode Overview](../integration-modes/non-integrated-mode-overview.md)  

**Technical**  
- [API Reference Index](../../technical-documentation/api-reference/README.md)  
- [Support Playbook](../../technical-documentation/support-playbook/README.md)  

---

# ⬅ Back to

**[Client Onboarding Home](./README.md)**  
