# Case Type Mismatch: JCIT Standard Enforcement Issue

## Overview
re:SearchTX enforces JCIT-standard case categories and case types when determining public visibility and access rules. If the CMS sends a case type value that does not exactly match a JCIT-approved value, the system cannot associate the case with any access rule for Registered Users (Role 5). When this occurs, the case and/or documents may display a padlock even when caseSecurity or documentSecurity values indicate public access.

This document describes the root cause, applicable JCIT standards, and required vendor actions.

---

## Problem Description
During testing, the CMS submitted the following case type value: Injury or Damage - Other Injury of Damage


This value does not exist in the official JCIT case type list. Because it cannot be matched to a JCIT standard case type, re:SearchTX applies a restricted-visibility rule. As a result, the Public profile is blocked from viewing the case or its documents, and a padlock icon appears.

This occurs even when:
- caseSecurity = PublicFilingPublicView  
- documentSecurity values are set correctly  
- A NotifyCaseEvent is sent  

The mismatch prevents re:Search from applying the appropriate public-access rules.

---

## JCIT Standard Case Types (Authoritative List)
Under **Civil – Injury or Damage**, the permissible JCIT case types (Version 10.0, July 2025) are:

- Assault/Battery  
- Construction  
- Defamation/Libel/Slander  
- Malpractice – Accounting  
- Malpractice – Legal  
- Malpractice – Medical  
- Malpractice – Other Professional Liability  
- Motor Vehicle Accident  
- Premises  
- Product Liability – Asbestos/Silica  
- Product Liability – Other  
- Other Injury or Damage

Source: JCIT Technology Standards v10.0, Sections 4.1.1 and 5.4.1.  
(Internal reference: technology-standards.pdf)

The correct value the CMS should send is:Other Injury or Damage

The string must match exactly for re:Search to map the case type to its visibility rules.

---

## How re:Search Applies Case Type Mapping
1. Case type is evaluated against the JCIT-approved list.  
2. If the value matches exactly, re:Search applies the corresponding access rule.  
3. If no match is found, the case is treated as restricted for at least one user role.  
4. Any restricted-role mapping triggers the padlock icon.  
5. caseSecurity/documentSecurity do not override a case-type mismatch.

---

## Resolution Steps
1. Vendor updates the CMS mapping so the case type matches a JCIT standard exactly.  
2. Vendor resends the appropriate NotifyCaseEvent.  
   - If the case type changed: eventType = `Case`  
   - If security changed: eventType = `CaseSecurity`  
3. re:SearchTX reindexes the case and updates public access.  
4. Padlock clears for the Public profile once case type and security values align.

---

## Vendor Guidance
Vendors must ensure all case types transmitted via GetCaseResponse or NotifyCaseEvent match JCIT standards verbatim. Any difference in punctuation, wording, spacing, or grouping prevents correct rule mapping.

We recommend periodically validating CMS case type dictionaries against JCIT Version 10.0 and future updates.

