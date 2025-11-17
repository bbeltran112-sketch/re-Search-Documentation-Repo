# NotifyCaseEvent – Event Types Reference

This document details how each `eventType` influences re:Search case indexing.

> **Critical Rule:**  
> re:Search *only* processes the XML elements associated with the `eventType` provided in NotifyCaseEvent.  
> All unrelated XML segments are ignored.

---

## CaseFiling

Triggers processing of:
- `<Filing>`  
- `<DocketEntry>`  
- Filing-level security  
- Associated document metadata  

Used when:
- Clerk creates or updates a filing  
- CMS adds a docket entry outside e-filing  
- Filing metadata changes  

Ignored if sent under other event types.

---

## CaseParty

Triggers processing of:
- `<Party>`  
- `<PartyAssociation>`  
- `<Role>`  

Used for:
- Adding, updating, or removing parties  

Party changes under any other event type will be ignored.

---

## CasePartyAttorney

Triggers processing of:
- `<Attorney>`  
- `<PartyAttorneyAssociation>`  

Used for:
- Assigning new attorneys  
- Removing or updating existing attorney-party links  

---

## CaseReassigned

Triggers processing of:
- `<Judge>`  
- `<CourtLocation>`  
- `<CaseAssignment>`  

Used when:
- Judge changes  
- Court or location changes  
- Case reassigned to new docket or courtroom  

---

## CaseDisposition

Triggers processing of:
- `<Disposition>`  
- `<CaseStatus>`  
- `<CloseReason>`  

Used when:
- Case is resolved  
- Case status updated  
- Disposition metadata modified  

---

## CaseSecurity

Triggers processing of:
- `<CaseSecurity>`  

Used when:
- Case is sealed  
- Case becomes public  
- Case status changes to restricted or confidential  

Security updates under any other event type will be ignored.

---

## DocumentSecurity

Triggers processing of:
- `<Document>` visibility and confidentiality metadata  

Used for:
- Restricting or unrestricting specific documents  

---

## CaseDelete

Triggers processing of:
- Case deletion or restoration status  

Used when:
- Case is removed or archived  
- Case restored to active status  

---

## CaseExpunge

Triggers processing of:
- `<ExpungementInfo>`  
- Case-level sealing or redaction metadata  

Used when:
- Case is expunged  
- Records are restricted based on statutory rules  

---

_Back to NotifyCaseEvent Overview:_  
➡️ [README.md](./README.md)

_See the DO/DON’T Guide:_  
➡️ [do-dont-guide.md](./do-dont-guide.md)
