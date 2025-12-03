# Integration Guide

> **Breadcrumb:** [Home](../README.md) > Integration Guide

This guide walks you through the complete process of integrating your CMS with re:Search, from initial planning to production go-live.

---

## Getting Started

**New to re:Search?** Read the [Overview](../getting-started/overview.md) first to understand what re:Search is and how it works.

**Ready to integrate?** Follow this guide step-by-step.

---

## Integration Process Overview
```
1. Choose Integration Mode
   ↓
2. Complete Onboarding Steps
   ↓
3. Develop & Test
   ↓
4. Certify & Go Live
   ↓
5. Monitor & Support
```

**Typical Timeline:** 2-6 months depending on chosen mode and CMS complexity

---

## Step 1: Choose Your Integration Mode

Select the approach that matches your technical capabilities and requirements.

### [Integration Modes →](./integration-modes/README.md)

Compare all four modes and decide which fits your court:

- **[ECF Mode](./integration-modes/ecf-mode.md)** - Real-time API integration (3-6 months)
- **[CIP Mode](./integration-modes/cip-mode.md)** - Middleware-based integration (4-8 months)
- **[Batch Mode](./integration-modes/batch-mode.md)** - Scheduled file exports (2-4 months)
- **[Non-Integrated Mode](./integration-modes/non-integrated-mode.md)** - Manual portal entry (1-2 months)

