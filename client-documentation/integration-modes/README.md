# Integration Modes Overview

The re:Search platform supports multiple integration modes to accommodate the diverse CMS environments and technical requirements across Tyler markets.  
These modes define how case, filing, party, and document data flows from a **Case Management System (CMS)** into **re:Search**.

As of 2025, Tyler’s strategy is to standardize all new integrations on the **Batch Mode** framework.

---

## Summary of Integration Modes

| Mode | Description | Primary Use Case | Status |
|------|-------------|------------------|--------|
| **Batch Mode** | Scheduled, file-based delivery of JSONL snapshots via S3/SFTP. | All new CMS integrations | Standard |
| **ECF Mode** | Event-driven communication using EFM SOAP APIs. | Legacy vendor integrations | Maintained |
| **CIP Mode (EJ)** | Direct EJ REST/XML communication. | Historic EJ implementations | Phasing out |
| **Non-Integrated** | No CMS integration; only e-filing activity. | Small or pilot courts | Limited |

---

## Batch Mode (Standard)

Batch Mode uses scheduled JSONL file delivery instead of APIs.  
It is the preferred integration method for all new markets and migrations.

**Learn more:**  
[Batch Mode Overview](./batch-mode-overview.md)

---

## ECF Mode

ECF Mode uses event-driven messages (`NotifyCaseEvent`, `RecordFiling`, etc.) between the CMS, EFM, and re:Search.

**Learn more:**  
[ECF Mode Overview](./ecf-mode-overview.md)

---

## CIP Mode (Enterprise Justice)

Used exclusively by EJ courts and now being retired in favor of Batch Mode.

**Learn more:**  
[CIP Mode Overview](./cip-mode-overview.md)

---

## Non-Integrated Mode

Used only where no automated integration exists and data flows solely from the e-filing platform.

**Learn more:**  
[Non-Integrated Mode Overview](./non-integrated-mode-overview.md)

---

## Integration Mode Selection Decision Tree

A simple decision guide for choosing the correct integration model.

[View the Decision Tree](./selection-decision-tree.md)

## Related Documentation

- [Batch Mode Overview](./batch-mode-overview.md)  
- [ECF Mode Overview](./ecf-mode-overview.md)  
- [CIP Mode Overview](./cip-mode-overview.md)  
- [Non-Integrated Mode Overview](./non-integrated-mode-overview.md)  
- [Integration Mode Selection Decision Tree](./selection-decision-tree.md)  

**Back to:** [Client Documentation](../README.md)
