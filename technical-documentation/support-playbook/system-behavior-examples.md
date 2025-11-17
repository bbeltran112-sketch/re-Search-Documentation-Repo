# 10. System Behavior Examples

This section provides concrete examples of common behaviors observed in re:SearchTX. These examples illustrate how CaseSecurity, DocumentSecurity, EventTypes, delays, and case-type rules interact in real scenarios.

## Example 1: Case-Level Lock Not Clearing

### Symptoms
- Vendor unseals case in CMS
- CaseSecurity updated correctly in GetCaseResponse
- re:SearchTX still shows the case as sealed
- Padlock icon remains

### Root Cause
Incorrect EventType sent.

Vendor used: eventType = CaseUpdate

Correct value needed:eventType = CaseSecurity

re:SearchTX ignores security changes unless the EventType reflects the type of change.

---

## Example 2: Case Visible Internally but Not to Public Users

### Symptoms
- Clerk and CMS confirm case is public
- Internal roles can access it
- Public users (Role 5) cannot see the case at all

### Root Cause
At least one document is: documentSecurity = PrivateView

Public users cannot view any case with a confidential or sealed document.

---

## Example 3: Criminal Case Shows No Documents

### Symptoms
- Criminal case appears publicly
- No documents available
- Vendor asks why filings are not visible

### Root Cause
Role 5 can only see these criminal document types:
- Information
- Indictment
- Sentence
- Judgment
- Order of Dismissal

All other document types are restricted from public view.

---

## Example 4: Family Case Appears as “Protected vs Protected”

### Symptoms
- Case style changed unexpectedly for public users
- Clerk sees full party names
- Public sees generic style

### Root Cause
Family case requiring Protected Style per JCIT rules.

Vendor correctly submitted: protectedStyle = “Protected vs Protected”


Or DAS applied protected style due to case type.

---

## Example 5: Case Missing from Search Entirely

### Symptoms
- Case does not appear in internal or public search
- Vendors report “we submitted the case”

### Root Causes
Often one of the following:
- Missing FiledDate
- Missing CaseType
- CaseType not mapped to JCIT standards
- Invalid caseCategory
- Message processed out of order
- CaseSecurity not included in initial CaseFiling
- GetCaseResponse missing required structure

---

## [Placeholder] Sequence Diagram




