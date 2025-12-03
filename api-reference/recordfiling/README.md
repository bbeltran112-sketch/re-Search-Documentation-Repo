# RecordFiling

**Navigation:**  
[Home](../../../README.md) › [Technical Documentation](../../README.md) › [API Reference](../README.md) › RecordFiling

RecordFiling delivers filings and associated metadata from the EFM to the CMS.

---

## Purpose

RecordFiling:

- Transfers the filing package  
- Initiates CMS processing  
- Links the EFM transaction to the CMS case  
- Starts the docketing-to-re:Search update chain  

---

## Transport and Protocol

| Property | Value |
|---------|--------|
| Direction | EFM → CMS |
| Protocol | SOAP 1.2 over HTTPS |
| Security | Mutual TLS |
| Standard | OASIS ECF |
| Message Type | RecordFilingMessage |

Authentication details:  
[Common Headers & Auth](../common-headers-and-auth.md)

---

## When This API Is Used

RecordFiling occurs:

- When an e-filed submission is transmitted  
- During new case creation  
- When delivering envelope contents to the CMS  

---

## Behavior Summary

CMS receives:

- Document metadata  
- Filing metadata  
- Parties and attorneys associated to the envelope  

Then sends:

- NotifyDocketingComplete  
- Which triggers GetCase  
- Which triggers reindexing in re:Search  

---
## Processing Workflow
<!-- XML Placeholder: Insert sample GetDocumentResponse here -->

---

## XML Example

