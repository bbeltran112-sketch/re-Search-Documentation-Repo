# Testing & Certification – Client Onboarding

This section defines how CMS vendors validate and certify their integration with re:Search.

---

## 1. Test Scenarios

You must complete the following scenarios:

### Case Creation & Updates
- Case initiation  
- Case update via NotifyCaseEvent  
- Case reassignment  
- Party changes  
- Security changes  

### Filings
- RecordFiling workflow  
- NotifyDocketingComplete  
- Document associations  
- Multiple filings in a case  

### Document Retrieval
- Full document IDs  
- GetDocument success  
- Security enforcement  

---

## 2. Batch Mode Tests
- Upload valid manifest + files  
- Validate schema failures  
- Validate missing file errors  
- Validate ingestion timestamp  
- Confirm case visibility in UI  

---

## 3. ECF Mode Tests
- All SOAP messages validated  
- NotifyCaseEvent covers all event types  
- GetCase returns full case payload  
- GetDocument returns real binaries  

---

## 4. Error Testing
You must intentionally trigger:

- Bad schema  
- Unsupported data types  
- Invalid CMSIDs  
- Invalid case identifiers  
- Missing security fields  

This ensures robust integration.

---

## 5. Certification Sign-Off

To complete certification:

1. All tests must pass  
2. All required fields must be implemented  
3. All event types must map correctly  
4. All documents must retrieve successfully  

Once complete, Tyler BIS will approve go-live readiness.

---

Next step:  
➡️ **Go-Live Readiness**  
`./07-go-live-readiness.md`
