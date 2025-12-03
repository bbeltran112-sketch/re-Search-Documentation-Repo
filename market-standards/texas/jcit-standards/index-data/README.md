# JCIT Index Data Requirements for CMS Vendors  
Version 10.0 – July 2025

re:SearchTX displays case metadata based on data supplied directly by the CMS.  
CMS vendors must provide all required index fields according to JCIT definitions.

## Required Index Fields

The CMS must transmit values for:

### General Information
- Style  
- Case Category  
- Case Type  
- Case Filed Date  
- Location  
- Judge  
- Case Status  

### Related Cases (if integrated)
- Related Case Number  
- Related Case Type  
- Related Case Filed Date  
- Reason for relation  

### Party Information
- Party Type  
- Party Name  
- Aliases  
- Attorneys for each party  

### Hearings (if integrated)
- Date/Time  
- Hearing Type  
- Judge  
- Location  
- Result  

### Events
- Event date  
- Event type  
- Filing type  
- Document list  
- Page counts  
- Comments  

## Index Data Rules

### 1. Case Category & Case Type must match JCIT exactly  
Incorrect values cause incorrect filtering & visibility.

### 2. Party names must be correctly structured  
Used for search, indexing, and attorney-of-record identification.

### 3. Attorney-of-record drives Role 2 visibility  
Incorrect attorney data → attorneys cannot see their own cases.

### 4. Events must contain accurate document metadata  
Document filenames must be valid and correctly associated.

### 5. Missing fields cause suppressed UI behavior  
re:SearchTX does not infer missing metadata.

### 6. Related case information must use JCIT-approved values  
Local/internal relationships must be mapped.

## Style (Case Title)  
CMS must provide the Style exactly as it should appear publicly or protected.

- Some Family cases require “Protected Style”  
- CMS must still send the original style; re:Search applies protection rules  

Incorrect/missing style causes broken indexing and search failures.
