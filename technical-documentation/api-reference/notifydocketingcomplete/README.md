# NotifyDocketingComplete

**Navigation:**  
[Home](../../../README.md) › [Technical Documentation](../../README.md) › [API Reference](../README.md) › NotifyDocketingComplete

NotifyDocketingComplete confirms that a filing has been docketed by the CMS following a successful RecordFiling request.

---

## Purpose

NDC signals the completion of:

- CMS-side docketing  
- Filing acceptance  
- Metadata synchronization  
- Trigger for re:Search index update  

---

## Transport and Protocol

| Property | Value |
|---------|--------|
| Direction | CMS → re:Search |
| Protocol | SOAP 1.2 over HTTPS |
| Security | Mutual TLS |
| Standard | OASIS ECF |
| Message Type | NotifyDocketingCompleteMessage |

Authentication details:  
[Common Headers & Auth](../common-headers-and-auth.md)

---

## When This API Is Used

The CMS sends NDC:

- After accepting a filing from the EFM  
- After docket entry creation  
- When confirming filing metadata  

---

## Behavior Summary

re:Search:

- Validates the filing metadata  
- Calls GetCase for authoritative data  
- Updates indexes and UI  

---
## Processing Workflow
<!-- XML Placeholder: Insert sample GetDocumentResponse here -->

---

