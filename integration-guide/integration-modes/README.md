# Integration Modes

> **Breadcrumb:** [Home](../../README.md) > [Integration Guide](../README.md) > Integration Modes

Choose the integration mode that fits your technical capabilities and court requirements.

## Decision Tree

**Answer these questions:**

1. **Is your CMS already integrated with EFM?**
   - Yes → [ECF Mode](./ecf-mode.md)
   - No → Continue to question 2

2. **Do you use Enterprise Justice (EJ)?**
   - Yes → [CIP Mode](./cip-mode.md)
   - No → Continue to question 3

3. **Can your CMS expose SOAP web services?**
   - Yes → [ECF Mode](./ecf-mode.md)
   - No → Continue to question 4

4. **Can your CMS generate scheduled XML exports?**
   - Yes → [Batch Mode](./batch-mode.md)
   - No → [Non-Integrated Mode](./non-integrated-mode.md)

---

## Mode Comparison

| Feature | ECF Mode | CIP Mode | Batch Mode | Non-Integrated |
|---------|----------|----------|------------|----------------|
| **Real-time updates** | ✅ | ✅ | ❌ | ❌ |
| **CMS development required** | High | Medium | Low | None |
| **Technical complexity** | High | Medium | Low | None |
| **Typical timeline** | 3-6 months | 4-8 months | 2-4 months | 1-2 months |
| **Best for** | EFM-integrated courts | EJ courts | Limited API capability | Very small courts |

---

## Detailed Mode Guides

### [ECF Mode →](./ecf-mode.md)
Full integration with EFM using SOAP APIs
- **Prerequisites:** CMS integrated with EFM, SOAP capability
- **APIs Required:** NotifyCaseEvent, GetCase, GetDocument
- **Effort:** 3-6 months development

### [CIP Mode →](./cip-mode.md)
Middleware-based integration for Enterprise Justice
- **Prerequisites:** Enterprise Justice deployment, CIP middleware
- **APIs Required:** CIP handles web services
- **Effort:** 4-8 months (includes CIP configuration)

### [Batch Mode →](./batch-mode.md)
Scheduled XML file exports
- **Prerequisites:** XML generation capability, file transfer access
- **APIs Required:** None
- **Effort:** 2-4 months development

### [Non-Integrated Mode →](./non-integrated-mode.md)
Manual portal entry by court staff
- **Prerequisites:** User training only
- **APIs Required:** None
- **Effort:** 1-2 months setup and training

---

**Need help choosing?** Contact your Tyler Technical Project Manager (TPM)