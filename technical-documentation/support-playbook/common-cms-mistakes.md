# 9. Common CMS Mistakes

Many visibility issues arise from vendor misconfigurations or misunderstandings of JCIT or re:SearchTX behavior. This section documents the most frequent issues observed by BIS support.

## 9.1 Top CMS Errors

### Incorrect EventType Used
Vendors often send:
- CaseUpdate instead of CaseSecurity
- DocumentFiling instead of DocumentSecurity

This prevents security updates.

### Non-JCIT CaseType Values
If a case type does not match JCIT standards, public visibility cannot be determined and the case may not appear.

### Missing Required Fields
Missing fields such as:
- FiledDate
- CaseType
- CaseCategory
- Location

will cause cases to fail indexing or appear incomplete.

### Incorrect DocumentSecurity Defaults
Some systems default new documents to PrivateView, unintentionally hiding the case from public users.

### Not Sending DocumentSecurity on Update
If a document status changes (sealed/unsealed), DocumentSecurity must be resent.

### Missing Protected Style for Family Cases
If a case requires protected style but the CMS does not provide it, visibility rules may break.

## 9.2 Suggested Vendor Checklist

Vendors should verify:
- EventType matches the change being made
- CaseType is JCIT-compliant
- DocumentSecurity is correct for each document
- CaseSecurity is properly set and sent
- FileDate is present
- Case has at least one party
- Location matches the expected county/court mapping
