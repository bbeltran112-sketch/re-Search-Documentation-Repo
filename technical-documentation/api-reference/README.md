# Technical Documentation

**Navigation:**  
[Home](../README.md) › Technical Documentation

The Technical Documentation provides all system-level, API, behavioral, and troubleshooting resources used to support integrations with the **re:Search** platform.  
It is intended for:

- CMS vendors and integrators  
- Court IT administrators  
- EFSPs participating in ECF pipelines  
- Tyler BIS support analysts, developers, and TPMs  

Documentation is structured to guide you through API behavior, system rules, visibility logic, architecture, and diagnostic workflows.

---

# 📘 API Reference

The API Reference documents all **SOAP and file-based** APIs used by re:Search.  
Each API page includes:

- Purpose & behavior  
- Required elements  
- EventType and security considerations  
- Inline XML placeholders  
- Link to the full **XML Library**

| API | Description |
|-----|-------------|
| **[API Reference Index](/technical-documentation/api-reference/README.md)** | Entry point for all APIs |
| **[Common Headers & Auth](/technical-documentation/api-reference/common-headers-and-auth.md)** | SOAP headers, authentication, mTLS |
| **[Error Codes](/technical-documentation/api-reference/error-codes.md)** | Error behaviors and response patterns |
| **[RecordFiling](/technical-documentation/api-reference/recordfiling/README.md)** | EFM → CMS filing submission |
| **[NotifyDocketingComplete](/technical-documentation/api-reference/notifydocketingcomplete/README.md)** | CMS docket completion |
| **[NotifyCaseEvent](/technical-documentation/api-reference/notifycaseevent/README.md)** | CMS-originated case/document/party/security updates |
| **[GetCase](/technical-documentation/api-reference/getcase/README.md)** | Authoritative case metadata provider |
| **[GetDocument](/technical-documentation/api-reference/getdocument/README.md)** | Document binary retrieval |


---

# 🛠 Support Playbook

The Support Playbook provides detailed guidance for diagnosing and resolving indexing issues, visibility mismatches, and integration errors.  
It is the primary resource used by BIS Support.

| Section | Description |
|---------|-------------|
| **[Playbook Index](/technical-documentation/support-playbook/README.md)** | Overview of support documentation |
| **[JCIT Roles](/technical-documentation/support-playbook/jcIT-roles.md)** | Visibility & responsibility definitions |
| **[Security Logic](/technical-documentation/support-playbook/security_logic.md)** | CaseSecurity & DocumentSecurity rules |
| **[Registered User Matrix](/technical-documentation/support-playbook/registered-user-matrix.md)** | Case visibility by user role |
| **[Delays & Special Rules](/technical-documentation/support-playbook/delays-and-special-rules.md)** | Criminal, 31-day, 180-day rules |
| **[EventType Logic](/technical-documentation/support-playbook/eventtype-logic.md)** | Processing rules |
| **[Integration Architecture](/technical-documentation/support-playbook/integration-architecture.md)** | System-level flow |
| **[Troubleshooting Guide](/technical-documentation/support-playbook/troubleshooting.md)** | Diagnostic workflow |
| **[Common CMS Mistakes](/technical-documentation/support-playbook/common-cms-mistakes.md)** | Frequent vendor errors |
| **[System Behavior Examples](/technical-documentation/support-playbook/system-behavior-examples.md)** | XML → UI mapping |
| **[Escalation Guidelines](/technical-documentation/support-playbook/escalation-guidelines.md)** | Internal escalation process |
| **[XML Examples (Deprecated)](/technical-documentation/support-playbook/xml-examples.md)** | Deprecated samples |
| **[References](/technical-documentation/support-playbook/references.md)** | Source standards |


---

# 📦 XML Library

The XML Library provides a **centralized set of canonical XML examples** used in integrations.  
These include:

- NotifyCaseEvent examples  
- CaseSecurity & DocumentSecurity patterns  
- GetCaseResponse structures  
- RecordDocketingCallbackMessage  
- Correct, incorrect, and vendor error examples

**Link:**  
➡ **[XML Library](/technical-documentation/xml-library/README.md)**

---

# ❓ FAQs

The FAQ answers the most common questions from courts, CMS vendors, and EFSPs, including:

- Why cases are or are not visible  
- Padlock and confidentiality rules  
- JCIT visibility constraints  
- Criminal, family, and restricted case behavior  
- EventType troubleshooting  

Link:  
➡ **[Visibility & Access FAQ](/technical-documentation/FAQ/re-search-faq.md)**

---

# 🧩 System Behavior & Architecture

This category includes deep-dive explanations of how:

- re:Search indexes case and document data  
- Case and document security interact  
- UI behavior maps to XML  
- Delays, restrictions, and JCIT rules are applied  
- System components interact (CMS, EFM, DAS, re:Search)

These topics appear throughout:

- API Reference  
- Support Playbook  
- XML Library  
- System Behavior Examples  

---

# 🔗 Related Topics

- **[Support Playbook](/technical-documentation/support-playbook/README.md)**  
- **[API Reference](/technical-documentation/api-reference/README.md)**  
- **[FAQ](/technical-documentation/FAQ/re-search-faq.md)**  
- **[XML Library](/technical-documentation/xml-library/README.md)**  

---
