# JCIT Data Exchange Requirements for CMS Systems  
Version 10.0 – July 2025

The Texas Administrative Code requires all case-related data exchanges to use the OASIS LegalXML ECF standard, a NIEM-conformant framework. CMS vendors participating in ECF or direct integrations must structure content accordingly.

## Required Exchange Standard  
All CMS → EFM → re:SearchTX exchanges must follow:

- OASIS LegalXML ECF specifications  
- NIEM data components & element definitions  
- JCIT case-type and category registries  
- JCIT security definitions  
- JCIT index data definitions  

These ensure interoperability and consistent interpretation of case metadata.

## CMS Responsibilities in the Data Exchange Model

A CMS is responsible for transmitting:

1. Case types (must match JCIT-defined)  
2. Case categories (must match JCIT-defined)  
3. Case security status  
4. Document security status  
5. Complete case index information  
6. Criminal document type classification  
7. Accurate, event-consistent updates  

## Key Rules for CMS Vendors

### 1. CMS MUST NOT invent, extend, or alter statewide values  
No modified values (e.g., “Real Property – Other Real Property”).  
Only JCIT values are permitted.

### 2. CMS MUST update re:Search using the correct EventType  
Incorrect event routing = updates ignored.

### 3. CMS MUST send index fields in a NIEM-compliant structure  
Required fields include:  
- Case category  
- Case type  
- Parties  
- Attorneys  
- Events  
- Filings  
- Hearings (if integrated)  
- Related cases  

### 4. CMS MUST treat re:Search as a “read-only consumer”  
The CMS is authoritative; re:Search does not infer or adjust incorrect data.

### 5. CMS MUST transmit document links, filenames & page counts correct  
These values drive re:Search indexing and display logic.

Failure in any above category results in downstream visibility and classification issues.

