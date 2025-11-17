# Post–Go-Live Support – Client Onboarding

This page describes how CMS vendors and courts maintain a healthy integration after production launch.

---

## Support Responsibilities

### CMS Vendor
- Maintain data accuracy  
- Ensure GetCase and GetDocument uptime  
- Maintain security rules  
- Provide logs for troubleshooting  
- Participate in issue triage  

### Court
- Validate filings  
- Validate case visibility  
- Report discrepancies  

### Tyler BIS
- Monitor ingestion health  
- Provide support for outages  
- Assist with integration anomalies  
- Maintain platform operations  

---

## Monitoring Expectations

### Batch Mode
- Monitor nightly uploads  
- Confirm manifest correctness  
- Review ingestion timestamps  

### ECF Mode
- Monitor SOAP failures  
- Track NotifyCaseEvent volume  
- Validate document retrieval  

---

## Issue Resolution Process

1. Vendor attempts to reproduce  
2. Vendor provides logs + XML/JSON  
3. BIS analyzes failure  
4. Fix applied (vendor or Tyler)  
5. Issue validated in Stage  
6. Fix deployed to Production  

---

## Contacts

- **Tyler BIS – Primary**: Bryce Beltran  
- **Tyler BIS – Support**: Assigned Analyst  
- **Vendor Support Lead**: (Provided during onboarding)

---

This completes the client onboarding documentation.