**Decision Help:**
- [Mode Comparison Table](./integration-modes/README.md#mode-comparison)
- [Decision Tree](./integration-modes/README.md#decision-tree)

---

## Step 2: Complete Onboarding

Follow the structured onboarding process to prepare for integration.

### [Onboarding Process →](./onboarding/README.md)

**Phase-by-phase guide:**

| Phase | Duration | Documentation |
|-------|----------|---------------|
| **Before You Begin** | Week 1 | [Prerequisites & Planning](./onboarding/01-before-you-begin.md) |
| **Environment Access** | Week 1-2 | [Get Test Credentials](./onboarding/02-environment-access.md) |
| **CMS Preparation** | Week 2-3 | [Prepare Your System](./onboarding/03-cms-preparation-checklist.md) |
| **Technical Onboarding** | Week 3-12 | [Build Integration](./onboarding/05-technical-onboarding.md) |
| **Testing & Certification** | Week 10-14 | [Validate Implementation](./onboarding/06-testing-and-certification.md) |
| **Go-Live Readiness** | Week 14-16 | [Production Preparation](./onboarding/07-go-live-readiness.md) |
| **Post-Launch Support** | Ongoing | [Monitoring & Support](./onboarding/08-post-go-live-support.md) |

**Quick Reference:**
- [Common Pitfalls](./onboarding/common-pitfalls.md) - Avoid these mistakes
- [FAQ](./onboarding/faq.md) - Common questions answered
- [Resources](./onboarding/resources.md) - Tools and references

---

## Step 3: Understand Your APIs

Depending on your integration mode, you'll work with these APIs:

### Core APIs (ECF Modes)

**[NotifyCaseEvent](../api-reference/notifycaseevent/README.md)**  
Notify re:Search when case data changes
- Triggers re:Search to pull updated data
- Required for real-time synchronization
- [Event Types Reference](../api-reference/notifycaseevent/event-types.md)

**[GetCase](../api-reference/getcase/README.md)**  
Return case data when requested by re:Search
- Called in response to NotifyCaseEvent
- Must return current case state
- [Behavior Guide](../api-reference/getcase/behavior-guide.md)

**[GetDocument](../api-reference/getdocument/README.md)**  
Return document PDFs when requested
- Called when users view documents
- Must enforce security rules
- [Implementation Examples](../api-reference/getdocument/examples/)

**[NotifyDocketingComplete](../api-reference/notifydocketingcomplete/README.md)**  
Receive confirmation after indexing (optional)
- Callback from re:Search to CMS
- Confirms successful processing
- [Callback Patterns](../api-reference/notifydocketingcomplete/examples/)

### Batch Mode

**[Batch File Specification](./integration-modes/batch-mode.md#file-format)**  
XML file format for bulk case exports
- Complete case data in each file
- Scheduled upload to designated location
- [Schema & Examples](./integration-modes/batch-mode.md#examples)

### Non-Integrated Mode

**Portal Training Materials**  
Manual data entry guidance for court staff
- No API development required
- [Portal User Guide](./integration-modes/non-integrated-mode.md#portal-usage)

[Complete API Reference →](../api-reference/README.md)

---

## Step 4: Review Market Standards

Some states have additional requirements beyond core re:Search standards.

### [Market-Specific Standards →](../market-standards/README.md)

**Check your jurisdiction:**

| Market | Standards Body | Additional Requirements |
|--------|----------------|-------------------------|
| **Texas** | JCIT | [Texas Standards](../market-standards/texas/README.md) |
| **Illinois** | AOIC | [Illinois Standards](../market-standards/illinois/README.md) |
| **Other** | None | Core re:Search standards only |

**What's market-specific?**
- Allowed case types
- Required event types
- Security/access rules
- Data mapping requirements

**When to review:** Before beginning development to ensure compliance

---

## Step 5: Develop & Test

Build your integration following best practices.

### Development Resources

**Implementation Patterns:**
- [Architecture Overview](../implementation-playbook/architecture-overview.md)
- [Event Processing Logic](../implementation-playbook/event-type-logic.md)
- [Security Implementation](../implementation-playbook/security-logic.md)
- [Best Practices](../implementation-playbook/best-practices.md)

**Testing Guidance:**
- [Testing Requirements](./onboarding/06-testing-and-certification.md)
- [Test Case Examples](../implementation-playbook/real-world-examples.md)
- [Common Mistakes](./onboarding/common-pitfalls.md)

**Troubleshooting:**
- [Common Issues](../troubleshooting/common-integration-issues.md)
- [Case Type Problems](../troubleshooting/case-type-mismatch.md)
- [Security Issues](../troubleshooting/security-and-authentication.md)
- [Event Handling](../troubleshooting/event-handling.md)

---

## Step 6: Certify & Go Live

Complete certification requirements and prepare for production.

### Certification Process

**Requirements:**
- [ ] All test scenarios passed
- [ ] Security rules validated
- [ ] Performance benchmarks met
- [ ] Market standards compliance verified
- [ ] Documentation complete

[Certification Checklist →](./onboarding/06-testing-and-certification.md)

### Go-Live Preparation

**Pre-production steps:**
- [ ] Production credentials obtained
- [ ] Cutover plan documented
- [ ] Rollback procedures defined
- [ ] Staff training completed
- [ ] Monitoring configured

[Go-Live Readiness Guide →](./onboarding/07-go-live-readiness.md)

---

## Step 7: Post-Launch Support

Monitor your integration and address issues.

### Support Resources

**Day 1 Support:**
- [Monitoring Guidelines](./onboarding/08-post-go-live-support.md)
- [Escalation Process](../troubleshooting/escalation-process.md)
- [Performance Optimization](../troubleshooting/performance-optimization.md)

**Ongoing Maintenance:**
- [System Updates](./onboarding/08-post-go-live-support.md#updates)
- [Adding New Features](./onboarding/08-post-go-live-support.md#enhancements)
- [Contact Information](./onboarding/08-post-go-live-support.md#support-contacts)

---

## Quick Navigation by Role

### CMS Developer
1. [Choose Integration Mode](./integration-modes/README.md)
2. [Review API Reference](../api-reference/README.md)
3. [Check Market Standards](../market-standards/README.md)
4. [Follow Technical Onboarding](./onboarding/05-technical-onboarding.md)

### Court IT Manager
1. [Understand Integration Options](./integration-modes/README.md)
2. [Review Prerequisites](./onboarding/01-before-you-begin.md)
3. [Plan Timeline](./onboarding/README.md)
4. [Prepare for Go-Live](./onboarding/07-go-live-readiness.md)

### Project Manager
1. [Complete Onboarding Process](./onboarding/README.md)
2. [Track Certification Requirements](./onboarding/06-testing-and-certification.md)
3. [Manage Go-Live](./onboarding/07-go-live-readiness.md)
4. [Monitor Post-Launch](./onboarding/08-post-go-live-support.md)

---

## Common Questions

**How long does integration take?**  
2-6 months depending on mode choice and team experience. [See timelines →](./integration-modes/README.md#mode-comparison)

**What skills does my team need?**  
Varies by mode. [See requirements by mode →](./integration-modes/README.md)

**Can I switch modes later?**  
Yes, but requires development work. [Contact your TPM for guidance](./onboarding/08-post-go-live-support.md)

**What if I encounter issues?**  
Start with [Troubleshooting](../troubleshooting/README.md), then [escalate if needed](../troubleshooting/escalation-process.md)

**Do I need to support all event types?**  
Depends on your market. [Check market standards →](../market-standards/README.md)

[More FAQs →](./onboarding/faq.md)

---

## Additional Resources

**Implementation Help:**
- [Implementation Playbook](../implementation-playbook/README.md) - Deep technical guidance
- [Real-World Examples](../implementation-playbook/real-world-examples.md) - Common scenarios
- [Common Mistakes](../implementation-playbook/common-mistakes.md) - What to avoid

**API Details:**
- [Complete API Reference](../api-reference/README.md)
- [Authentication Guide](../api-reference/authentication.md)
- [Error Codes](../api-reference/error-codes.md)

**Support:**
- [Troubleshooting Guide](../troubleshooting/README.md)
- [Escalation Process](../troubleshooting/escalation-process.md)

---

**Questions?** Contact your assigned Tyler Technical Project Manager (TPM)