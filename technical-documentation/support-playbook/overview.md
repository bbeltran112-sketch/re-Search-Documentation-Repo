# 1. System Overview

re:SearchTX is a statewide case lookup and document access platform driven by data from two primary sources:

1. Integrated CMS systems
2. The Electronic Filing Manager (EFM)

Understanding which system provides which data is critical for troubleshooting visibility issues.

## 1.1 Integrated Counties

Integrated counties provide their own:
- Case index and core metadata
- Case-level security
- Document metadata
- Party and attorney information
- Case type and category

Documents for integrated counties are still routed through the EFM, but document metadata may originate from either system depending on the configuration.

## 1.2 Non-Integrated Counties

Non-integrated counties rely entirely on the EFM for:
- Case index metadata
- Case category and case type (EFM mappings)
- Document metadata
- Security information

This means visibility for non-integrated counties is affected by:
- The EFM’s internal configuration and mappings
- The standardization of document types
- The consistency of filing metadata from EFSPs

## 1.3 Required Fields for Indexing

re:SearchTX requires the following minimum fields for a case to appear in search:

- Case Category
- Case Type
- Filed Date
- Location (County/Office)
- Style or Protected Style
- CaseSecurity
- DocumentSecurity

Missing required fields may cause:
- Case not appearing in search
- Case appearing without documents
- Case visible internally but not publicly

## 1.4 Architecture Overview

The general data flow is:

CMS or EFM → NotifyCaseEvent → DAS → Search Index → re:SearchTX UI

## [Placeholder] Architecture Diagram
