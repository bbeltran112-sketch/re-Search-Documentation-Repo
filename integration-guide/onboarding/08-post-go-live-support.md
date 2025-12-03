# Post-Go-Live Support – Client Onboarding

**Navigation:**  
[Home](../../../README.md) › [Client Documentation](../README.md) › [Client Onboarding](./README.md) › Post-Go-Live Support

This page describes what happens **after** a court or CMS vendor goes live in re:Search.  
It outlines the short-term stabilization period, long-term expectations, and how BIS monitors and supports new integrations.

---

# 🎯 Purpose

The goal of post-go-live support is to:

- Ensure the new integration is stable  
- Quickly address any court or vendor-reported issues  
- Monitor data quality, event delivery, and visibility rules  
- Transition the integration into standard operational support (BIS Support Team)  

This period typically lasts **1–3 weeks** depending on the court size and integration mode.

---

# 🧭 What Happens After Go-Live

## 1. **Initial Stabilization Period**

BIS monitors:

- Batch ingestion results (Batch Mode)  
- Event throughput, NCE/NDC flow (ECF Mode)  
- GetCase volume and error rates  
- Case visibility mismatches  
- Missing filings/documents  
- Unexpected data spikes  

Common issues resolved in this window include:

- Incorrect case security visibility  
- Delayed or missing batches  
- Incorrect EventTypes (ECF Mode)  
- Party or attorney mismatches  
- Incorrect document metadata  

---

## 2. **Vendor/Court Responsibilities**

During the stabilization period, vendors and courts should:

- Continue monitoring logs and system health  
- Respond promptly to BIS questions or data clarification requests  
- Correct issues in CMS as needed  
- Provide updated JSONL exports (Batch) when necessary  
- Resend proper NotifyCaseEvent messages (ECF Mode) if errors are found  

---

## 3. **BIS Responsibilities**

BIS performs:

- Daily checks on ingestion, indexing, and UI visibility  
- Security verification (JCIT / market rules)  
- Review of any reported discrepancies  
- Coordination with CMS vendor, court SME, and Tyler Support  
- Documentation of issues discovered and corrective actions taken  
- Sign-off for transition to steady-state support  

---

# 🛠 Monitoring Activities

## Batch Mode  
BIS reviews:

- Manifest validation  
- Missing files or mismatched record counts  
- Failed or partial batch runs  
- Schema violations  
- Indexing lag  
- Case/doc counts vs expectations  

## ECF Mode  
BIS reviews:

- NotifyCaseEvent volume & event-type accuracy  
- NotifyDocketingComplete behavior  
- GetCase failures  
- Mismatched security settings  
- Docket/filing visibility timing  
- Document retrieval success  

---

# 📝 Issue Reporting & Routing

During this period, all issues should go to:

- **Tyler BIS Support** (primary)  
- **CMS Vendor Lead** (secondary)  
- **Court SME** (for case-specific context)

Issues may also be entered into:

- Tyler CRM  
- Vendor ticketing systems  
- Internal engineering Jira (as needed, created by BIS)

---

# 🔄 Transition to Long-Term Support

A deployment is considered stable when:

- No ingestion failures for 7 consecutive days  
- No visibility/security mismatches observed  
- Event flow and GetCase patterns stabilize  
- No critical issues open  
- Vendor and court confirm all expected behavior  

Then the integration is transitioned to **standard BIS Support**.

---

# 🔗 Related Documentation

**Integration**  
- [Integration Modes Overview](../integration-modes/README.md)  
- [Batch Mode Overview](../integration-modes/batch-mode-overview.md)  
- [ECF Mode Overview](../integration-modes/ecf-mode-overview.md)

**Onboarding**  
- [Client Onboarding Home](./README.md)  
- [Go-Live Readiness](./07-go-live-readiness.md)  
- [Go-Live Cutover Checklist](./08-go-live-cutover-checklist.md)

**Support**  
- [Support Playbook](../../technical-documentation/support-playbook/README.md)  
- [Troubleshooting Guide](../../technical-documentation/support-playbook/troubleshooting.md)

---

# ➡️ Next Steps

Proceed to:

**➡ [Transition to Steady-State Support](./09-steady-state-support.md)**  
(You can rename or merge this if preferred.)

---

# ⬅ Back to

**[Client Onboarding Home](./README.md)**  
