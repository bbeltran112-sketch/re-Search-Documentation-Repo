# Go-Live Readiness – Client Onboarding

**Navigation:**  
[Home](../../../README.md) › [Client Documentation](../README.md) › [Client Onboarding](./README.md) › Go-Live Readiness

This page outlines the final steps required before a CMS vendor or court is approved for **production deployment** in re:Search.

---

# 🎯 Purpose

Go-live readiness ensures that:

- All technical dependencies are in place  
- All data is validated and functioning correctly  
- Security rules match JCIT or market-specific standards  
- Stakeholders are aligned on launch timing and expectations  
- Production access and credentials are properly configured  

This is the last checkpoint before a court becomes visible in **re:Search production**.

---

# ✅ Go-Live Readiness Checklist

The following must be completed **before** BIS schedules a production cutover.

---

## 1. **Integration Certification Completed**
Vendor must have passed full certification:

- Technical validation  
- Functional workflow testing  
- Case and document visibility review  
- SME verification  
- BIS sign-off  

If certification has not been completed, stop here.

---

## 2. **Production Environment Preparation**

### Confirm:
- Production S3/SFTP credentials issued (Batch)  
- Production SOAP endpoints + mTLS certs configured (ECF)  
- Production firewall rules opened  
- Production user accounts created (if applicable)  

### Environment Documentation  
➡ **[Environment Access →](./02-environment-access.md)**

---

## 3. **Data Review & Quality Checks**

Before launch, BIS reviews:

- Case counts align with expected totals  
- No schema validation failures  
- No missing identifiers (Case IDs, Filing IDs, location codes)  
- No invalid date formats  
- No broken document references  
- No mismatches between CMS security and re:Search security  

**Batch Mode Add-On Requirements:**

- Manifest integrity validated  
- JSONL record counts verified  
- No missing or duplicate batches  

---

## 4. **Security & Visibility Verification**

Courts must review:

- Public case availability  
- Confidential restrictions  
- Sealed cases and sealed documents  
- Attorney/party visibility  
- Judge/location assignments  

BIS verifies that UI visibility matches JCIT or the market-specific security model.

---

## 5. **Operational Readiness**

Vendor and court must confirm:

- A support contact is available during cutover  
- Logging and monitoring are enabled  
- Error alerts are configured (internal or vendor-side)  
- Internal court SMEs are ready for post-launch validation  

---

## 6. **Cutover Planning**

BIS will coordinate:

- Production cutover date  
- Required blackout or change freeze windows  
- Any final data refresh or initial batch load (Batch mode)  
- Timing of first NotifyCaseEvent/NotifyDocketingComplete traffic (ECF mode)  

A short go-live meeting may be scheduled with:

- CMS vendor  
- Court representatives  
- Tyler BIS  
- Product support (as needed)

---

## 7. **Post-Go-Live Monitoring**

For 1–3 weeks after production launch, BIS monitors:

- Batch ingestion health (Batch mode)  
- Event activity volume (ECF mode)  
- Cases failing visibility/security checks  
- Missing or malformed filings  
- Any court-reported discrepancies  

This period ensures the deployment stabilizes successfully.

---

# 🧾 Required Deliverables Before Go-Live

| Deliverable | Required From | Notes |
|------------|---------------|-------|
| Final data validation | CMS Vendor + Court | Must align with expected totals |
| Security model confirmation | Court | Cases must display correctly |
| Production credentials | Tyler BIS | Delivered securely |
| Cutover checklist | Vendor | Ensures readiness for launch |
| SME sign-off | Court | Confirms UI accuracy |
| BIS approval | Tyler BIS | Final go-live authorization |

---

# 🔗 Related Documentation

**Integration**  
- [Integration Modes Overview](../integration-modes/README.md)  
- [Batch Mode Overview](../integration-modes/batch-mode-overview.md)  
- [ECF Mode Overview](../integration-modes/ecf-mode-overview.md)  
- [Non-Integrated Mode Overview](../integration-modes/non-integrated-mode-overview.md)

**Onboarding**  
- [Client Onboarding Home](./README.md)  
- [Technical Prerequisites](./01-technical-prerequisites.md)  
- [Environment Access](./02-environment-access.md)  
- [Testing & Certification](./06-testing-and-certification.md)

---

# ➡️ Next Steps

Once Go-Live Readiness is complete, proceed to:

**➡ [Go-Live Cutover Checklist](./08-go-live-cutover-checklist.md)**

This final checklist ensures the deployment is executed smoothly and that both the vendor and court are aligned on the launch plan.

---

# ⬅ Back to

**[Client Onboarding Home](./README.md)**  
