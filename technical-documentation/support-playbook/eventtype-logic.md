# 6. EventType Logic

The NotifyCaseEvent API call includes an EventType field that determines how re:SearchTX interprets and applies the accompanying data.

re:SearchTX updates only the elements associated with the supplied event type. If the wrong event type is used, the system will not process the intended updates even if the correct values appear in the GetCaseResponse.

Incorrect EventType usage is the most common root cause of padlock, visibility, and indexing issues.

## 6.1 EventType Summary Table

| EventType             | Updates                               | Does Not Update                        |
|-----------------------|----------------------------------------|-----------------------------------------|
| CaseSecurity          | Case-level visibility                  | DocumentSecurity                        |
| DocumentSecurity      | Document-level visibility               | CaseSecurity                            |
| CaseFiling            | New case creation, initial metadata     | Existing case security values           |
| CaseUpdate            | General metadata, parties, style changes| CaseSecurity, DocumentSecurity          |
| DocumentFiling        | Document metadata and additions         | CaseSecurity                            |

## 6.2 Example: Updating Case Security

If the CMS updates CaseSecurity but sends: eventType = CaseUpdate

then:
- The system will ignore the CaseSecurity change.
- Padlocks will not update.
- Public visibility will remain unchanged.

To update case-level security correctly: eventType = CaseSecurity


## 6.3 Example: Updating DocumentSecurity

If a document is sealed or unsealed, the CMS must send: eventType = DocumentSecurity

Using DocumentFiling, CaseUpdate, or CaseFiling will not update document visibility.

## 6.4 Impact on Troubleshooting

When padlocks do not clear or cases remain public after being marked confidential, EventType errors are usually the cause.

To validate:
- Confirm the EventType sent by the CMS.
- Compare it against the intended change.
- Check if GetCaseResponse contains updated CaseSecurity or DocumentSecurity values.

## [Placeholder] EventType Flow Diagram



