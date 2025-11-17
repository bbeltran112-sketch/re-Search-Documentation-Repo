# 12. XML Examples (Updated)

The following examples reflect the actual structures used by re:Search, DAS, and CMS/EFM vendors as observed in current production and stage environments.

All examples follow the patterns used in your NotifyCaseEvent, GetCaseResponse, DocumentSecurity, CaseSecurity, and RecordDocketingCallback flows.

---

## 12.1 Correct NotifyCaseEvent — CaseSecurity Update

A correct CaseSecurity update **must** include:
- `<EventType>CaseSecurity</EventType>`
- `<Case>` block with updated `<CaseSecurity>`
- Correct `<CaseTrackingID>` matching the target case

```xml
<NotifyCaseEvent> <EventType>CaseSecurity</EventType> <Case> <CaseTrackingID>CV45614</CaseTrackingID> <CaseSecurity>PublicFilingPublicView</CaseSecurity> </Case> </NotifyCaseEvent>
```


## 12.2 Incorrect CaseSecurity Update — Wrong EventType

Even if CaseSecurity is correct, this update will be ignored:

```xml
<NotifyCaseEvent>
    <EventType>CaseUpdate</EventType>
    <Case>
        <CaseTrackingID>CV45614</CaseTrackingID>
        <CaseSecurity>PublicFilingPublicView</CaseSecurity>
    </Case>
</NotifyCaseEvent>
```
## 12.3 Correct NotifyCaseEvent — DocumentSecurity Update

A correct document-level security update must include:

- <EventType>DocumentSecurity</EventType>

- Document identifiers under <Document>

- Required fields for the updated document

```xml
<NotifyCaseEvent>
    <EventType>DocumentSecurity</EventType>
    <Case>
        <CaseTrackingID>CV45614</CaseTrackingID>
    </Case>
    <Document>
        <DocumentIdentification>
            <IdentificationID>DOC-00123</IdentificationID>
        </DocumentIdentification>
        <DocumentSecurity>Public</DocumentSecurity>
    </Document>
</NotifyCaseEvent>
```
## 12.4 Incorrect DocumentSecurity Update — Wrong EventType

This is a very common vendor error:

```xml
<NotifyCaseEvent>
    <EventType>DocumentFiling</EventType>
    <Case>
        <CaseTrackingID>CV45614</CaseTrackingID>
    </Case>
    <Document>
        <DocumentIdentification>
            <IdentificationID>DOC-00123</IdentificationID>
        </DocumentIdentification>
        <DocumentSecurity>Public</DocumentSecurity>
    </Document>
</NotifyCaseEvent>
```
re:Search will not update document-level security here.

## 12.5 Correct Case + Document Update (Two-Block Form)

Vendors sometimes send both security updates in the same message.
This is allowed as long as EventType matches the highest-level change.

```xml
<NotifyCaseEvent>
    <EventType>CaseSecurity</EventType>
    <Case>
        <CaseTrackingID>CV45614</CaseTrackingID>
        <CaseSecurity>Confidential</CaseSecurity>
    </Case>
    <Document>
        <DocumentIdentification>
            <IdentificationID>DOC-00123</IdentificationID>
        </DocumentIdentification>
        <DocumentSecurity>PrivateView</DocumentSecurity>
    </Document>
</NotifyCaseEvent>
```
Note: If you want both CaseSecurity and DocumentSecurity processed,
you must send CaseSecurity as the EventType — it is the higher-level event.

## 12.6 Correct GetCaseResponse (Integrated County)

Matches your recent examples:

