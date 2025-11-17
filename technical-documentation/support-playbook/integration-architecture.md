# 7. Integration Architecture

Understanding how data flows through the CMS, EFM, DAS, and re:SearchTX is essential for identifying issues related to indexing, security, visibility, and document availability.

This section outlines the high-level architecture and the functional roles of each system involved.

## 7.1 Data Flow Overview

General flow:
CMS or EFM
→ NotifyCaseEvent
→ Document Access System (DAS)
→ Search Index
→ re:SearchTX UI

- NotifyCaseEvent triggers processing.
- GetCaseResponse provides full case metadata.
- DAS evaluates the EventType and applies the update.
- The search index is rebuilt or partially updated.
- The UI displays the change according to JCIT rules.

## 7.2 Integrated Counties

Integrated counties provide:
- Full case metadata
- CaseSecurity and DocumentSecurity
- Party and attorney information
- Case type/category mapping

Documents are still routed through the EFM but are associated with CMS metadata.

## 7.3 Non-Integrated Counties

Non-integrated counties rely on EFM metadata for:
- Case index
- Case types/categories (EFM mappings)
- Security values
- Document metadata

In these counties:
- Incorrect EFSP filing metadata can affect visibility
- EFM configuration has significant impact on case categorization

## 7.4 Key Integration Considerations

- Missing or mismatched CaseType values often originate from CMS misconfiguration.
- EFM-driven counties are more likely to have inconsistent or incomplete metadata.
- DAS processing depends entirely on receiving correct EventType and valid GetCaseResponse.

## [Placeholder] Architecture Diagram
