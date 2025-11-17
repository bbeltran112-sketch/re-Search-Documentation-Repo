# 4. Registered User Matrix (JCIT 5.4)

The JCIT Registered User Matrix defines visibility for all case categories and types for Role 5 (public users).

This is the core reference for determining which cases are expected to be visible or restricted.

## 4.1 Case Types Public Users Can View

Public users can view:
- Most Civil case types
- Most Probate case types
- Basic case data for some Family case types
- A limited set of Criminal documents:
  - Information
  - Indictment
  - Sentence
  - Judgment
  - Order of Dismissal

These document types are visible only if they are not confidential or sealed.

## 4.2 Case Types Public Users Cannot View

Public users cannot view:
- Confidential cases
- Sealed cases
- Any case with one or more confidential documents
- Juvenile, adoption, mental health cases
- Case categories with statutory restrictions
- Family cases requiring delay windows
- Criminal case documents except for the small approved set

## 4.3 Common Scenarios

Scenario: Case visible to clerk but not to public  
Cause: DocumentSecurity = PrivateView

Scenario: Case appears as “Protected vs Protected”  
Cause: Family case with protected style requirement

Scenario: Criminal case shows no documents  
Cause: Only the approved document types are public

## 4.4 Importance of JCIT-Compliant Case Types

If the CMS submits a case type that does not match JCIT standards:
- Public visibility rules cannot be applied
- Case may not appear at all for Role 5
- Cases may appear under incorrect categories

## [Placeholder] Registered User Matrix Graphic