```xml
<GetCaseResponse>
    <Case>
        <CaseTrackingID>CV45614</CaseTrackingID>
        <CaseCategory>Civil</CaseCategory>
        <CaseType>Contract</CaseType>
        <FiledDate>2025-11-15</FiledDate>
        <Location>Red River County</Location>
        <Style>ABC Contractors LLC vs. John Doe</Style>
        <CaseSecurity>PublicFilingPublicView</CaseSecurity>

        <Party>
            <PartyType>Plaintiff</PartyType>
            <FullName>ABC Contractors LLC</FullName>
        </Party>
        <Party>
            <PartyType>Defendant</PartyType>
            <FullName>John Doe</FullName>
        </Party>

        <Documents>
            <Document>
                <DocumentIdentification>
                    <IdentificationID>DOC-00123</IdentificationID>
                    <CMSID>127364</CMSID>
                </DocumentIdentification>
                <DocumentType>Petition</DocumentType>
                <DocumentSecurity>Public</DocumentSecurity>
                <FileDate>2025-11-15</FileDate>
            </Document>
        </Documents>
    </Case>
</GetCaseResponse>
```

## 12.7 Correct GetCaseResponse (Document Security Causes Hidden Case)
```xml
<GetCaseResponse>
    <Case>
        <CaseTrackingID>CV45614</CaseTrackingID>
        <CaseCategory>Civil</CaseCategory>
        <CaseType>Contract</CaseType>
        <FiledDate>2025-11-15</FiledDate>
        <Location>Red River County</Location>
        <Style>ABC Contractors LLC vs. John Doe</Style>
        <CaseSecurity>PublicFilingPublicView</CaseSecurity>

        <Documents>
            <Document>
                <DocumentIdentification>
                    <IdentificationID>DOC-00999</IdentificationID>
                </DocumentIdentification>
                <DocumentType>Exhibit</DocumentType>
                <DocumentSecurity>PrivateView</DocumentSecurity>
            </Document>
        </Documents>
    </Case>
</GetCaseResponse>

Result:
Even though case-level security is public, the public user will not see this case because at least one document is confidential.
```
## 12.8 Correct RecordDocketingCallbackMessage (from your examples)

```xml
<RecordDocketingCallbackMessage xmlns="urn:tyler:efm:services:RecordDocketingCallbackMessage">
    <MessageMetaData>
        <SendingMDECode>EFM</SendingMDECode>
        <ReceivingMDECode>CMS</ReceivingMDECode>
        <MessageType>RecordDocketingCallback</MessageType>
        <MessageID>efm12345-6789</MessageID>
        <Timestamp>2025-11-16T08:12:42Z</Timestamp>
    </MessageMetaData>

    <RecordDocketingCallback>
        <EnvelopeID>EE123456</EnvelopeID>
        <CaseTrackingID>CV45614</CaseTrackingID>
        <DocketingComplete>true</DocketingComplete>
    </RecordDocketingCallback>
</RecordDocketingCallbackMessage>
```
## 12.9 Minimal Valid NotifyCaseEvent (Non-Integrated County)
```xml
<NotifyCaseEvent>
    <EventType>CaseFiling</EventType>
    <Case>
        <CaseTrackingID>CV12345</CaseTrackingID>
        <CaseCategory>Civil</CaseCategory>
        <CaseType>Other Civil</CaseType>
        <FiledDate>2025-11-15</FiledDate>
        <Style>Protected vs Protected</Style>
        <CaseSecurity>PublicFilingPublicView</CaseSecurity>
    </Case>
</NotifyCaseEvent>
```
## 12.10 Incorrect CaseFiling (Missing Fields)

This is a realistic example of why cases fail to index:
```xml
<NotifyCaseEvent>
    <EventType>CaseFiling</EventType>
    <Case>
        <CaseTrackingID></CaseTrackingID>           <!-- missing -->
        <CaseCategory></CaseCategory>               <!-- missing -->
        <CaseType>InvalidType</CaseType>            <!-- not JCIT compliant -->
        <FiledDate></FiledDate>                     <!-- missing -->
        <Location></Location>                       <!-- missing -->
    </Case>
</NotifyCaseEvent>
```
Result:
Case will never index or appear in re:SearchTX.
