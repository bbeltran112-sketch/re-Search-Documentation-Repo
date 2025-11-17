# NotifyCaseEvent – DO / DON’T Guide for CMS Integrators

This guide defines mandatory rules CMS vendors must follow to ensure re:Search properly processes case updates.

> **Critical Rule:**  
> re:Search only processes the XML segments corresponding to the `eventType`.  
> All other updates—even if correct—are ignored.

This rule controls *all* event-driven indexing.

---

# 1. Case Security (Sealed, Unsealed, Restricted)

| Action | Correct Behavior | Why |
|--------|------------------|------|
| **DO** send `eventType = CaseSecurity` | re:Search processes `<CaseSecurity>` | Only processed when eventType matches |
| **DON’T** send security updates under CaseFiling or CaseParty | Security ignored | Wrong event type |

---

# 2. Filings & Docket Entries

| Action | Correct Behavior | Why |
|--------|------------------|------|
| **DO** use `CaseFiling` | Filing and docket entry blocks processed | Filing logic runs only for CaseFiling |
| **DON’T** expect filing changes to appear when using CaseParty or CaseSecurity | Ignored | Event mismatch |

---

# 3. Parties & Party Attorneys

| Action | Correct Behavior | Why |
|--------|------------------|------|
| **DO** use `CaseParty` or `CasePartyAttorney` | Parties/attorneys updated | re:Search maps directly to these event types |
| **DON’T** include party changes under CaseFiling | Ignored | Wrong event type |

---

# 4. Reassignment (Judge, Court, Location)

| Action | Correct Behavior | Why |
|--------|------------------|------|
| **DO** send `CaseReassigned` | Judge/court/location updated | Only event type scanned for reassignment |
| **DON’T** rely solely on GetCaseResponse | Ignored | re:Search requires correct event type |

---

# 5. Case Disposition

| Action | Correct Behavior | Why |
|--------|------------------|------|
| **DO** use `CaseDisposition` | Case disposition updated | Only processed during CaseDisposition |
| **DON’T** send disposition updates under CaseFiling | Ignored | Wrong event type |

---

# 6. General Rules

### DO:
- Send **full** GetCaseResponse after every NCE  
- Match eventType **exactly** (case-sensitive)  
- Send multiple events for multiple update types  
- Ensure FilingID present for CaseFiling  

### DON’T:
- Assume re:Search reads the entire GetCaseResponse  
- Send partial case data  
- Mix multiple update types into one event  
- Use incorrect casing (`sealed`, `PUBLIC`, etc.)  

---

_Back to NotifyCaseEvent Overview:_  
➡️ [README.md](./README.md)

_Back to Event Types:_  
➡️ [event-types.md](./event-types.md)