## XML Example
```xml
<RecordDocketingCallbackMessage
    xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:RecordDocketingCallbackMessage-4.0"
    xmlns:j="http://niem.gov/niem/domains/jxdm/4.0"
    xmlns:nc="http://niem.gov/niem/niem-core/2.0"
    xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CommonTypes-4.0"
    xmlns:s="http://niem.gov/niem/structures/2.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <!-- EFM filing ID(s) echoed from RecordFiling -->
    <nc:DocumentIdentification s:id="FilingID1">
        <nc:IdentificationID>001-f786e6d6-558c-448f-b249-067002d716a0</nc:IdentificationID>
    </nc:DocumentIdentification>
    <!-- Sender identity -->
    <ecf:SendingMDELocationID>
        <nc:IdentificationID>https://courtrecordmde.com</nc:IdentificationID>
    </ecf:SendingMDELocationID>
    <ecf:SendingMDEProfileCode>
        urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:WebServicesMessaging-2.0
    </ecf:SendingMDEProfileCode>
    <!-- Case header updates -->
    <civil:CivilCase xmlns:civil="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CivilCase-4.0">
        <nc:CaseTitleText>Troy Star vs Jane Smith</nc:CaseTitleText>
        <nc:CaseCategoryText>CV</nc:CaseCategoryText>
        <nc:CaseTrackingID>22222222</nc:CaseTrackingID>
        <nc:CaseDocketID>CV-2013-10000</nc:CaseDocketID>
        <j:CaseAugmentation>
            <j:CaseCourt>
                <nc:OrganizationIdentification>
                    <nc:IdentificationID>harris:dc</nc:IdentificationID>
                </nc:OrganizationIdentification>
            </j:CaseCourt>
            <j:CaseJudge>
                <j:JudicialOfficialBarMembership>
                    <j:JudicialOfficialBarIdentification>
                        <nc:IdentificationID>111</nc:IdentificationID>
                        <nc:IdentificationCategoryText>JUDGEID</nc:IdentificationCategoryText>
                    </j:JudicialOfficialBarIdentification>
                </j:JudicialOfficialBarMembership>
            </j:CaseJudge>
            <j:CaseLineageCase>
                <nc:CaseTrackingID>B4EE599A-C9F8-4E77-9937-65AE754A3494</nc:CaseTrackingID>
            </j:CaseLineageCase>
        </j:CaseAugmentation>
        <!-- Participant CMSID mapping (ATTORNEYID/CASEPARTYATTORNEYID + CMSID) -->
        <tyler:CaseAugmentation xmlns:tyler="urn:tyler:ecf:extensions:Common">
            <ecf:CaseOtherEntityAttorney>
                <nc:RoleOfPersonReference s:ref="Attorney1"/>
                <j:CaseOfficialRoleText>LEAD</j:CaseOfficialRoleText>
                <ecf:CaseRepresentedPartyReference s:ref="Party1"/>
            </ecf:CaseOtherEntityAttorney>
            <ecf:CaseParticipant>
                <ecf:EntityPerson s:id="Attorney1">
                    <nc:PersonOtherIdentification>
                        <nc:IdentificationID>037BAEC1-F6E8-43ED-A102-B8C5E76748EB</nc:IdentificationID>
                        <nc:IdentificationCategoryText>ATTORNEYID</nc:IdentificationCategoryText>
                    </nc:PersonOtherIdentification>
                    <nc:PersonOtherIdentification>
                        <nc:IdentificationID>22222</nc:IdentificationID>
                        <nc:IdentificationCategoryText>CMSID</nc:IdentificationCategoryText>
                    </nc:PersonOtherIdentification>
                </ecf:EntityPerson>
                <ecf:CaseParticipantRoleCode>ATTY</ecf:CaseParticipantRoleCode>
            </ecf:CaseParticipant>
            <ecf:CaseParticipant>
                <ecf:EntityPerson s:id="Party1">
                    <nc:PersonOtherIdentification>
                        <nc:IdentificationID>037BAEC1-F6E8-43ED-A102-B8C5E76748EB</nc:IdentificationID>
                        <nc:IdentificationCategoryText>PARTYID</nc:IdentificationCategoryText>
                    </nc:PersonOtherIdentification>
                    <nc:PersonOtherIdentification>
                        <nc:IdentificationID>333333</nc:IdentificationID>
                        <nc:IdentificationCategoryText>CMSID</nc:IdentificationCategoryText>
                    </nc:PersonOtherIdentification>
                </ecf:EntityPerson>
                <ecf:CaseParticipantRoleCode>P</ecf:CaseParticipantRoleCode>
            </ecf:CaseParticipant>
            <!-- Example org participant -->
            <ecf:CaseParticipant>
                <ecf:EntityOrganization s:id="Party3">
                    <tyler:OrganizationOtherIdentification>
                        <tyler:Identification>
                            <nc:IdentificationID>2f9be3aa-d424-4a64-bff2-af5d32236bca</nc:IdentificationID>
                            <nc:IdentificationCategoryText>PARTYID</nc:IdentificationCategoryText>
                        </tyler:Identification>
                        <tyler:Identification>
                            <nc:IdentificationID>223</nc:IdentificationID>
                            <nc:IdentificationCategoryText>CMSID</nc:IdentificationCategoryText>
                        </tyler:Identification>
                    </tyler:OrganizationOtherIdentification>
                </ecf:EntityOrganization>
                <ecf:CaseParticipantRoleCode>D</ecf:CaseParticipantRoleCode>
            </ecf:CaseParticipant>
            <tyler:CaseTypeText>KDM</tyler:CaseTypeText>
        </tyler:CaseAugmentation>
        <!-- Optional civil augmentation -->
        <ecf:CauseOfActionCode/>
        <civil:ClassActionIndicator>false</civil:ClassActionIndicator>
        <civil:JuryDemandIndicator>false</civil:JuryDemandIndicator>
        <civil:ReliefTypeCode/>
    </civil:CivilCase>
    <!-- Overall filing status -->
    <ecf:FilingStatus>
        <nc:StatusDescriptionText>Accepted without modification.</nc:StatusDescriptionText>
        <ecf:FilingStatusCode>accepted</ecf:FilingStatusCode>
    </ecf:FilingStatus>
    <!-- Reviewed docs: return CMSIDs for filings + documents -->
    <tyler:ReviewedLeadDocument xmlns:tyler="urn:tyler:ecf:extensions:Common">
        <ecf:DocumentMetadata>
            <j:RegisterActionDescriptionText/>
            <ecf:FilingAttorneyID/>
            <ecf:FilingPartyID/>
        </ecf:DocumentMetadata>
        <ecf:DocumentRendition>
            <ecf:DocumentRenditionMetadata>
                <ecf:DocumentAttachment s:id="Attachment1"/>
            </ecf:DocumentRenditionMetadata>
        </ecf:DocumentRendition>
        <!-- Maps doc attachment to CMSID -->
        <tyler:DocumentAttachmentIdentification>
            <tyler:DocumentAttachmentReference s:ref="Attachment1"/>
            <tyler:DocumentID>AFB14CC5-3DC8-48A4-9448-00F26D7CE2C2</tyler:DocumentID>
            <tyler:CMSID>1121212</tyler:CMSID>
        </tyler:DocumentAttachmentIdentification>
        <!-- Maps filing to CMSID -->
        <tyler:FilingIdentification>
            <tyler:FilingIdentificationReference s:ref="FilingID1"/>
            <tyler:CMSID>34333</tyler:CMSID>
        </tyler:FilingIdentification>
    </tyler:ReviewedLeadDocument>
</RecordDocketingCallbackMessage>
```
---

