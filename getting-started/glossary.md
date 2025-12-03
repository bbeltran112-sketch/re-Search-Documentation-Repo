# Glossary

## Table of Contents

**Quick Navigation by Letter:**  
[A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [I](#i) | [J](#j) | [L](#l) | [M](#m) | [N](#n) | [O](#o) | [P](#p) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x)

**Additional Resources:**
- [Common Abbreviations Table](#common-abbreviations)
- [Need More Information?](#need-more-information)

---

Common terms, acronyms, and definitions used in re:Search documentation and the judicial e-filing ecosystem.

## A

### Access Code
A secure code provided to parties for accessing sealed or restricted cases through an EFSP portal.

### API (Application Programming Interface)
A set of protocols and tools that allow different software systems to communicate with each other. re:Search uses SOAP/XML-based APIs.

### Appellate
Relating to appeals of lower court decisions. Appellate cases follow different rules and workflows than trial court cases.

### Attorney of Record
An attorney formally registered with the court to represent a party in a case. Must be properly associated with a case to file documents on behalf of a client.

## B

### Batch Mode
An integration method where case data is exchanged through scheduled file transfers rather than real-time API calls.

### Base64 Encoding
A method of encoding binary data (like PDF files) into text format for transmission in XML messages.

## C

### Case Initiation
The process of creating a new case in the court system, typically through filing a complaint or petition.

### Case Management System (CMS)
Software used by courts to manage case records, documents, calendaring, and workflows. Examples include Tyler Odyssey, Tyler Eagle, and Proteus.

### Case Tracking ID
A unique identifier assigned to a case, used to reference the case in API messages. May differ from the public case number.

### Case Type
A classification of cases by legal category (e.g., Civil, Criminal, Family, Probate). Must match between CMS and re:Search for proper routing.

### CJIS (Criminal Justice Information Services)
FBI security standards that apply to systems handling criminal justice data. re:Search complies with CJIS requirements.

### CIP (Court Integration Platform)
Middleware that enables legacy CMS systems to integrate with re:Search by translating between different data formats and protocols.

### Clerk Review
The process where court staff review electronically filed documents before accepting them into the official record.

### Code
In the context of parties, a code represents the role or relationship type (e.g., Attorney, Guardian, Expert Witness).

## D

### Docket Entry
A record of an event or filing in a case, typically with a date, description, and associated documents.

### Docketing Complete
The final stage of the filing process where the court has reviewed and either accepted or rejected a filing.

### Document Metadata
Descriptive information about a document, including title, filing date, filer, document type, and access restrictions.

### Document Stamp
An official mark or notation added by the court to indicate a document has been filed, typically including date and time.

## E

### ECF (Electronic Court Filing)
The OASIS LegalXML standard (version 5.0) used for electronic filing message formats. Also refers generally to electronic court filing systems.

### EFSP (Electronic Filing Service Provider)
A certified third-party vendor that provides a portal for attorneys and self-represented litigants to file documents electronically.

### Envelope ID
A unique identifier for a specific filing submission, used to track the filing through its lifecycle.

### Event Type
A classification code indicating what kind of case activity occurred (e.g., case-filing, document-filing, case-disposition).

### Expungement
The legal process of sealing or destroying court records. When a case is expunged, it must be removed from public access systems.

## F

### Filer
The person or entity submitting documents to the court, whether through an EFSP or directly.

### Filing Assembly
A collection of documents submitted together as a single filing transaction.

### Filing Code
A court-specific code that identifies the type of document being filed (e.g., Complaint, Motion, Answer).

### Filing Fee
The amount charged by the court for accepting a filing. May vary by document type and case type.

## G

### GetCase
An API endpoint that allows EFSPs to retrieve case information from re:Search, including parties, documents, and case status.

### GetDocument
An API endpoint that allows authorized users to retrieve the actual content of filed documents.

### Go-Live
The date when a court or EFSP begins using re:Search in production for real filings.

## I

### Index Data
Key case information extracted from documents or case records for searching and reporting purposes.

### Integration Mode
The method by which a court's CMS connects to re:Search (ECF, CIP, Batch, or Non-Integrated).

## J

### JCIT (Judicial Case Information Technology)
The standards and systems for electronic filing and case management in state courts.

### JSON (JavaScript Object Notation)
A lightweight data format sometimes used for configuration or logging, though re:Search primarily uses XML.

## L

### Lead Document
The primary document in a filing assembly, typically containing the substantive pleading or motion.

### Legacy System
An older CMS that may lack modern API capabilities, requiring CIP or Batch mode integration.

## M

### Message ID
A unique identifier assigned to each API message for tracking and troubleshooting purposes.

### Metadata
Data about data. In re:Search, this includes information about documents, cases, and parties rather than the content itself.

### MTOM (Message Transmission Optimization Mechanism)
A method for efficiently transmitting binary data (like PDFs) in SOAP messages, used by some ECF implementations.

## N

### NIEM (National Information Exchange Model)
A standards framework that ECF builds upon, providing common vocabulary for justice information exchange.

### NotifyCaseEvent
An API endpoint where courts send case updates (new parties, dispositions, document filings, etc.) to re:Search.

### NotifyDocketingComplete
An API endpoint where re:Search sends filing acceptance/rejection notifications to EFSPs.

## O

### OASIS
Organization for the Advancement of Structured Information Standards, the body that maintains ECF standards.

### Odyssey
A popular CMS platform by Tyler Technologies that supports native ECF integration.

## P

### Party
An individual or entity involved in a case (plaintiff, defendant, attorney, etc.).

### Party ID
A unique identifier assigned to a party in a specific case.

### Payload
The main content of an API message, containing the actual case data or filing information.

### Production Environment
The live system where real court filings are processed, as opposed to test/development environments.

### Public Access
Information about cases and documents that is available to the general public, subject to privacy and security restrictions.

### Public Access Matrix
A set of rules defining what case information is publicly accessible based on case type, party type, and document type.

## R

### Receipt
An acknowledgment sent to the filer immediately upon submission, confirming the court has received the filing.

### RecordFiling
The primary API endpoint where EFSPs submit electronic filings to re:Search.

### Registered User
A person who has created an account with an EFSP and been verified, allowing them to file documents and access case information.

### Rejection
When a court declines to accept a filing, typically due to errors, omissions, or procedural issues.

### REST (Representational State Transfer)
A modern API architecture style. re:Search primarily uses SOAP/XML but may support REST in future versions.

## S

### Schema
An XSD (XML Schema Definition) file that defines the structure and rules for XML messages.

### Sealed Case
A case where all information is restricted from public access by court order, typically in sensitive matters.

### Security Event
A notification from the court that case access restrictions have changed.

### Self-Represented Litigant (SRL)
A party representing themselves without an attorney. Also called "pro se" litigants.

### Service Contact
A party or attorney who should receive official notices and served documents.

### Service of Process
The official delivery of legal documents to parties, often handled electronically through re:Search.

### SOAP (Simple Object Access Protocol)
A protocol for exchanging XML messages over HTTP, used by re:Search APIs.

### Staging Environment
A pre-production testing environment that mirrors production settings, used for final validation before go-live.

## T

### TLS (Transport Layer Security)
Encryption protocol used to secure communications between systems. re:Search requires TLS 1.2 or higher.

### Token
A credential used for API authentication, typically time-limited for security.

### Transaction ID
A unique identifier for a specific API request/response pair, used for logging and troubleshooting.

### Tyler Technologies
A major vendor of court management systems (Odyssey, Eagle, Modria).

## U

### URI (Uniform Resource Identifier)
A string that identifies a resource, often used as namespace identifiers in XML.

### URL (Uniform Resource Locator)
The web address where API endpoints are accessed (e.g., https://research.courts.state/services/RecordFiling).

### UTF-8
A character encoding standard that supports all Unicode characters, required for re:Search XML messages.

## V

### Validation
The process of checking that a submitted filing meets all technical and business rules before acceptance.

### Venue
The geographic location or specific court where a case should be filed.

## W

### Web Service
A software system designed to support interoperable machine-to-machine interaction over a network.

### Wrapper
An outer XML envelope that contains authentication and routing information along with the main message payload.

## X

### XML (eXtensible Markup Language)
A markup language used for structuring data in re:Search API messages. All API messages are in XML format.

### XSD (XML Schema Definition)
A file that defines the structure, elements, and validation rules for XML documents. ECF has published XSD files.

## Common Abbreviations

| Abbreviation | Full Term | Context |
|--------------|-----------|---------|
| ADA | Americans with Disabilities Act | Accessibility requirements |
| AKA | Also Known As | Alternate party names |
| API | Application Programming Interface | Technical integration |
| CIP | Court Integration Platform | Integration mode |
| CJIS | Criminal Justice Information Services | Security standards |
| CMS | Case Management System | Court software |
| DOB | Date of Birth | Party information |
| ECF | Electronic Court Filing | Filing standard |
| EFSP | Electronic Filing Service Provider | Vendor type |
| FBI | Federal Bureau of Investigation | CJIS authority |
| HTTP | Hypertext Transfer Protocol | Communication protocol |
| HTTPS | HTTP Secure | Encrypted HTTP |
| JCIT | Judicial Case Information Technology | Court technology |
| MTOM | Message Transmission Optimization | Binary attachment method |
| NIEM | National Information Exchange Model | Data standard |
| OASIS | Organization for the Advancement of Structured Information Standards | Standards body |
| PDF | Portable Document Format | Document format |
| REST | Representational State Transfer | API architecture |
| SOAP | Simple Object Access Protocol | Message protocol |
| SRL | Self-Represented Litigant | Unrepresented party |
| SSN | Social Security Number | PII (restricted) |
| TLS | Transport Layer Security | Encryption |
| URI | Uniform Resource Identifier | Namespace identifier |
| URL | Uniform Resource Locator | Web address |
| UTF-8 | Unicode Transformation Format 8-bit | Character encoding |
| XML | eXtensible Markup Language | Data format |
| XSD | XML Schema Definition | Schema format |

## Need More Information?

If you encounter terms not defined here:

- Check the [API Reference](../api/README.md) for technical terms
- Review [Standards Documentation](../standards/README.md) for ECF/NIEM terminology
- See [FAQ](../reference/faq.md) for commonly confused concepts
- Contact support for clarification on court-specific terminology

---

**Last Updated:** November 2025  
**Contributions:** If you notice missing terms, please submit feedback through your court liaison or EFSP contact.