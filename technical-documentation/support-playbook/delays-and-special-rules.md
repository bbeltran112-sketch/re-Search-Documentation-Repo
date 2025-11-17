# 5. Visibility Delays and Special Rules

Certain case types and scenarios require additional rules to determine visibility for Role 5 users. These rules come from statutory requirements and JCIT Technology Standards.

This section outlines all known delay windows, protected style behaviors, and special-case visibility rules applied in re:SearchTX.

## 5.1 Family Case 31-Day Delay

Many Family case types require a 31-day waiting period before a case becomes visible to public users. This is based on JCIT privacy recommendations to protect sensitive filings.

During the delay:
- Case may appear to internal users
- Case will **not** appear to Role 5 users

## 5.2 Eviction Appeal 180-Day Delay

Some residential eviction appeals (typically from Justice Courts to County Courts) require a 180-day delay before case information can be displayed publicly.

This is often encountered when:
- CaseCategory = Civil, CaseType = Eviction
- Case escalates or is appealed

## 5.3 Protected Style Requirements

JCIT requires certain Family case types to display only a protected style (“Protected vs Protected”) to public users.

Example:

Original:


Protected style does not affect clerk or internal visibility.

## 5.4 Criminal Case Restrictions

Public users may only view five document types for criminal cases:
- Information
- Indictment
- Sentence
- Judgment
- Order of Dismissal

All other documents remain restricted regardless of case-level security.

## 5.5 Juvenile, Adoption, and Mental Health Restrictions

These case types are categorically restricted from public view:
- Juvenile (including delinquency)
- Adoption
- Mental health commitments

No visibility delays apply; these are entirely hidden.

## 5.6 Additional Special Rules

- Cases missing necessary JCIT fields do not appear to the public.
- Any case containing a confidential or sealed document is hidden from Role 5 entirely.
- Cases using non-standard case types may default to restricted visibility.

## [Placeholder] Delay Logic Diagram

