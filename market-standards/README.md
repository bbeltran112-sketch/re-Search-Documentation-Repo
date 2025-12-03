# Market Standards

> **Breadcrumb:** [Home](../README.md) > Market Standards

Market-specific requirements and standards that vary by jurisdiction. These supplement the universal re:Search requirements with state or regional rules.

---

## Overview

re:Search has two types of requirements:

**Universal Requirements** (apply to all courts):
- Core API structure and behavior
- Basic security concepts
- General data flow patterns
- Standard authentication methods

**Market-Specific Requirements** (vary by jurisdiction):
- Allowed case types
- Required event types
- Public access rules
- Security matrix definitions
- Data mapping standards

**Important:** Always implement universal requirements first, then add your market's specific rules.

---

## Available Markets

### [Texas](./texas/README.md)
**Standards Body:** JCIT (Judicial Committee on Information Technology)

Texas courts must comply with JCIT standards in addition to core re:Search requirements.

**Key Requirements:**
- JCIT-approved case types only
- Specific event type requirements
- 31-day and 180-day publication delays
- JCIT security matrix and roles
- Registered user visibility rules

**Documentation:**
- [Texas Overview](./texas/README.md)
- [Case Types](./texas/case-types/README.md)
- [Event Types](./texas/event-types/README.md)
- [Security Requirements](./texas/security/README.md)
- [Public Access Matrix](./texas/public-access-matrix/README.md)
- [Data Mapping](./texas/data-mapping/README.md)
- [Index Data Requirements](./texas/index-data/README.md)
- [Integration Rules](./texas/integration-rules/README.md)

---

### [Illinois](./illinois/README.md)
**Standards Body:** AOIC (Administrative Office of Illinois Courts)

Illinois courts must comply with AOIC standards in addition to core re:Search requirements.

**Key Requirements:**
- AOIC-approved case types only
- Illinois-specific event types
- AOIC security rules
- Illinois public access standards
- State data mapping requirements

**Documentation:**
- [Illinois Overview](./illinois/README.md)
- [Case Types](./illinois/case-types/README.md)
- [Event Types](./illinois/event-types/README.md)
- [Security Requirements](./illinois/security/README.md)
- [Public Access Matrix](./illinois/public-access-matrix/README.md)
- [Data Mapping](./illinois/data-mapping/README.md)
- [Index Data Requirements](./illinois/index-data/README.md)
- [Integration Rules](./illinois/integration-rules/README.md)

---

### [Other Markets](./other-markets/README.md)
**Standards Body:** None (universal requirements only)

Markets without specific standards body requirements follow core re:Search standards only.

**What applies:**
- Universal API requirements
- Standard security concepts
- Core event types
- Basic data mapping

**What doesn't apply:**
- State-specific case type restrictions
- Publication delay rules
- Jurisdiction-specific security matrices

**Documentation:**
- [Other Markets Overview](./other-markets/README.md)

---

## Understanding Market Requirements

### What's Market-Specific?

**Case Types**
- Which case types are allowed in the jurisdiction
- Local case type codes and names
- How CMS case types map to standard codes

**Event Types**
- Which event types must be supported
- Event type requirements for different case types
- Market-specific event definitions

**Security & Access**
- Case-level security rules (sealed, confidential, public)
- Document-level security overrides
- Role-based access definitions
- Public access matrix by case type

**Data Mapping**
- Required index fields
- Field name mappings
- Data validation rules
- Format requirements

**Integration Rules**
- Market-specific API requirements
- Publication timing rules
- Special processing requirements
- Compliance validation

---

## When to Review Market Standards

### During Planning
**Before choosing integration mode:**
- Review your market's requirements
- Understand complexity implications
- Assess development effort needed

### During Development
**Before implementing APIs:**
- Reference case type mappings
- Implement event type requirements
- Build security logic per market rules
- Follow data mapping specifications

### During Testing
**Before certification:**
- Validate case type compliance
- Test event type coverage
- Verify security enforcement
- Confirm public access rules

---

## How Markets Affect Implementation

