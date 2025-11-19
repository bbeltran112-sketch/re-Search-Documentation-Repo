# JCIT Security Standards for CMS Integrations  
Version 10.0 – July 2025

Security in re:SearchTX depends entirely on accurate CMS-supplied values.  
CMS vendors must map all case and document security to one of the three allowed statewide values:

- Public  
- Confidential  
- Sealed  

## Case-Level Security (CaseSecurity)

Controls visibility of the entire case.

Allowed values:

- Public  
- Confidential  
- Sealed  

Incorrect values default to restrictive behavior.

### Important:
If the CMS sends the correct CaseSecurity value but uses the wrong EventType, re:SearchTX ignores the update entirely.

## Document-Level Security (DocumentSecurity)

Controls visibility of individual documents.

Allowed values:

- Public  
- Confidential  
- Sealed  

CMS values such as “PublicView” or “PrivateView” must be mapped before transmission.

## Role-Based Visibility (Summary)

### Public  
Visible to all roles except when restricted by case type.

### Confidential  
Visible to:
- Attorneys on the case  
- Firm staff  
- Clerks  
- Justice partners  
- Judges  

### Sealed  
Visible only to:
- Judges  
- Clerk Administrators  

All documents hidden from everyone else.

## Security + Case Type Interaction  
Even when security is correct, an incorrect case type can suppress access.

Example:

- CaseSecurity = Public  
- CaseType = Invalid / Non-standard  

Result: Case may still appear confidential or hidden.

CMS must ensure both CaseSecurity and CaseType are valid for visibility to function as expected.
