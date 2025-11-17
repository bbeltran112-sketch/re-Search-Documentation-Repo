# API Reference – re:Search Integration

This section contains the complete reference for all re:Search integration APIs.  
It covers both event-driven ECF operations and document/case retrieval methods used during indexing.

All API pages follow a consistent structure:

- Purpose and overview  
- Direction of communication  
- Transport and contract details  
- Required headers and authentication  
- Validation and sequencing rules  
- Example request/response messages  
- Common errors  
- Related documentation  

This API reference supports the **re:Search ECF Integration Mode**, which follows **OASIS ECF 4.x** standards using SOAP/XML over HTTPS with **mutual TLS** (mTLS).

---

## Core APIs

| API | Direction | Description |
|-----|-----------|-------------|
| [RecordFiling](./recordfiling/README.md) | **EFM → CMS** | Sends accepted filings to CMS for docketing |
| [NotifyDocketingComplete](./notifydocketingcomplete/README.md) | **CMS → EFM** | Confirms docketing and returns CMS-assigned identifiers |
| [NotifyCaseEvent](./notifycaseevent/README.md) | **CMS → re:Search** | Sends case update notifications to trigger synchronization |
| [GetCase](./getcase/README.md) | **re:Search → CMS** | Retrieves complete case and filing metadata |
| [GetDocument](./getdocument/README.md) | **re:Search → CMS** | Retrieves document binaries for authorized requests |

---

## Common Artifacts

- [Common Headers & Authentication](./common-headers-and-auth.md)  
- [Error Codes](./error-codes.md)  

---

## Integration Mode Context

These APIs support the broader ECF Mode architecture:

- [Integration Modes Overview](../../client-documentation/integration-modes/README.md)  
- [ECF Mode Overview](../../client-documentation/integration-modes/ecf-mode-overview.md)  
- [Batch Mode Overview](../../client-documentation/integration-modes/batch-mode-overview.md)  

---

## Related Documentation

- [Batch Mode Guide](../integration-guides/batch-mode-guide.md)  
- [ECF Mode Guide](../integration-guides/ecf-mode-guide.md)  
- [System Behavior](../system-behavior/README.md)  
- [Runbooks](../runbooks/README.md)

**Back to:** [Technical Documentation](../README.md)
