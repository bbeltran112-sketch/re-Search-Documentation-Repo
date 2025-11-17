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

Link:  
[XML Examples](../xml-library/xml-examples.md)
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
[Support Playbook](../support-playbook/README.md)

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


