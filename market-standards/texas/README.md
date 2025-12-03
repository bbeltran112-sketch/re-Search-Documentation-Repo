# Texas Market Standards

> **Breadcrumb:** [Home](../../README.md) > [Market Standards](../README.md) > Texas

Texas-specific requirements for re:Search integration. Texas courts must comply with JCIT (Judicial Committee on Information Technology) standards in addition to universal re:Search requirements.

---

## Overview

**Standards Body:** JCIT (Judicial Committee on Information Technology)  
**Jurisdiction:** Texas state courts  
**Compliance:** Required for all Texas re:Search implementations

Texas has established statewide standards for case management and electronic filing through JCIT. These standards define:
- Approved case types and codes
- Required event types for case updates
- Security and public access rules
- Data field requirements and mappings
- Publication timing and delays

**Important:** Implement [universal re:Search requirements](../../api-reference/README.md) first, then add Texas-specific rules.

---

## Texas-Specific Requirements

### [Case Types](./case-types/README.md)
JCIT-approved case type codes and definitions

**What you'll find:**
- Complete list of approved Texas case types
- Case type codes (CR, CV, FM, etc.)
- How to map your CMS case types to JCIT codes
- Restrictions and special handling by case type

**Key Topics:**
- Criminal (CR) case types
- Civil (CV) case types
- Family (FM) case types
- Juvenile (JV) case types
- Probate and other specialized types

**Use this when:** Mapping CMS case types or implementing case type validation

---

### [Event Types](./event-types/README.md)
Required event types and when to send them

**What you'll find:**
- JCIT event type requirements
- Which events are required for which case types
- Event type definitions and usage
- Special event handling rules

**Key Topics:**
- Required events by case type
- CaseFiling event requirements
- CaseDisposition event requirements
- Party and document event requirements

**Use this when:** Implementing NotifyCaseEvent or troubleshooting event issues

---

### [Security Requirements](./security/README.md)
Case and document security rules for Texas

**What you'll find:**
- JCIT security levels and definitions
- Case-level security (sealed, confidential, public)
- Document-level security overrides
- Publication delay rules (31-day, 180-day)
- Registered user requirements

**Key Topics:**
- Criminal case security rules
- Family case confidentiality
- Juvenile case restrictions
- Sealed case handling
- Document security inheritance

**Use this when:** Implementing security logic or diagnosing access issues

---

### [Public Access Matrix](./public-access-matrix/README.md)
Who can see what based on case type and security level

**What you'll find:**
- Access rules by case type
- Role-based visibility (Public, Attorney, Registered User, Court Staff)
- JCIT role definitions
- Special access rules and exceptions

**Key Topics:**
- Public access to criminal cases
- Family case visibility restrictions
- Juvenile case access rules
- Attorney access rights
- Registered user privileges

**Use this when:** Implementing access controls or troubleshooting visibility issues

---

### [Data Mapping](./data-mapping/README.md)
Field mappings and data format requirements

**What you'll find:**
- Required fields by case type
- CMS field to JCIT standard mappings
- Data format requirements
- Validation rules

**Key Topics:**
- Party data requirements
- Attorney data requirements
- Case metadata fields
- Document metadata fields
- Index data standards

**Use this when:** Implementing GetCase responses or validating data quality

---

### [Index Data Requirements](./index-data/README.md)
Required searchable fields and indexing rules

**What you'll find:**
- Fields required for public search
- Indexing requirements by case type
- Search optimization guidelines
- JCIT index standards

**Key Topics:**
- Party name indexing
- Case number formats
- Date field requirements
- Attorney indexing
- Document text indexing

**Use this when:** Implementing search functionality or optimizing queries

---

### [Integration Rules](./integration-rules/README.md)
Texas-specific integration requirements

**What you'll find:**
- JCIT integration standards
- API requirements specific to Texas
- Timing and delay rules
- Compliance validation

**Key Topics:**
- 31-day publication delay for criminal cases
- 180-day delay for sealed documents
- Event timing requirements
- Data refresh intervals

**Use this when:** Planning integration architecture or implementing timing logic

---

## JCIT Compliance Overview

### What Makes Texas Different?

**Publication Delays**
- Criminal cases: 31-day delay before public visibility
- Sealed documents: 180-day delay before availability
- Family cases: Immediate but restricted access

**Case Type Restrictions**
- Only JCIT-approved case types allowed
- Strict case type code enforcement
- Case type-specific event requirements

**Security Model**
- JCIT-defined roles (Public, Attorney, Registered User, Court Staff)
- Case type-specific access rules
- Document-level security overrides

**Data Standards**
- Required fields defined by JCIT
- Specific format requirements
- Index data standards

---

## Implementation Workflow for Texas

### Step 1: Review JCIT Standards
**Before development:**
- [ ] Read [JCIT Case Types](./case-types/README.md)
- [ ] Understand [Security Requirements](./security/README.md)
- [ ] Review [Event Type Requirements](./event-types/README.md)
- [ ] Study [Public Access Matrix](./public-access-matrix/README.md)

### Step 2: Map Your Data
**During planning:**
- [ ] Map CMS case types to JCIT codes
- [ ] Identify required fields vs available fields
- [ ] Document security level mappings
- [ ] Plan event generation strategy

