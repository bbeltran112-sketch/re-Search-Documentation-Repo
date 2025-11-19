# Client Onboarding

**Navigation:**  
[Home](./README.md) › [Client Documentation](./client-documentation/README.md) › Client Onboarding

---

## 📖 Quick Navigation

<table>
<tr>
<td width="50%" valign="top">

### 🚀 Getting Started
- [Overview](#overview)
- [Onboarding Journey](#onboarding-journey)
- [Timeline & Expectations](#timeline--expectations)
- [Onboarding Phases](#onboarding-phases)

</td>
<td width="50%" valign="top">

### 📚 Resources
- [Common Pitfalls](./common-pitfalls.md)
- [Frequently Asked Questions](./faq.md)
- [Downloadable Resources](./resources.md)
- [Support & Contact](#support--contact)

</td>
</tr>
</table>

---

## Overview

The client onboarding process guides **CMS vendors** and **courts** through all steps required to integrate with re:Search, from initial preparation through go-live and ongoing support.

This structured approach ensures:
- ✅ Clear expectations at each phase
- ✅ Proper technical preparation before development begins
- ✅ Comprehensive testing and validation
- ✅ Smooth production deployment
- ✅ Ongoing operational support

**Who should use this guide:**
- CMS vendors implementing re:Search integration
- Court IT teams preparing for re:Search deployment
- Project managers overseeing integration projects
- Technical architects planning system changes

---

## Onboarding Journey
```mermaid
graph LR
    A[1. Preparation] --> B[2. Planning]
    B --> C[3. Development]
    C --> D[4. Testing]
    D --> E[5. Go-Live]
    E --> F[6. Support]
    
    style A fill:#4CAF50,color:#fff,stroke:#2E7D32,stroke-width:2px
    style B fill:#2196F3,color:#fff,stroke:#1565C0,stroke-width:2px
    style C fill:#FF9800,color:#fff,stroke:#E65100,stroke-width:2px
    style D fill:#9C27B0,color:#fff,stroke:#6A1B9A,stroke-width:2px
    style E fill:#F44336,color:#fff,stroke:#C62828,stroke-width:2px
    style F fill:#607D8B,color:#fff,stroke:#37474F,stroke-width:2px
```

---

## Timeline & Expectations

<table>
<tr>
<td width="33%" align="center">
<h3>⏱️ Average Timeline</h3>
<strong>8-16 weeks</strong><br/>
Varies by integration mode
</td>
<td width="33%" align="center">
<h3>👥 Key Stakeholders</h3>
<strong>TPM, CMS Dev Team,<br/>Court IT, BIS Team</strong>
</td>
<td width="33%" align="center">
<h3>📋 Success Rate</h3>
<strong>95%+</strong><br/>
Most vendors certify within 2 attempts
</td>
</tr>
</table>

### Timeline by Integration Mode

| Integration Mode | Total Duration | Key Phases |
|------------------|----------------|------------|
| **Batch Mode** | 8-12 weeks | Prep: 2 weeks, Dev: 3-4 weeks, Testing: 2-3 weeks, Go-Live: 1-2 weeks |
| **ECF Mode** | 12-16 weeks | Prep: 3 weeks, Dev: 6-8 weeks, Testing: 3-4 weeks, Go-Live: 1-2 weeks |
| **CIP Mode** | 10-14 weeks | Prep: 2 weeks, Dev: 4-6 weeks, Testing: 3-4 weeks, Go-Live: 1-2 weeks |
| **Non-Integrated** | 2-4 weeks | Prep: 1 week, Setup: 1 week, Training: 1-2 weeks |

**Note:** Your Technical Project Manager will provide a customized timeline during kickoff.

---

## Onboarding Phases

### Phase 1: Preparation & Requirements
**Duration:** 1-3 weeks | **Goal:** Establish foundation and obtain access

- **[Before You Begin →](./client-documentation/onboarding/01-before-you-begin.md)** – Prerequisites and key concepts
- **[Environment Access →](./client-documentation/onboarding/02-environment-access.md)** – Credentials and endpoints
- **[CMS Preparation Checklist →](./client-documentation/onboarding/03-cms-preparation-checklist.md)** – Technical readiness assessment

**Key Deliverables:** TPM assigned, credentials obtained, stakeholders identified

---

### Phase 2: Integration Planning
**Duration:** 1 week | **Goal:** Select optimal integration mode

- **[Integration Mode Selection →](./client-documentation/onboarding/04-integration-mode-selection.md)** – Decision framework and comparison

**Key Deliverables:** Mode selected and approved, technical requirements confirmed

**Need help choosing?** Use our [Mode Selection Decision Tree](./client-documentation/integration-modes/selection-decision-tree.md)

---

### Phase 3: Technical Enablement & Development
**Duration:** 3-8 weeks | **Goal:** Implement integration and establish connectivity

- **[Technical Onboarding →](./client-documentation/onboarding/05-technical-onboarding.md)** – Implementation guidance and API specs

**Key Deliverables:** Code complete, connectivity established, error handling implemented

**Technical Resources:**
- [Batch Mode Guide](./client-documentation/integration-modes/batch-mode-overview.md)
- [ECF Mode Guide](./client-documentation/integration-modes/ecf-mode-overview.md)
- [API Reference](./technical-documentation/api-reference/README.md)

---

### Phase 4: Testing & Certification
**Duration:** 2-4 weeks | **Goal:** Validate data quality and integration behavior

- **[Testing & Certification →](./client-documentation/onboarding/06-testing-and-certification.md)** – Test scenarios and validation criteria

**Key Deliverables:** All tests passed, data quality validated, BIS certification approved

**Focus Areas:** Data accuracy, security handling, error recovery, performance

---

### Phase 5: Go-Live Preparation
**Duration:** 1-2 weeks | **Goal:** Final validation and production readiness

- **[Go-Live Readiness →](./client-documentation/onboarding/07-go-live-readiness.md)** – Production credentials and cutover planning

**Key Deliverables:** Production validated, cutover plan approved, rollback procedures defined

---

### Phase 6: Post-Launch Support
**Duration:** Ongoing | **Goal:** Ensure smooth operations

- **[Post-Go-Live Support →](./client-documentation/onboarding/08-post-go-live-support.md)** – Monitoring and troubleshooting

**Key Activities:** Daily monitoring, issue resolution, performance optimization

**Support Resources:**
- [Support Playbook](./technical-documentation/support-playbook/README.md)
- [Troubleshooting Guide](./technical-documentation/support-playbook/troubleshooting.md)

---

## Quick Links

**Planning & Preparation:**
- [Common Pitfalls to Avoid →](./client-documentation/onboarding/common-pitfalls.md)
- [Frequently Asked Questions →](./client-documentation/onboarding/faq.md)
- [Downloadable Resources →](./client-documentation/onboarding/resources.md)

**Integration Modes:**
- [Integration Modes Overview](./client-documentation/integration-modes/README.md)
- [Batch Mode](./client-documentation/integration-modes/batch-mode-overview.md)
- [ECF Mode](./client-documentation/integration-modes/ecf-mode-overview.md)
- [CIP Mode](./client-documentation/integration-modes/cip-mode-overview.md)

**Technical Documentation:**
- [API Reference](./technical-documentation/api-reference/README.md)
- [Support Playbook](./technical-documentation/support-playbook/README.md)
- [XML Library](./technical-documentation/xml-library/README.md)

---

## Support & Contact

### During Onboarding
Your **Technical Project Manager (TPM)** is your primary contact. They will:
- Provide customized timeline and milestones
- Coordinate access and credentials
- Review technical implementation
- Approve certification and go-live readiness

### Post Go-Live
**Internal Tyler Team:**  
BIS Team – Bryce Beltran  
📧 [BIS Team](mailto:brEFMInfo@tylertech.com)

**External Partners:**  
Contact your assigned TPM or submit requests through the Tyler Partner Portal

---

## Ready to Begin?

**Next Steps:**

1. **Read [Before You Begin](./client-documentation/onboarding/01-before-you-begin.md)** – Understand prerequisites
2. **Contact your TPM** – Schedule kickoff meeting
3. **Review [Common Pitfalls](./client-documentation/onboarding/common-pitfalls.md)** – Learn from others' experiences
4. **Request credentials** – Via [Environment Access](./client-documentation/onboarding/02-environment-access.md)
5. **Select integration mode** – Use [Mode Selection Guide](./client-documentation/integration-modes/04-integration-mode-selection.md)

**Questions?** Check our [FAQ](./client-documentation/onboarding/faq.md) or contact the BIS team.

---

[← Back to Client Documentation](./client-documentation/README.md) | [Before You Begin →](./client-documentation/onboarding/01-before-you-begin.md)