# JCIT EventType Requirements for CMS Vendors  
Version 10.0 – July 2025

EventType controls which data fields re:SearchTX evaluates during an update.  
If a CMS sends an update using the wrong EventType, re:SearchTX will ignore all unrelated fields.

## Key Rule
**re:Search only processes fields associated with the EventType included in the message.**

## Example of Incorrect Behavior

If CMS sends:

```
<CaseSecurity>Sealed</CaseSecurity>
```

but EventType = `DocumentSecurityUpdate`  
→ re:SearchTX **will not update** the case security.

## Essential EventType Categories for CMS

### 1. CaseSecurity Update Events  
Used when sending updates to:
- CaseSecurity  
- CaseType  
- CaseCategory  
- Case status  

### 2. DocumentSecurity Update Events  
Used when sending:
- DocumentSecurity updates  
- New documents  
- Modified documents  

### 3. Case Index Update Events  
Used when sending:
- Party updates  
- Attorney updates  
- Judge updates  
- Style changes  

### 4. Event/Filing Updates  
Used when sending:
- New filings  
- Updated filings  
- Service events  

### 5. Hearing Updates (Only for CMS → re:Search)  
Triggered when sending:
- Hearing date/time  
- Hearing outcomes  
- Judge assigned  

## EventType Mistakes CMS Vendors Commonly Make

- Updating security in the wrong event  
- Updating case type in a document event  
- Sending a mixture of document & case data in the same event  
- Omitting required event-type attributes  
- Reusing stale event types  

Correct EventType usage is critical for re:Search to correctly update case visibility, document security, and index information.