[Use Data Mapping Guide →](./data-mapping/README.md)

### Step 3: Implement JCIT Rules
**During development:**
- [ ] Enforce JCIT case type codes
- [ ] Implement publication delays
- [ ] Apply security matrix rules
- [ ] Generate required events

[See Integration Rules →](./integration-rules/README.md)

### Step 4: Test JCIT Compliance
**During certification:**
- [ ] Validate case type compliance
- [ ] Test publication delay logic
- [ ] Verify security enforcement
- [ ] Confirm event coverage

[Testing Requirements →](../../integration-guide/onboarding/06-testing-and-certification.md)

---

## JCIT Roles and Visibility

Texas defines four primary user roles with different access levels:

### Public
**Access Level:** Most restricted
- Can view public cases only
- Cannot see sealed or confidential cases
- Subject to publication delays
- Limited document access

### Attorney
**Access Level:** Standard professional access
- Can view cases where they represent a party
- Can see attorney-accessible documents
- May have earlier access than public
- Still restricted from sealed cases (unless involved)

### Registered User
**Access Level:** Enhanced access
- County/court-specific registration required
- Can view cases in their registered jurisdiction
- Earlier access to some case types
- Subject to JCIT registered user matrix

### Court Staff
**Access Level:** Full administrative access
- Can view all cases in their court
- No publication delays
- Access to sealed cases
- Document management capabilities

[Complete role definitions →](./public-access-matrix/README.md)

---

## Common Texas-Specific Scenarios

### Criminal Case Filing
```
1. Case filed in CMS
   ↓
2. CMS sends CaseFiling event
   ↓
3. re:Search indexes case
   ↓
4. 31-day delay period begins
   ↓
5. After 31 days → Public can search case
```

[Criminal case requirements →](./case-types/README.md#criminal-cases)

### Family Case with Sealed Documents
```
1. Family case visible immediately (restricted)
   ↓
2. Specific documents marked sealed
   ↓
3. Public can see case but not sealed docs
   ↓
4. Registered users may access based on rules
   ↓
5. 180-day seal period may apply
```

[Family case requirements →](./case-types/README.md#family-cases)

### Party Attorney Update
```
1. Attorney added to case in CMS
   ↓
2. CMS sends CasePartyAttorney event
   ↓
3. re:Search updates case index
   ↓
4. Attorney gains access to case
   ↓
5. Attorney can view case documents
```

[Party event requirements →](./event-types/README.md#party-events)

---

## JCIT Compliance Checklist

Before go-live in Texas, verify:

### Case Types
- [ ] All case types use JCIT-approved codes
- [ ] Case type mapping documented and validated
- [ ] Unknown case types handled per JCIT rules
- [ ] Case type-specific requirements implemented

### Events
- [ ] All required events implemented by case type
- [ ] Event timing meets JCIT standards
- [ ] Criminal case events include disposition
- [ ] Party/attorney events working correctly

### Security
- [ ] 31-day delay implemented for criminal cases
- [ ] 180-day delay implemented for sealed documents
- [ ] Security levels map correctly to JCIT definitions
- [ ] Public access matrix enforced

### Data Quality
- [ ] All JCIT-required fields populated
- [ ] Field formats match JCIT standards
- [ ] Party/attorney data complete
- [ ] Index data meets requirements

### Roles & Access
- [ ] JCIT roles implemented correctly
- [ ] Registered user matrix applied
- [ ] Attorney access working properly
- [ ] Court staff access appropriate

---

## JCIT Resources

### Official JCIT Standards
**Texas Office of Court Administration**  
Website: [https://www.txcourts.gov/jcit/](https://www.txcourts.gov/jcit/)

**JCIT Standards Documentation**  
Available through Texas OCA and your TPM

### re:Search Support
**Technical Questions:**  
Contact your assigned Tyler Technical Project Manager (TPM)

**JCIT Interpretation:**  
Work with your TPM to clarify JCIT requirements

**Compliance Validation:**  
BIS team validates JCIT compliance during certification

---

## Common Questions

**Do all Texas courts follow JCIT standards?**  
Yes. JCIT compliance is required for all Texas re:Search implementations.

**What if my CMS doesn't support JCIT case type codes?**  
You must create a mapping layer. See [Case Types](./case-types/README.md) for guidance.

**How do I handle the 31-day delay?**  
re:Search handles this automatically based on case type and filing date. [See Security Requirements](./security/README.md).

**What if a case type isn't in the JCIT list?**  
Contact your TPM. New case types require JCIT approval before implementation.

**Can I use different event types than JCIT requires?**  
No. JCIT event type requirements are mandatory. [See Event Types](./event-types/README.md).

---

## Related Documentation

### For Core Requirements
→ [API Reference](../../api-reference/README.md) - Universal API specifications

### For Integration Process
→ [Integration Guide](../../integration-guide/README.md) - Step-by-step onboarding

### For Implementation Help
→ [Implementation Playbook](../../implementation-playbook/README.md) - Best practices and patterns

### For Other Markets
→ [Market Standards](../README.md) - Compare with other jurisdictions

---

**Last Updated:** January 2025  
**JCIT Standards Version:** Current as of publication date - verify with Texas OCA for latest