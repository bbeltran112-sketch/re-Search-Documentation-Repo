# CMS Preparation Checklist – Client Onboarding

This checklist helps CMS vendors prepare their system, architecture, and operational processes before beginning re:Search integration.  
Completing this ensures a smooth onboarding and minimizes blockers during testing.

---

## System Readiness

### Case & Filing Data
- CMS can generate **complete case snapshots** including:
  - Case header  
  - Parties & attorneys  
  - Filings & docket entries  
  - Document metadata  
  - Case and document security  
- CMS maintains **stable case identifiers** (CaseTrackingID).
- CMS maintains **stable document identifiers** (CMSID).

### Document Repository
- Documents are accessible for retrieval (via URL or GetDocument).  
- All documents have unique CMSIDs.  
- Document security can be expressed using exact enumerations.

---

## Integration Mode Readiness

### If using **Batch Mode**
- Ability to generate JSON/JSONL exports.  
- Ability to generate a valid manifest.json.  
- Ability to deliver files to **S3** or **SFTP**.  
- Ability to produce full snapshots on a schedule.

### If using **ECF Mode**
- SOAP client supports mTLS.  
- CMS can process RecordFiling and NotifyDocketingComplete.  
- CMS can generate NotifyCaseEvent messages.  
- CMS can return full case metadata through GetCase.  
- CMS can return binaries through GetDocument.

---

## Security & Network Readiness

- Outbound HTTPS allowed to Tyler endpoints.  
- For Batch: outbound S3/SFTP allowed.  
- For ECF: mTLS enabled with Tyler-provided certificate.  
- Ability to rotate keys/certificates when required.  
- CMS environment supports logging for all integration calls.

---

## Operational Readiness

- Logging / audit trail configured (debug, info, warn, error).  
- Monitoring for failed uploads, SOAP faults, or invalid payloads.  
- Support team identified for:
  - Integration failures  
  - Document retrieval issues  
  - Security mismatches  
  - Case metadata inconsistencies  

---

## Documentation & Review

Please confirm the vendor team has reviewed:

- Integration Modes → `../integration-modes/README.md`
- Batch Mode Overview → `../integration-modes/batch-mode-overview.md`
- ECF Mode Overview → `../integration-modes/ecf-mode-overview.md`

---

## Final Confirmation

Once all checklist items are complete, notify Tyler BIS to begin:

➡️ **Technical Onboarding**  
`./05-technical-onboarding.md`
