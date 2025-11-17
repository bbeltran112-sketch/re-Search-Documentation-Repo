# Integration Modes Overview

**Navigation:**  
[Home](../../../README.md) › [Client Documentation](../README.md) › Integration Modes

This page provides a unified, high-level view of all supported re:Search integration modes: how they work, when to use them, and where to find their detailed documentation.

---

## Purpose

Integration modes define **how case, filing, and security data enter re:Search**.  
Selecting the correct mode ensures accurate indexing, proper visibility handling, and predictable synchronization behavior.

---

# 📁 Integration Modes

| Mode | Link | Summary |
|------|------|---------|
| **Batch Mode** | [Batch Overview →](./batch-mode-overview.md) | JSONL + S3 ingestion; bulk, scheduled publishing. |
| **ECF Mode** | [ECF Overview →](./ecf-mode-overview.md) | Real-time event-driven model using ECF 4.x SOAP. |
| **CIP Mode** | [CIP Overview →](./cip-mode-overview.md) | Legacy EJ → re:Search REST publishing. |
| **Non-Integrated Mode** | [Non-Integrated Overview →](./non-integrated-mode-overview.md) | Manual/assisted publishing via BIS tools. |

<br>

---

<br>

# 🧭 When to Use Each Mode (Quick Guide)

| Mode | When to Use It | Ideal Scenarios |
|------|----------------|-----------------|
| **Batch Mode** | CMS cannot support ECF/SOAP; court prefers scheduled or bulk updates; large historical ingestion required | Non-ECF courts; vendors needing low overhead or high-volume ingestion |
| **ECF Mode** | Court already integrates with EFM; real-time updates required; CMS can support NotifyCaseEvent + GetCase | Fully integrated courts needing accurate, immediate synchronization |
| **CIP Mode** | Court is on legacy EJ; transition to a modern CMS is pending; REST ingestion preferred | EJ courts in transition with minimal infrastructure changes |
| **Non-Integrated Mode** | No CMS integration path exists; very small or low-volume courts; manual or limited publishing acceptable | Small counties, limited resources, BIS-assisted publishing |

---

# 📚 Related Technical Areas

- [API Reference](../../technical-documentation/api-reference/README.md)  
- [Support Playbook](../../technical-documentation/support-playbook/index.md)  
- [XML Library](../../technical-documentation/xml-library/xml-library.md)

---

# ➡ Next Steps

- [Client Onboarding Guide →](../onboarding/README.md)
- [Client Documentation Home →](../README.md) 
- [Repository Home →](../../../README.md)
