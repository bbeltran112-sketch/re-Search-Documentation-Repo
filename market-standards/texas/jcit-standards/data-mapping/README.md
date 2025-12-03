# JCIT Data Mapping Requirements for CMS Vendors  
Version 10.0 – July 2025

JCIT requires CMS systems to map internal values to statewide standard values before transmitting them to re:SearchTX.  
Incorrect or non-standard values result in:

- improper role-based access restrictions  
- documents appearing confidential when they should be public  
- cases excluded from the public visibility matrix  
- indexing failures  
- EventType misinterpretation  
- case- or document-level security not updating  

## Mandatory Mapping Areas

CMS vendors must map internal values to JCIT-defined values in the following categories:

1. Case Categories  
2. Case Types  
3. Document Security  
4. Case Security  
5. Criminal Document Types  
6. EventType values  
7. Index information fields (Parties, Attorneys, Events, Hearings, etc.)

## Strict Matching Requirement

JCIT states:

- “Case Category – MUST match case category in JCIT standards to be shown to Registered Users.”
- “Case Type – MUST match case type in JCIT standards to be shown to Registered Users.”

Values MUST match *exactly*.  
Small differences (extra words, punctuation changes, county formatting, abbreviations) will cause incorrect classification.

## Examples of Invalid vs. Valid Case Type Mapping

| CMS-Sent Value (Invalid) | JCIT Standard (Valid) |
|--------------------------|------------------------|
| “Real Property – Other Real Property” | “Other Real Property” |
| “Family Law – Custody w/ Child” | “Custody or Visitation” |
| “Debt Collection – Consumer” | “Debt/Contract – Consumer/DTPA” |
| “Felony – Third Degree” | “Felony 3” |

## EventType Mapping Requirements

re:SearchTX *only processes the fields associated with the EventType provided*.

For example:

- If the CMS sends `CaseSecurity="Sealed"`  
- But EventType is `DocumentSecurityUpdate`

Then re:SearchTX **will not update** the CaseSecurity.

The EventType must correspond to the type of update being performed.

## Document Type Mapping for Criminal Cases

JCIT restricts which criminal documents may be publicly visible.  
Only the following document types are eligible for public release:

- Information  
- Indictment  
- Sentence  
- Judgment  
- Order of Dismissal  

CMS vendors **must map** all internal criminal document types to one of these values if being transmitted as a public document.
