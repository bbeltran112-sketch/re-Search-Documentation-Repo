# JCIT Case Type & Category Standards for CMS Vendors  
Version 10.0 – July 2025

Case Categories and Case Types are the foundation of JCIT’s statewide configuration for:

- Public access  
- Role-based permissions  
- Registered User Matrix behavior  
- Document visibility rules  
- Protected Style enforcement  

CMS vendors must treat JCIT case types as a strict, authoritative registry.

## Hard Requirements for CMS Integrators

### 1. Case Types MUST match JCIT values exactly  
Any deviation renders the case non-standard.

### 2. Case Categories MUST match JCIT values exactly  
Categories determine grouping within statewide systems.

### 3. CMS internal values MUST be mapped before sending  
re:SearchTX does **not** perform mapping.

### 4. Case Type impacts:
- Public access  
- Protected Style  
- Document visibility  
- Event display  
- Indexing behavior  

### 5. Incorrect Case Types cause:
- Documents to appear confidential  
- Cases to be hidden from public users  
- Suppressed index information  
- Role 5 access denials  
- Incorrect grouping/filtering in UI  

## Non-Standard Values Are Automatically Restricted  
If a CMS sends a case type not in JCIT’s registry, re:Search will default to more restrictive access.

## Examples of Incorrect CMS Values

| CMS Value | JCIT-Compliant Value |
|-----------|------------------------|
| “Real Property – Other Real Property” | “Other Real Property” |
| “Family – Custody” | “Custody or Visitation” |
| “Civil – Debt Collection” | “Debt/Contract – Debt Collection” |
| “Felony Third Degree” | “Felony 3” |

## Categories Covered by JCIT (All Required)

CMS must map to statewide values for:

- Civil  
- Family  
- Probate  
- Criminal  
- Juvenile  
- MDL  
- Courts of Appeals  

A CMS must *never*:  
- Create additional case types  
- Modify the statewide list  
- Combine types  
- Use jurisdiction-specific naming  