## Example XML Files

| File | Description |
|------|-------------|
| [message-receipt-accepted.xml](./examples/message-receipt-accepted.xml) | Receipt confirmation |
| [recorddocketingcallback-annotated.xml](./examples/recorddocketingcallback-annotated.xml) | Annotated callback |

---

## Related API Pages

- [RecordFiling](../recordfiling/README.md)  
- [NotifyCaseEvent](../notifycaseevent/README.md)  
- [GetCase](../getcase/README.md)  
- [GetDocument](../getdocument/README.md)  

---

## Related Topics

- [System Behavior Examples](../../support-playbook/system-behavior-examples.md)
- [Troubleshooting Guide](../../support-playbook/troubleshooting.md)
- [Full XML Library](../../xml-library/xml-library.md)






















# NotifyDocketingComplete – re:Search Integration API

Version: 3.4.x  
Audience: Technical  
Owner: BIS – Bryce Beltran  
Last Updated: 2025-11-12  

`NotifyDocketingComplete` is the message the **CMS sends to the EFM** after a filing has been successfully docketed.  
This message confirms acceptance of the filing, provides CMS-assigned identifiers, and signals that the CMS has completed its internal processing.

It is a critical part of the ECF workflow because it ensures that:

- The EFM can finalize the filing,
- The EFSP can notify the filer,
- re:Search can synchronize the case using a subsequent `GetCase` call.

---

## Purpose

NotifyDocketingComplete allows the CMS to:

- Confirm the court has accepted and docketed a filing  
- Return CMS-assigned document IDs, filing IDs, and docket entry identifiers  
- Indicate docketing details including date, time, and clerk reviewer  
- Complete the ECF filing lifecycle initiated by RecordFiling  

This message also indirectly triggers re:Search synchronization because the EFM sends an ECF event to re:Search after receiving this confirmation.

---

## Transport and Protocol

| Property | Value |
|----------|-------|
| Direction | **CMS → EFM** |
| Protocol | SOAP 1.2 over HTTPS |
| Security | Mutual TLS (mTLS) |
| Schema | OASIS ECF 4.x – `NotifyDocketingCompleteMessage` |
| Triggered By | Successful docketing of a filing |

Headers follow ECF standard SOAP envelope extensions.  
See: `../common-headers-and-auth.md`

---

## When NotifyDocketingComplete Occurs

NotifyDocketingComplete is always sent **after a RecordFiling request** has been processed by the CMS and the filing is officially docketed.  
Common triggers:

- Clerk approves the filing  
- Automatic docketing occurs (auto-accept sites)  
- CMS processes envelope and attaches documents to the case  
- CMS generates filing and document IDs  

If a CMS does not send this message, the EFM cannot complete the transaction and re:Search may not receive accurate filing metadata.

---

## High-Level Workflow

Diagram here

### Steps

1. **EFM Sends RecordFiling**  
   CMS receives filing payload and extracts documents, metadata, parties, and associations.

