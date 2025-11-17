# EventType Logic

**Navigation:**  
[Home](../../README.md) › [Technical Documentation](../README.md) › [Support Playbook](./index.md) › EventType Logic

EventTypes determine which sections of GetCaseResponse re:Search processes.

---

## Purpose

Define how re:Search selectively processes XML based on the declared EventType.

---

## Rules

EventType → drives → XML evaluation.

Examples:

- CaseSecurity → evaluate CaseSecurity only  
- DocumentSecurity → evaluate document-level updates only  
- CaseFiling → evaluate filings section only  

Incorrect EventType = stale or incorrect visibility.

---

## Related Topics

- [Security Logic](./security-logic.md)
- [Troubleshooting](./troubleshooting.md)
- [Full XML Library](../xml-library/xml-library.md)
