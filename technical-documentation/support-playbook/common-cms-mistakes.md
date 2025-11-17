# Common CMS Mistakes

**Navigation:**  
[Home](../../README.md) › [Technical Documentation](../README.md) › [Support Playbook](./index.md) › Common CMS Mistakes

These are the most frequent causes of re:Search visibility or indexing problems.

---

## Common Issues

- Incorrect EventType values  
- Missing or incorrect CaseSecurity  
- Missing DocumentSecurity  
- Party/attorney mismatches  
- Incorrect case hierarchy  
- Missing unique identifiers  
- Reassigned fields not populated  

---

### Case Type Mismatch (JCIT Standards Not Followed)

When the CMS sends a case type value that does not exactly match the JCIT standard list, re:Search cannot map the case to any public-access rule. This results in the Public profile displaying a padlock even when caseSecurity and documentSecurity are correct.

For full details, root cause analysis, and the JCIT-approved case type list, see:

[Case Type Mismatch – Troubleshooting Guide](../troubleshooting/case-type-mismatch.md)
---

## Related Topics

- [Troubleshooting](./troubleshooting.md)
- [Security Logic](./security-logic.md)
- [Full XML Library](../xml-library/xml-library.md)
