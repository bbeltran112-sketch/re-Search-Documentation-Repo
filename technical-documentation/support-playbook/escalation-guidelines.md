# 11. Escalation Guidelines

This section defines when BIS should escalate issues internally or externally.

The purpose is to provide consistent and predictable escalation behavior for re:SearchTX support.

## 11.1 Escalation to DAS Engineering

Escalate to DAS when:
- NotifyCaseEvent messages are received but not processed
- Queue backlogs persist or grow unexpectedly
- Messages appear stuck or unacknowledged
- Indexing fails despite correct inputs
- Search results differ from DAS logs
- EventType processing appears incorrect

## 11.2 Escalation to EFM Team

Escalate to EFM when:
- Document metadata mismatch occurs
- Filing metadata is missing or malformed
- DocumentSecurity originates from EFM and is incorrect
- File associations (Envelope, Filing ID) appear corrupted

## 11.3 Escalation to CMS Vendor

Escalate back to vendor when:
- EventType is incorrect
- CaseSecurity or DocumentSecurity missing/inaccurate
- CaseType not JCIT-compliant
- Required fields missing (FiledDate, Category, Party list, etc.)
- Style formatting incorrect
- DocumentSecurity not resent after changes

## 11.4 Escalation to Product/Leadership

Escalate internally when:
- County-level misunderstanding requires policy clarification
- Vendor is refusing to send required fields
- Business logic conflicts with JCIT rules
- Unclear configuration issue requiring cross-team alignment

## 11.5 BIS Escalation Workflow

1. Validate security values
2. Validate EventType(s)
3. Validate GetCaseResponse structure
4. Identify integration mode (CMS or EFM)
5. Replicate visibility behavior internally
6. Identify escalation target
7. Prepare summary + message traffic
8. Submit escalation with timestamps and screenshot links
