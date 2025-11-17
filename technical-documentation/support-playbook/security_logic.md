# 3. Security Logic

Security logic in re:SearchTX is governed by two independent fields:

1. CaseSecurity  
2. DocumentSecurity  

Both must be evaluated when diagnosing visibility issues.

## 3.1 Case-Level Security

CaseSecurity determines:
- Whether the case is visible to Role 5
- Whether the case displays a padlock in the UI
- Whether the case requires Protected Style

CaseSecurity values include:
- PublicFilingPublicView
- PublicFilingRestrictedView
- Confidential
- Sealed

A case with CaseSecurity = Confidential or Sealed will not be visible to Role 5.

## 3.2 Document-Level Security

DocumentSecurity determines:
- Visibility of individual documents
- Whether a document displays a padlock icon
- Whether a document is restricted from public access

Common values include:
- Public
- PrivateView
- Sealed

A single confidential or sealed document will hide the entire case from Role 5.

## 3.3 Combined Security Logic

Final visibility =  
CaseSecurity rules  
+ DocumentSecurity rules  
+ User Role  
+ Case Category/Type matrix rules

This means:

- A case can appear internally but not publicly if a document is confidential.
- A case may show public CaseSecurity but still appear hidden if documentSecurity is private.
- CaseSecurity and DocumentSecurity must both be correct for public visibility.

## 3.4 Security Interaction Patterns

Examples:
- CaseSecurity = PublicFilingPublicView + DocumentSecurity = PrivateView → Case not visible to public
- CaseSecurity = Public but eventType wrong → Lock not updating
- CaseSecurity updated but DocumentSecurity not updated → Mixed visibility

## [Placeholder] Security Flowchart
