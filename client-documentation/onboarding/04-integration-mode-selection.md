# Integration Mode Selection – Client Onboarding

This guide helps courts and CMS vendors determine the correct re:Search integration mode.  
Selecting the right mode ensures a stable, maintainable, and scalable integration.

---

## Available Integration Modes

| Mode | Best For | Status |
|------|----------|--------|
| **Batch Mode** | EJ or 3rd-party CMS with export capabilities | Standard |
| **ECF Mode** | Legacy vendors already integrated with EFM | Legacy |
| **Non-Integrated** | Courts without CMS integration capability | Limited |

Detailed overviews:

- Batch Mode → `../integration-modes/batch-mode-overview.md`  
- ECF Mode → `../integration-modes/ecf-mode-overview.md`  
- Non-Integrated → `../integration-modes/non-integrated-mode-overview.md`  

---

## Decision Factors

### 1. CMS Capabilities
- Can the CMS generate scheduled exports (JSON/JSONL)?  
  → **Choose Batch Mode**

- Does the CMS already support ECF through an EFSP/EFM?  
  → **Choose ECF Mode**

- Does the CMS have *no integration layer at all*?  
  → **Choose Non-Integrated**

---

## Decision Tree (Short Form)

1. **Can you generate JSON snapshots?**  
   ✔ Yes → Batch  
   ✖ No → Continue

2. **Do you already operate through EFM (ECF 4.x)?**  
   ✔ Yes → ECF  
   ✖ No → Continue

3. **Do you have no integration capability at all?**  
   ✔ Yes → Non-Integrated  

---

## Recommended Default

**Tyler BIS recommends Batch Mode for all new markets and migrations.**

Batch offers:
- Lower support cost  
- Better error isolation  
- No dependency on EFM  
- High scalability  
- Better self-diagnostics  

---

For full mode descriptions, see:  
➡️ `../integration-modes/README.md`
