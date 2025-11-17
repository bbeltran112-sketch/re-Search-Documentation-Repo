# Environment Access – Client Onboarding

This page explains how CMS vendors and courts access Tyler-hosted environments required for re:Search development, testing, and certification.

---

## Access Types

### 1. re:Search UI Access (Stage)
Used for validating case visibility, filings, and document access.

### 2. EFM Web Services Access (ECF Mode)
Required for RecordFiling, NotifyDocketingComplete, and NotifyCaseEvent.

### 3. S3 or SFTP Access (Batch Mode)
Required for File/Manifest delivery.

---

## Credentials & Provisioning

### Batch Mode  
Tyler provides:
- S3 Access Key + Secret  
- Bucket path  
- Court/CMS folder structure  
- IAM policy  
- SFTP username/password (if needed)

### ECF Mode  
Tyler provides:
- mTLS client certificate  
- Endpoint list  
- Court/county codes  
- WSDL and schemas  

---

## Network Requirements

- Firewall rules permitting HTTPS traffic to Tyler endpoints  
- Ability to call SOAP endpoints from CMS  
- Ability to upload to S3/SFTP  

---

## Testing Environments

| Environment | Purpose |
|-------------|---------|
| **Stage** | Full integration testing, certification, error debugging |
| **Production** | Live environment after sign-off |

---

## Related Pages

- Batch Mode Overview → `../integration-modes/batch-mode-overview.md`  
- ECF Mode Overview → `../integration-modes/ecf-mode-overview.md`  
