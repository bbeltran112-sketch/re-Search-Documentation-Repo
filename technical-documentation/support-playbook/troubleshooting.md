# Troubleshooting Guide

**Navigation:**  
[Home](../../README.md) › [Technical Documentation](../README.md) › [Support Playbook](./index.md) › Troubleshooting

This guide provides step-by-step workflows to diagnose visibility issues, stale data, incorrect XML, and indexing discrepancies.

---

## Common Scenarios

- Case does not appear in UI  
- Incorrect padlock behavior  
- Missing filings or docket entries  
- Document download blocked  
- Security updates not reflected  
- EventType mismatch  

---

## General Troubleshooting Steps

1. Review the EventType  
2. Check CaseSecurity and DocumentSecurity  
3. Retrieve the most recent GetCaseResponse  
4. Validate XML structure  
5. Confirm CMS logic  
6. Trigger reindex if necessary  

---

## Related Topics

- [Common CMS Mistakes](./common-cms-mistakes.md)
- [EventType Logic](./eventtype-logic.md)
- [Security Logic](./security-logic.md)
- [Full XML Library](../xml-library/xml-library.md)
