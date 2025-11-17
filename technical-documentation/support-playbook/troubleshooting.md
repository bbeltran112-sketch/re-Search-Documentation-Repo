# 8. Troubleshooting Guide

This guide provides a structured workflow for diagnosing visibility, indexing, and security issues within re:SearchTX.

Visibility issues are usually related to:
- Security values
- Role-based restrictions
- EventType mismatches
- Data quality or missing fields
- JCIT-driven case-type rules

## 8.1 Standard Troubleshooting Workflow

1. Determine the user's role (often Role 5).
2. Check CaseSecurity in the latest GetCaseResponse.
3. Check DocumentSecurity for all documents.
4. Validate case type and category against JCIT Registered User Matrix.
5. Check for applicable delays (31-day or 180-day).
6. Confirm correct EventType was used to update the values.
7. Inspect GetCaseResponse for missing required fields.
8. Identify integration mode (CMS vs EFM).
9. Request NotifyCaseEvent and GetCaseResponse XML from vendor.
10. Compare message timestamps to verify order and recency.

## 8.2 Scenario-Based Troubleshooting

### Scenario A: Case visible internally but not publicly  
Cause: At least one document has DocumentSecurity = PrivateView or Sealed.

### Scenario B: Padlock not clearing after unsealing  
Cause: CMS sent CaseUpdate instead of CaseSecurity.

### Scenario C: Case missing from public search  
Possible causes:
- Family 31-day delay  
- Eviction appeal 180-day delay  
- CaseCategory or CaseType not matching JCIT  
- Incorrect or missing filed date  
- Case contains confidential documents

### Scenario D: Criminal case missing documents  
Cause: Only five document types are allowed for public view.

### Scenario E: Vendor claims case is unsealed but re:Search still shows locked  
Cause: DocumentSecurity not updated or wrong EventType.

## 8.3 When to Request Message Traffic

Request NotifyCaseEvent and GetCaseResponse when:
- A user reports security not updating
- Case does not appear after CMS update
- Document is visible internally but not externally
- Vendor states “we unsealed it” but UI contradicts that

## [Placeholder] Troubleshooting Flowchart
