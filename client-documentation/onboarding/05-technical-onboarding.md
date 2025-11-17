# Technical Onboarding – Client Onboarding

This section outlines the technical steps required to begin implementation of a re:Search integration.

---

## 1. Kickoff Requirements

### Required Attendees
- CMS Lead Developer  
- CMS Architect  
- Court SME  
- Tyler BIS Support Analyst  
- Tyler TPM (Beltran)  

### Required Documents
- Integration Modes Overview  
- CMS architecture diagram  
- Available transport capabilities  
- API readiness (for ECF)  
- JSON schema readiness (for Batch)

---

## 2. Network & Security Setup

### Batch Mode
- Configure S3 or SFTP connectivity  
- Configure secure upload processes  
- Implement IAM keys or SSH authentication  

### ECF Mode
- Import Tyler client certificate  
- Validate outbound HTTPS + mutual TLS  
- Confirm WSDL compatibility  

---

## 3. Message & Schema Setup

### Batch
- Implement JSON Lines generation  
- Implement manifest.json creation  
- Validate schemas before upload  

### ECF
- Implement the following APIs:  
  - RecordFiling  
  - NotifyDocketingComplete  
  - NotifyCaseEvent  
  - GetCase  
  - GetDocument  
- Validate all XML against ECF 4.x schemas  
- Apply Tyler extensions (if required)

---

## 4. Logging & Monitoring

You must configure logging for:

- SOAP inbound & outbound messages  
- S3/SFTP transfer logs  
- Integration failures  
- Timeouts and retries  
- Security & authorization failures  

---

## 5. Development Expectations

- Implement full case payloads (no partial updates)  
- Implement security fields exactly  
- Ensure stable CaseTrackingID and CMSID  
- Ensure document retrieval works reliably  
- Ensure timestamps are ISO 8601 UTC  

---

## Next Steps

Once development begins:  
➡️ Proceed to **Testing & Certification**  
`./06-testing-and-certification.md`
