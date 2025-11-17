# XML Library for re:SearchTX Integrations

This XML Library provides authoritative examples of the message formats exchanged between CMS vendors, EFSPs, the Electronic Filing Manager (EFM), and the re:SearchTX platform. These samples reflect real structures observed in production and stage environments and are intended to support message validation, troubleshooting, and vendor guidance.

This library includes:

- Correct and incorrect NotifyCaseEvent examples  
- CaseSecurity and DocumentSecurity update patterns  
- GetCaseResponse structures  
- Document metadata blocks  
- RecordDocketingCallbackMessage examples  
- Minimal and invalid message patterns  
- Integration mode differences  
- Common vendor errors

All examples follow the structures processed by the Document Access System (DAS) and re:Search's indexing logic.

---

# NotifyCaseEvent Examples

## Correct — CaseSecurity Update
<!-- Placeholder for CaseSecurity XML -->


## Incorrect — Wrong EventType for CaseSecurity
<!-- Placeholder for bad CaseSecurity XML -->

## Correct — DocumentSecurity Update
<!-- Placeholder for DocumentSecurity XML -->

## Incorrect — Wrong EventType for DocumentSecurity
<!-- Placeholder for bad DocumentSecurity XML -->

## Combined Case + Document Security Update
<!-- Placeholder for multi-block XML -->


---

# GetCaseResponse Examples

## Correct Integrated County Response
<!-- Placeholder for GetCaseResponse XML -->

## Case Hidden Because of Confidential Document
<!-- Placeholder for GetCaseResponse (private doc) XML -->


---

# RecordDocketingCallbackMessage
<!-- Placeholder for RecordDocketingCallbackMessage XML -->


---

# Minimal Valid Non-Integrated CaseFiling
<!-- Placeholder for Minimal CaseFiling XML -->


---

# Invalid CaseFiling Example
<!-- Placeholder for Invalid CaseFiling XML -->

---

---

# Related Documentation

For additional context, behavior rules, troubleshooting workflows, and API specifications, refer to:

## Support Playbook
A complete guide covering:
- JCIT roles
- Security logic
- EventType processing
- Common CMS mistakes
- Behavior examples
- Escalation guidelines
- Troubleshooting flowcharts

Link:  
[Support Playbook](../support-playbook/index.md)

---

## API Reference
Defines message behavior, required fields, schema expectations, and inline XML examples for each core API.

Links:
- [API Reference Overview](../api-reference/README.md)
- [NotifyCaseEvent](../api-reference/notifycaseevent/README.md)
- [NotifyDocketingComplete](../api-reference/notifydocketingcomplete/README.md)
- [RecordFiling](../api-reference/recordfiling/README.md)
- [GetCase](../api-reference/getcase/README.md)
- [GetDocument](../api-reference/getdocument/README.md)

---

## Troubleshooting Guides
Step-by-step diagnosis of visibility, indexing, CaseSecurity/DocumentSecurity issues, missing cases, and vendor message errors.

Link:  
[Troubleshooting Guide](../support-playbook/troubleshooting.md)

---