```xml
<RecordFilingRequestMessage
    xmlns="urn:oasis:names:tc:legalxml-courtfiling:wsdl:WebServicesProfile-Definitions-4.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="urn:oasis:names:tc:legalxml-courtfiling:wsdl:WebServicesProfile-Definitions-4.0 ../../Schema/Substitution/RecordFilingRequestMessage.xsd">

    <!-- ============ RecordDocketingMessage ============ -->
    <RecordDocketingMessage
        xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:RecordDocketingMessage-4.0"
        xmlns:tyler="urn:tyler:ecf:extensions:Common"
        xmlns:j="http://niem.gov/niem/domains/jxdm/4.0"
        xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CommonTypes-4.0"
        xmlns:nc="http://niem.gov/niem/niem-core/2.0"
        xmlns:s="http://niem.gov/niem/structures/2.0">
        <!-- [REQUIRED] Official docket timestamp -->
        <nc:DocumentPostDate><nc:DateTime>2014-11-13T11:00:00Z</nc:DateTime></nc:DocumentPostDate>
        <!-- [REQUIRED] Sending MDE identity -->
        <ecf:SendingMDELocationID><nc:IdentificationID>https://filingreviewmde.com</nc:IdentificationID></ecf:SendingMDELocationID>
        <ecf:SendingMDEProfileCode>urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:WebServicesMessaging-2.0</ecf:SendingMDEProfileCode>
        <!-- [REQUIRED] Court mapping -->
        <j:CaseCourt>
            <nc:OrganizationIdentification><nc:IdentificationID>8000</nc:IdentificationID></nc:OrganizationIdentification>
        </j:CaseCourt>
        <!-- [Subsequent only] -->
        <nc:CaseTrackingID/>
        <!-- [REQUIRED] Accepted filings: Lead + Connected -->
        <tyler:ReviewedLeadDocument>
            <nc:DocumentDescriptionText>test</nc:DocumentDescriptionText>
            <nc:DocumentFileControlID>test</nc:DocumentFileControlID>
            <nc:DocumentIdentification><nc:IdentificationID>230a7cf7-5aa7-4b9e-9d42-44243fce8c42</nc:IdentificationID></nc:DocumentIdentification>
            <nc:DocumentSequenceID>0</nc:DocumentSequenceID>

            <ecf:DocumentMetadata>
                <j:RegisterActionDescriptionText>8609</j:RegisterActionDescriptionText>
                <ecf:FilingAttorneyID/>
                <ecf:FilingPartyID/>
            </ecf:DocumentMetadata>
            <ecf:DocumentRendition>
                <ecf:DocumentRenditionMetadata>
                    <ecf:DocumentAttachment>
                        <!-- Depending on market config: may be inline or omitted -->
                        <nc:BinaryBase64Object>base64 will go here</nc:BinaryBase64Object>
                        <nc:BinaryDescriptionText>States.pdf</nc:BinaryDescriptionText>
                        <nc:BinaryFormatStandardName>CONF</nc:BinaryFormatStandardName>
                        <nc:BinaryLocationURI>States.tiff</nc:BinaryLocationURI>
                        <nc:BinaryCategoryText>LEAD</nc:BinaryCategoryText>
                        <ecf:AttachmentSequenceID>0</ecf:AttachmentSequenceID>
                    </ecf:DocumentAttachment>
                    <ecf:RedactedIndicator>false</ecf:RedactedIndicator>
                </ecf:DocumentRenditionMetadata>
            </ecf:DocumentRendition>
            <!-- Reviewer context -->
            <tyler:DocumentReviewer>
                <ecf:EntityPerson>
                    <nc:PersonName>
                        <nc:PersonGivenName>John</nc:PersonGivenName>
                        <nc:PersonSurName>Doe</nc:PersonSurName>
                    </nc:PersonName>
                    <nc:PersonOtherIdentification><nc:IdentificationID>john.doe@tylertech.com</nc:IdentificationID></nc:PersonOtherIdentification>
                </ecf:EntityPerson>
            </tyler:DocumentReviewer>
            <tyler:DocumentReviewDate><nc:DateTime>2014-11-13T17:15:58Z</nc:DateTime></tyler:DocumentReviewDate>
            <tyler:WaiverIndicator>true</tyler:WaiverIndicator>
        </tyler:ReviewedLeadDocument>

        <SealCaseIndicator>false</SealCaseIndicator>
    </RecordDocketingMessage>

    <!-- ============ CoreFilingMessage ============ -->
    <CoreFilingMessage
        xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CoreFilingMessage-4.0"
        xmlns:j="http://niem.gov/niem/domains/jxdm/4.0"
        xmlns:nc="http://niem.gov/niem/niem-core/2.0"
        xmlns:appellate="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:AppellateCase-4.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:ecf="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:CommonTypes-4.0"
        xmlns:s="http://niem.gov/niem/structures/2.0"
        xmlns:tyler="urn:tyler:ecf:extensions:Common">
        <!-- Envelope and submitter context -->
        <nc:DocumentFiledDate><nc:DateTime>2013-07-08T15:33:44Z</nc:DateTime></nc:DocumentFiledDate>
        <!-- Identifiers: ENVELOPEID, ORIGINALENVELOPEID, EFSPNAME -->
        <nc:DocumentIdentification><nc:IdentificationID>12346</nc:IdentificationID><nc:IdentificationCategoryText>ENVELOPEID</nc:IdentificationCategoryText></nc:DocumentIdentification>
        <nc:DocumentIdentification><nc:IdentificationID>12345</nc:IdentificationID><nc:IdentificationCategoryText>ORIGINALENVELOPEID</nc:IdentificationCategoryText></nc:DocumentIdentification>
        <nc:DocumentIdentification><nc:IdentificationID>An EFSP Name</nc:IdentificationID><nc:IdentificationCategoryText>EFSPNAME</nc:IdentificationCategoryText></nc:DocumentIdentification>
        <nc:DocumentSubmitter>
            <tyler:EntityFiler>
                <nc:PersonName><nc:PersonGivenName>Joe</nc:PersonGivenName><nc:PersonSurName>Filer</nc:PersonSurName></nc:PersonName>
                <nc:PersonOtherIdentification><nc:IdentificationID>joe.filer@email.com</nc:IdentificationID></nc:PersonOtherIdentification>
                <tyler:FirmName>Joe Filer Firm</tyler:FirmName>
            </tyler:EntityFiler>
        </nc:DocumentSubmitter>
        <ecf:SendingMDELocationID><nc:IdentificationID>https://filingassemblymde.com</nc:IdentificationID></ecf:SendingMDELocationID>
        <ecf:SendingMDEProfileCode>urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:WebServicesMessaging-2.0</ecf:SendingMDEProfileCode>
        <!-- Example initiation using AppellateCase (only for supported initiation types) -->
        <appellate:AppellateCase>
            <nc:CaseCategoryText>ACCT</nc:CaseCategoryText>
            <nc:CaseTrackingID/>
            <j:AppellateCaseOriginalCase>
                <nc:CaseTitleText>Someone vs Someone</nc:CaseTitleText>
                <nc:CaseDocketID>10-1000</nc:CaseDocketID>
            </j:AppellateCaseOriginalCase>
            <j:CaseAugmentation>
                <j:CaseCourt><nc:OrganizationIdentification><nc:IdentificationID>harris:dc</nc:IdentificationID></nc:OrganizationIdentification></j:CaseCourt>
                <j:CaseLineageCase><nc:CaseTrackingID/></j:CaseLineageCase>
            </j:CaseAugmentation>
            <!-- Parties/attorneys and roles -->
            <tyler:CaseAugmentation>
                <ecf:CaseOtherEntityAttorney>
                    <nc:RoleOfPersonReference s:ref="Attorney1"/>
                    <ecf:CaseRepresentedPartyReference s:ref="Party1"/>
                </ecf:CaseOtherEntityAttorney>
                <ecf:CaseParticipant>
                    <ecf:EntityPerson s:id="Attorney1">
                        <nc:PersonName><nc:PersonGivenName>Steve</nc:PersonGivenName><nc:PersonSurName>Smith</nc:PersonSurName></nc:PersonName>
                        <nc:PersonOtherIdentification>
                            <nc:IdentificationID>037BAEC1-F6E8-43ED-A102-B8C5E76748EB</nc:IdentificationID>
                            <nc:IdentificationCategoryText>ATTORNEYID</nc:IdentificationCategoryText>
                        </nc:PersonOtherIdentification>
                    </ecf:EntityPerson>
                    <ecf:CaseParticipantRoleCode>ATTY</ecf:CaseParticipantRoleCode>
                </ecf:CaseParticipant>
                <ecf:CaseParticipant>
                    <ecf:EntityPerson s:id="Party1">
                        <nc:PersonName><nc:PersonGivenName>John</nc:PersonGivenName><nc:PersonSurName>Doe</nc:PersonSurName></nc:PersonName>
                    </ecf:EntityPerson>
                    <ecf:CaseParticipantRoleCode>P</ecf:CaseParticipantRoleCode>
                </ecf:CaseParticipant>
                <tyler:CaseTypeText>KDM</tyler:CaseTypeText>
                <tyler:FilerTypeText>NCP</tyler:FilerTypeText>
                <tyler:LowerCourtText>Orange</tyler:LowerCourtText>
                <tyler:ProbateEstateAmount>2000.00</tyler:ProbateEstateAmount>
            </tyler:CaseAugmentation>
        </appellate:AppellateCase>
        <!-- Example subsequent filings (lead + connected) -->
        <tyler:FilingLeadDocument s:id="Filing1">
            <nc:DocumentDescriptionText>Test Filing 1</nc:DocumentDescriptionText>
            <nc:DocumentFileControlID>1</nc:DocumentFileControlID>
            <nc:DocumentSequenceID>0</nc:DocumentSequenceID>
            <ecf:DocumentMetadata>
                <j:RegisterActionDescriptionText>CON</j:RegisterActionDescriptionText>
                <ecf:FilingAttorneyID><nc:IdentificationID>037BAEC1-F6E8-43ED-A102-B8C5E76748EB</nc:IdentificationID><nc:IdentificationCategoryText>IDENTIFICATION</nc:IdentificationCategoryText></ecf:FilingAttorneyID>
                <ecf:FilingPartyID><nc:IdentificationID>Party1</nc:IdentificationID><nc:IdentificationCategoryText>REFERENCE</nc:IdentificationCategoryText></ecf:FilingPartyID>
            </ecf:DocumentMetadata>
            <ecf:DocumentRendition>
                <ecf:DocumentRenditionMetadata>
                    <ecf:DocumentAttachment>
                        <!-- In CoreFilingMessage, binaries are typically omitted (see RecordDocketingMessage) -->
                        <nc:BinaryBase64Object/>
                        <nc:BinaryDescriptionText>test1.pdf</nc:BinaryDescriptionText>
                        <nc:BinaryFormatStandardName>CIVILDOC</nc:BinaryFormatStandardName>
                        <nc:BinaryCategoryText>LEAD</nc:BinaryCategoryText>
                        <ecf:AttachmentSequenceID>0</ecf:AttachmentSequenceID>
                    </ecf:DocumentAttachment>
                    <ecf:RedactedIndicator>false</ecf:RedactedIndicator>
                </ecf:DocumentRenditionMetadata>
            </ecf:DocumentRendition>
        </tyler:FilingLeadDocument>
        <tyler:FilingConnectedDocument s:id="Filing2">
            <nc:DocumentDescriptionText>Test Filing 2</nc:DocumentDescriptionText>
            <nc:DocumentFileControlID>2</nc:DocumentFileControlID>
            <nc:DocumentSequenceID>1</nc:DocumentSequenceID>
            <ecf:DocumentMetadata>
                <j:RegisterActionDescriptionText>COR</j:RegisterActionDescriptionText>
            </ecf:DocumentMetadata>
            <ecf:DocumentRendition>
                <ecf:DocumentRenditionMetadata>
                    <ecf:DocumentAttachment>
                        <nc:BinaryBase64Object/>
                        <nc:BinaryDescriptionText>test1.pdf</nc:BinaryDescriptionText>
                        <nc:BinaryFormatStandardName>CIVILDOC</nc:BinaryFormatStandardName>
                        <nc:BinaryCategoryText>LEAD</nc:BinaryCategoryText>
                        <ecf:AttachmentSequenceID>0</ecf:AttachmentSequenceID>
                    </ecf:DocumentAttachment>
                    <ecf:DocumentAttachment>
                        <nc:BinaryBase64Object/>
                        <nc:BinaryDescriptionText>test2.pdf</nc:BinaryDescriptionText>
                        <nc:BinaryFormatStandardName>CIVILDOC</nc:BinaryFormatStandardName>
                        <nc:BinaryCategoryText>ATT</nc:BinaryCategoryText>
                        <ecf:AttachmentSequenceID>1</ecf:AttachmentSequenceID>
                    </ecf:DocumentAttachment>
                    <ecf:DocumentAttachment>
                        <nc:BinaryBase64Object/>
                        <nc:BinaryDescriptionText>test3.pdf</nc:BinaryDescriptionText>
                        <nc:BinaryFormatStandardName>CIVILDOC</nc:BinaryFormatStandardName>
                        <nc:BinaryCategoryText>ATT</nc:BinaryCategoryText>
                        <ecf:AttachmentSequenceID>2</ecf:AttachmentSequenceID>
                    </ecf:DocumentAttachment>
                    <ecf:RedactedIndicator>false</ecf:RedactedIndicator>
                </ecf:DocumentRenditionMetadata>
            </ecf:DocumentRendition>
        </tyler:FilingConnectedDocument>
    </CoreFilingMessage>
</RecordFilingRequestMessage>
```
---

## Example XML Files

| File | Description |
|------|-------------|
| [recordfiling-request-annotated.xml](./examples/recordfiling-request-annotated.xml) | Annotated request |
| [recordfiling-response-receipt-annotated.xml](./examples/recordfiling-response-receipt-annotated.xml) | Annotated receipt |

---

## Related API Pages

- [NotifyDocketingComplete](../notifydocketingcomplete/README.md)  
- [NotifyCaseEvent](../notifycaseevent/README.md)  
- [GetCase](../getcase/README.md)  
- [GetDocument](../getdocument/README.md)  

---

## Related Topics

- [System Behavior Examples](../../support-playbook/system-behavior-examples.md)
- [Troubleshooting Guide](../../support-playbook/troubleshooting.md)
- [Full XML Library](../../xml-library/xml-library.md)