2. **CMS Dockets Filing**  
   The CMS creates:
   - CMS Filing ID  
   - Docket Entry ID  
   - CMS Document IDs for each attachment  

3. **CMS Sends NotifyDocketingComplete**  
   Returned data includes:
   - Envelope ID  
   - Filing IDs and document IDs  
   - Docket entry information  
   - Timestamp and reviewed‐by data  

4. **EFM Acknowledges and Finalizes**  
   EFSP receives confirmation and updates its transaction history.

5. **EFM Sends Event to re:Search**  
   re:Search calls `GetCase` to retrieve the updated case data.

---

## Required Message Elements

### Request (CMS → EFM)

| Element | Description | Required |
|---------|-------------|----------|
| `EnvelopeID` | ID of the original filing envelope | Yes |
| `FilingID` | CMS-assigned filing identifier | Yes |
| `DocumentIdentifiers` | All CMS document IDs created during docketing | Yes |
| `DocketEntryID` | Unique docket entry identifier | Recommended but expected |
| `ReviewedBy` | Clerk or auto-accept processor ID | Recommended |
| `DocketDate` | When docketing occurred | Yes |
| `CaseTrackingID` | CMS case number used for consistency | Yes |
| `CourtIdentifier` | Routing identifier | Yes |

### Response (EFM → CMS)

| Element | Description |
|---------|-------------|
| `MessageReceipt` | Standard ECF acknowledgment |
| `Status` | Success or error context |
| `Timestamp` | When receipt processed |

---

## Relationship to Other APIs

NotifyDocketingComplete works together with:

- **RecordFiling** – submitted from EFM → CMS to initiate docketing  
- **GetCase** – used after docketing to retrieve updated metadata  
- **NotifyCaseEvent** – may also be sent when CMS performs non-filing updates  
- **GetDocument** – re:Search uses this to retrieve actual documents referenced in docketing  

This message finalizes the e-filing transaction and ensures data flows downstream consistently.

---

## Validation Requirements

### CMS Must Ensure:

- All returned DocumentIDs match those referenced in GetCase  
- DocumentIDs are unique and stable  
- CMS dates and timestamps are accurate  
- EnvelopeID matches the RecordFiling request  
- CaseTrackingID matches the docketed case  

### EFM Will Reject:

- Missing or duplicate DocumentIDs  
- Mismatched EnvelopeID  
- Invalid XML schema  
- Missing required filing identifiers  

---

## Common Vendor Mistakes

| Mistake | Result |
|---------|--------|
| Not returning all document IDs | re:Search and EFSP show missing documents |
| Returning different IDs in GetCase | Inconsistent indexing and broken UI references |
| Using temporary or session‐based document identifiers | Future GetDocument calls fail |
| Incorrect envelope or filing ID mapping | EFM rejects the transaction |
| Missing docket date | EFM may mark the filing as incomplete |

---

## Examples

XML examples can be found here:

technical/api-reference/notify-docketing-complete/examples/
---

## Example URLs (Developer Quick Links)

| Example | URL |
|---------|-----|
| Successful Docketing | `/technical/api-reference/notify-docketing-complete/examples/success.xml` |
| Missing Document ID Fault | `/technical/api-reference/notify-docketing-complete/examples/missing-doc-id.xml` |
| Envelope Mismatch | `/technical/api-reference/notify-docketing-complete/examples/envelope-mismatch.xml` |
| Docket Failure | `/technical/api-reference/notify-docketing-complete/examples/docket-failure.xml` |

(Placeholders until populated.)

---

## Related API Reference Pages

| API | Path | Purpose |
|------|------|---------|
| **RecordFiling** | [../record-filing/index.md](../record-filing/index.md) | Submits filings to CMS for docketing |
| **GetCase** | [../get-case/index.md](../get-case/index.md) | Retrieves updated case metadata post-docketing |
| **NotifyCaseEvent** | [../notify-case-event/index.md](../notify-case-event/index.md) | CMS-initiated update events for all non-filing changes |
| **GetDocument** | [../get-document/index.md](../get-document/index.md) | Retrieves document binaries |

---