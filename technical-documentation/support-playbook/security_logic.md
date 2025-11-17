# Security Logic

**Navigation:**  
[Home](../../README.md) › [Technical Documentation](../README.md) › [Support Playbook](./index.md) › Security Logic

Security values drive case/document visibility in re:SearchTX.

---

## Purpose

Explain how CaseSecurity and DocumentSecurity control access to content across JCIT roles.

---

## CaseSecurity Behavior

| Value | Behavior |
|-------|----------|
| PublicFilingPublicView | Fully visible to Role 5 |
| PrivateView | Hidden from public; may show padlocks |
| Sealed | Hidden entirely except to authorized roles |

---

## DocumentSecurity Behavior

document security governs **individual filings**.

Padlock behavior rules:

- Case-level padlock driven by CaseSecurity  
- Document-level padlock driven by DocumentSecurity  
- Document padlocks do *not* affect case padlock visibility  

---

## Related Topics

- [EventType Logic](./eventtype-logic.md)
- [Troubleshooting](./troubleshooting.md)
- [Full XML Library](../xml-library/xml-library.md)
