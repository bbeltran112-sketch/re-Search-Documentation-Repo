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
- JCIT Case Type Mismatch (CMS sending non-standard case type)

---

## High-Level Issues

### JCIT Case Type Mismatch  
When a CMS sends a case type value that does not match any JCIT-approved case type, re:Search cannot map the case to a public-access rule. This often results in incorrect padlock behavior even when caseSecurity and documentSecurity values are set correctly.

See the full troubleshooting article:  
[Case Type Mismatch – JCIT Standard Enforcement Issue](./case-type-mismatch.md)

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
