# CIP Mode (Court Integration Platform)

## Table of Contents

- [Overview](#overview)
- [Important Notice](#important-notice)
- [At a Glance](#at-a-glance)
- [What is CIP Mode](#what-is-cip-mode)
- [How CIP Mode Works](#how-cip-mode-works)
  - [Integration Flow](#integration-flow)
  - [Update Process](#update-process)
- [Key Concepts](#key-concepts)
- [When CIP Mode Was Used](#when-cip-mode-was-used)
- [Current Status and Migration](#current-status-and-migration)
  - [Why the Migration](#why-the-migration)
  - [Migration Timeline](#migration-timeline)
  - [Benefits of Migrating to Batch Mode](#benefits-of-migrating-to-batch-mode)
- [Technical Architecture](#technical-architecture)
  - [Communication Protocol](#communication-protocol)
  - [Authentication](#authentication)
- [CIP Mode vs Other Modes](#cip-mode-vs-other-modes)
- [For Courts Currently Using CIP Mode](#for-courts-currently-using-cip-mode)
  - [What You Need to Know](#what-you-need-to-know)
  - [Migration Process](#migration-process)
  - [Timeline Expectations](#timeline-expectations)
- [Getting Help](#getting-help)
- [Related Documentation](#related-documentation)

---

## Overview

CIP Mode (Court Integration Platform) is a **legacy integration pattern** used primarily by Enterprise Justice (EJ) markets, especially in Texas. It uses REST/XML to send case and filing updates directly to re:Search without involving the Electronic Filing Manager (EFM).

**Important:** CIP Mode is being **phased out** in favor of [Batch Mode](./batch-mode.md), which provides better scalability, easier maintenance, and lower operational complexity.

## Important Notice

**This integration mode is under active migration and will be deprecated.**

- **No new implementations** - New courts should use [Batch Mode](./batch-mode.md)
- **Existing users** - Migration to Batch Mode planned through 2025-2026
- **Support timeline** - Full support until migration complete; maintenance mode after
- **Documentation status** - This page serves as reference for existing implementations

For questions about migration, contact your Technical Project Manager or re:Search support team.

## At a Glance

| Attribute | Description |
|-----------|-------------|
| **Integration Pattern** | Direct REST/XML push from CMS to re:Search |
| **Primary Market** | Enterprise Justice (EJ) installations |
| **Transport Method** | REST API over HTTPS |
| **Data Format** | XML |
| **Real-Time** | Yes (near real-time updates) |
| **Current Status** | **Legacy - Being Replaced by Batch Mode** |
| **New Implementations** | **Not Recommended** |
| **Existing Implementations** | Supported until migration complete |
| **Migration Target** | [Batch Mode](./batch-mode.md) |

## What is CIP Mode

CIP Mode represents a direct integration approach where the Enterprise Justice CMS pushes case and filing updates directly to re:Search as they occur. This was implemented before Batch Mode existed and served as an interim solution for EJ markets.

**Key Characteristics:**

- **Direct Push Model** - EJ sends updates to re:Search immediately when they occur
- **REST-Based** - Uses RESTful API calls over HTTPS
- **XML Messaging** - Data exchanged in XML format
- **No EFM Dependency** - Bypasses Electronic Filing Manager entirely
- **EJ-Specific** - Only works with Enterprise Justice CMS
- **Higher Coupling** - Tight integration between EJ and re:Search environments

## How CIP Mode Works

### Integration Flow

```
┌─────────────┐                    ┌──────────┐
│ Enterprise  │                    │re:Search │
│  Justice    │───────────────────►│ Platform │
│    (EJ)     │  REST/XML          └──────────┘
└─────────────┘  Direct Push              │
      │                                    │
      │ Case/Filing                        │ Index
      │ Event Occurs                       │ Update
      │                                    │
      │ 1. Detect Change                   │
      │                                    │
      │ 2. Build XML                       │
      │    Message                         │
      │                                    │
      │ 3. POST to                         │ 4. Validate
      │    re:Search API    ──────────────►│    XML
      │                                    │
      │                                    │ 5. Process
      │                                    │    Update
      │                                    │
      │ 6. Receive          ◄──────────────┤ 6. Return
      │    Confirmation                    │    Response
```

### Update Process

1. **Event Detection**
   - Case or filing event occurs in Enterprise Justice
   - EJ triggers update notification to re:Search
   - Event types: case filing, party change, disposition, security update

2. **XML Message Generation**
   - EJ builds XML message with case/filing data
   - Includes all relevant metadata
   - Conforms to re:Search XML schema

3. **REST API Call**
   - EJ sends HTTPS POST to re:Search endpoint
   - Authenticated via API key or certificate
   - Includes complete case/filing information

4. **re:Search Processing**
   - Validates XML structure and content
   - Applies business rules
   - Updates search index
   - Returns success/failure response

5. **Confirmation**
   - EJ receives HTTP response
   - Logs success or handles errors
   - May retry on failure

## Key Concepts

### Direct Push Architecture

- **EJ pushes updates directly to re:Search** - No intermediate systems or file storage
- **No EFM involvement** - Bypasses Electronic Filing Manager workflow
- **Real-time updates** - Changes reflected quickly in re:Search
- **Synchronous communication** - EJ waits for response from re:Search

### Environment Coupling

- **Higher coupling** - Direct dependency between EJ and re:Search environments
- **Version dependencies** - EJ and re:Search versions must be compatible
- **Network dependencies** - Requires stable, always-on connectivity
- **Error propagation** - Issues in either system affect the other

### Limited Scope

- **EJ markets only** - Not available for other CMS platforms
- **Texas-specific** - Primarily used in Texas EJ installations
- **Legacy technology** - Based on older integration patterns
- **No new features** - Development frozen in favor of Batch Mode

## When CIP Mode Was Used

CIP Mode was developed for specific use cases:

### Original Use Cases

**Enterprise Justice Markets (2018-2023)**
- Texas courts using Enterprise Justice CMS
- Need for case synchronization with re:Search
- Before Batch Mode was available
- Interim solution during re:Search rollout

**Direct Integration Requirements**
- Courts needing immediate updates
- EJ installations with API capability
- Markets without EFM infrastructure
- Pilot implementations

### Why CIP Mode Existed

1. **EJ-Specific Needs** - Enterprise Justice required custom integration
2. **Timeline Pressures** - Faster to implement than waiting for Batch Mode
3. **Market Demands** - Texas courts needed solution immediately
4. **Technology Available** - EJ already had REST API capability

## Current Status and Migration

### Why the Migration

CIP Mode is being phased out for several important reasons:

**Technical Limitations:**
- Higher operational complexity
- More failure points (network, API, authentication)
- Difficult to troubleshoot and debug
- Version compatibility challenges
- Scalability constraints

**Operational Challenges:**
- Requires constant monitoring
- Error handling complexity
- Network dependency issues
- More support overhead
- Difficult to test and validate

**Better Alternative Available:**
- [Batch Mode](./batch-mode.md) provides superior approach
- Simpler architecture and operation
- Better scalability and performance
- Easier troubleshooting and monitoring
- Lower long-term costs

### Migration Timeline

**Current Phase (2024-2025):**
- All new courts directed to Batch Mode
- Existing CIP Mode courts supported
- Migration planning with affected courts
- Parallel operation during transition

**Active Migration (2025-2026):**
- Court-by-court migration to Batch Mode
- Coordinated cutover windows
- Validation and testing for each court
- Support for both modes during transition

**Completion Target (Mid-2026):**
- All courts migrated to Batch Mode
- CIP Mode endpoints decommissioned
- Final documentation archived
- Lessons learned documented

### Benefits of Migrating to Batch Mode

**Operational Benefits:**
- Reduced complexity and support burden
- Easier troubleshooting and monitoring
- More predictable processing schedules
- Lower operational costs
- Better resource utilization

**Technical Benefits:**
- Handles larger data volumes efficiently
- Better error handling and recovery
- Simplified network requirements
- Easier testing and validation
- More flexible scheduling

**Strategic Benefits:**
- Aligned with re:Search platform direction
- Better long-term supportability
- Consistent integration approach across courts
- Foundation for future enhancements

## Technical Architecture

### Communication Protocol

**REST API Endpoints:**
- Base URL: `https://research-api.courts.state/cip/v1/`
- Endpoints: `/cases`, `/filings`, `/parties`, `/documents`
- Method: HTTP POST
- Content-Type: `application/xml`
- Character Encoding: UTF-8

**XML Message Structure:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CaseUpdate xmlns="http://research.courts.state/cip/schema/v1">
  <CourtCode>ELLIS-CC</CourtCode>
  <CaseId>2024-CV-00123</CaseId>
  <CaseNumber>CV-2024-00123</CaseNumber>
  <UpdateType>CASE_PARTY</UpdateType>
  <UpdateTimestamp>2025-11-19T14:30:00Z</UpdateTimestamp>
  <CaseData>
    <!-- Case details here -->
  </CaseData>
</CaseUpdate>
```

### Authentication

**API Key Authentication:**
- API key provided by re:Search team
- Sent in `X-API-Key` HTTP header
- Keys rotated every 90 days
- Separate keys for test and production

**Certificate-Based (Alternative):**
- Client certificate for mutual TLS
- Certificate provided during onboarding
- Renewal every 12 months
- Used in high-security environments

## CIP Mode vs Other Modes

| Feature | CIP Mode | Batch Mode | ECF Mode |
|---------|----------|------------|----------|
| **Status** | Legacy | Current Standard | Legacy (EFM markets) |
| **Complexity** | High | Low-Medium | Very High |
| **Real-Time** | Yes | No | Yes |
| **Scalability** | Limited | Excellent | Good |
| **Maintenance** | High | Low | High |
| **CMS Support** | EJ Only | Any CMS | ECF-Compatible |
| **Recommended** | No | Yes | Only if EFM exists |
| **New Implementations** | Blocked | Preferred | Only with EFM |

**Clear Recommendation:** New implementations should use [Batch Mode](./batch-mode.md)

## For Courts Currently Using CIP Mode

### What You Need to Know

**Your Integration is Supported:**
- Full support continues during migration period
- No immediate action required
- Migration planned and coordinated with you
- Transition will be smooth and well-managed

**No Disruption to Operations:**
- Your current integration keeps working
- Migration scheduled during low-activity periods
- Testing and validation before cutover
- Rollback plan in place if needed

**You Will Be Contacted:**
- Technical Project Manager will reach out
- Migration timeline discussed and agreed upon
- Technical assistance provided throughout
- Training on new Batch Mode processes

### Migration Process

**Phase 1: Planning (2-3 months before migration)**
1. Technical Project Manager contacts you
2. Review current CIP Mode usage and data
3. Assess Batch Mode requirements
4. Establish migration timeline and windows
5. Identify stakeholders and technical contacts

**Phase 2: Preparation (1-2 months before migration)**
1. Implement Batch Mode export in parallel
2. Test Batch Mode uploads to staging
3. Validate data consistency between modes
4. Train staff on new processes
5. Prepare cutover procedures

**Phase 3: Migration (1-2 weeks)**
1. Final validation in staging environment
2. Schedule cutover window (off-hours)
3. Enable Batch Mode in production
4. Monitor initial batches closely
5. Disable CIP Mode endpoints
6. Confirm successful migration

**Phase 4: Post-Migration (1 month after)**
1. Continued monitoring and support
2. Address any issues promptly
3. Optimize batch schedules if needed
4. Document lessons learned
5. Close migration project

### Timeline Expectations

**Typical Migration Timeline:**
- Planning: 2-3 months
- Preparation: 1-2 months
- Migration: 1-2 weeks
- Post-migration support: 1 month
- **Total: 4-6 months** from initial contact to completion

**Your Involvement:**
- 2-4 hours for planning meetings
- 10-20 hours for testing and validation
- 4-8 hours during cutover weekend
- Minimal ongoing effort after migration

## Getting Help

### For Current CIP Mode Users

**Migration Questions:**
- Contact your Technical Project Manager
- Email: research-migration@courts.state
- Include your court code and CIP Mode details

**Technical Support:**
- Current issues: research-support@courts.state
- Phone: Available during business hours
- Include API logs and error messages

**Emergency Support:**
- Critical production issues
- Contact TPM directly
- Escalation procedures available

### For New Implementations

**Do Not Implement CIP Mode:**
- CIP Mode is not available for new courts
- Use [Batch Mode](./batch-mode.md) instead
- Contact re:Search team to get started
- Review [Onboarding Guide](../onboarding/README.md)

## Related Documentation

### Integration Modes
- [Integration Modes Overview](./README.md) – Compare all integration methods
- [Batch Mode](./batch-mode.md) – **Recommended replacement for CIP Mode**
- [ECF Mode](./ecf-mode.md) – Real-time API-driven integration for EFM markets
- [Non-Integrated Mode](./non-integrated-mode.md) – Manual portal-based processing

### Migration Resources
- [Onboarding Guide](../onboarding/README.md) – Process for implementing Batch Mode
- [Batch Mode Implementation](./batch-mode.md#implementation-checklist) – What you need to do
- [Testing & Certification](../onboarding/05-testing-certification.md) – Validation requirements

### Technical Resources
- [API Reference](../../api/README.md) – API documentation (legacy reference)
- [Standards Guide](../../standards/README.md) – Data formats and requirements
- [Troubleshooting Guide](../../troubleshooting/README.md) – Common issues and solutions
- [FAQ](../../reference/faq.md) – Frequently asked questions

### Support Resources
- [Support Playbook](../../troubleshooting/README.md) – Troubleshooting guides
- [System Behavior](../../troubleshooting/system-behavior.md) – Understanding integration
- [Integration Architecture](../../troubleshooting/integration-architecture.md) – Technical overview

---

**Important Reminders:**

- **CIP Mode is legacy technology** - Use Batch Mode for new implementations
- **Migration is coming** - Existing users will be contacted with timeline
- **Full support continues** - Until migration is complete
- **Better solution available** - Batch Mode is simpler and more reliable

**Questions about CIP Mode or migration?** Contact your Technical Project Manager or re:Search support team.

---

**Last Updated:** November 2025  
**Status:** Legacy - Migration in Progress