### Case Type Mapping
Each market defines allowed case types. Your CMS case types must map to approved market codes.

**Example (Texas):**
```
CMS Case Type          →  JCIT Case Type
"Criminal Felony"      →  "CR" (Criminal)
"Family Law - Divorce" →  "FM" (Family)
"Civil Suit"           →  "CV" (Civil)
```

[Texas Case Types →](./texas/case-types/README.md)  
[Illinois Case Types →](./illinois/case-types/README.md)

### Event Type Requirements
Markets may require specific event types for different case types.

**Example (Texas):**
- Criminal cases require `CaseFiling` and `CaseDisposition` events
- Family cases may have restricted visibility during proceedings
- Civil cases follow standard event patterns

[Texas Event Types →](./texas/event-types/README.md)  
[Illinois Event Types →](./illinois/event-types/README.md)

### Security Rules
Markets define how case and document security works.

**Example (Texas):**
- JCIT defines standard roles and visibility
- 31-day delay for certain criminal cases
- 180-day delay for sealed documents
- Registered user requirements

[Texas Security →](./texas/security/README.md)  
[Illinois Security →](./illinois/security/README.md)

---

## Market Compliance Checklist

Before go-live, verify compliance with your market's requirements:

### Case Types
- [ ] All CMS case types mapped to market codes
- [ ] Only approved case types are used
- [ ] Local case type variations documented
- [ ] Unknown case types handled appropriately

### Event Types
- [ ] All required event types implemented
- [ ] Event types match case type requirements
- [ ] Event generation triggers correct
- [ ] Market-specific events supported

### Security & Access
- [ ] Security levels follow market rules
- [ ] Public access matrix implemented
- [ ] Role-based access working correctly
- [ ] Publication delays applied (if required)

### Data Quality
- [ ] All required fields populated
- [ ] Data formats match market standards
- [ ] Field mappings validated
- [ ] Index data complete

### Integration Rules
- [ ] Market-specific API requirements met
- [ ] Special processing rules implemented
- [ ] Timing requirements followed
- [ ] Compliance validation passing

[Complete testing guide →](../integration-guide/onboarding/06-testing-and-certification.md)

---

## Market-Specific Support

### Texas (JCIT)
**JCIT Standards:** [Office of Court Administration](https://www.txcourts.gov/jcit/)  
**re:Search Documentation:** [Texas Standards](./texas/README.md)

### Illinois (AOIC)
**AOIC Standards:** [Administrative Office of Illinois Courts](http://www.illinoiscourts.gov/)  
**re:Search Documentation:** [Illinois Standards](./illinois/README.md)

### Other Markets
**re:Search Documentation:** [Other Markets](./other-markets/README.md)

---

## Common Questions

**Do I need to implement market standards if I'm in that state?**  
Yes. Market standards are **required** for courts in those jurisdictions.

**Can I operate in multiple markets?**  
Yes. Implement universal requirements plus each market's specific rules.

**What if my market isn't listed?**  
Follow universal requirements only. See [Other Markets](./other-markets/README.md).

**How often do market standards change?**  
Varies by jurisdiction. Texas JCIT reviews annually. Check with your TPM for updates.

**What if my CMS case types don't match market codes?**  
You must create a mapping. See your market's [Case Types](./texas/case-types/README.md) documentation.

---

## Related Documentation

### For Core Requirements
→ [API Reference](../api-reference/README.md) - Universal API specifications

### For Integration Process
→ [Integration Guide](../integration-guide/README.md) - Step-by-step onboarding

### For Implementation Help
→ [Implementation Playbook](../implementation-playbook/README.md) - Best practices and patterns

### For Problem Solving
→ [Troubleshooting Guide](../troubleshooting/README.md) - Issue resolution

---

## Adding New Markets

If your jurisdiction is establishing new standards:

1. Contact your Tyler Technical Project Manager (TPM)
2. Provide standards documentation from your jurisdiction
3. Work with Tyler to define requirements
4. Allow time for documentation and system configuration

New market documentation will be added to this section as standards are established.

---

**Last Updated:** January 2025