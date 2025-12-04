# Integration Modes

> **Breadcrumb:** [Home](../../README.md) > [Integration Guide](../README.md) > Integration Modes

Choose the integration mode that fits your court's technical capabilities and operational requirements.

## Quick Navigation
- [Quick Decision Guide](#quick-decision-guide) - Answer 5 questions to find your mode
- [Mode Comparison](#mode-comparison) - Side-by-side comparison table
- [ECF Mode](#-ecf-mode---real-time-api-integration) - Real-time SOAP integration
- [CIP Mode](#-cip-mode---middleware-integration-platform) - Enterprise Justice middleware
- [Batch Mode](#-batch-mode---scheduled-file-exports) - File-based integration
- [Non-Integrated Mode](#-non-integrated-mode---manual-portal-entry) - Manual entry
- [Choosing the Right Mode](#choosing-the-right-mode) - Decision flowchart and scenarios
- [Migration Paths](#migration-paths) - Switching between modes
- [Getting Started](#getting-started) - Next steps
- [FAQ](#frequently-asked-questions)

---

## Quick Decision Guide

Not sure which mode to use? Answer these questions:

### 1. Do you need real-time updates?
- **Yes** → ECF Mode or CIP Mode
- **No, hourly/nightly is fine** → Batch Mode
- **No, we're very small** → Non-Integrated Mode

### 2. Is your CMS already integrated with Tyler's EFM?
- **Yes** → [ECF Mode](./ecf-mode/) (leverages existing infrastructure)
- **No** → Continue to question 3

### 3. Do you use Tyler Enterprise Justice?
- **Yes** → [CIP Mode](./cip-mode/) (EJ has built-in integration)
- **No** → Continue to question 4

### 4. Can your CMS provide SOAP web services?
- **Yes** → [ECF Mode](./ecf-mode/) (if you want real-time)
- **No** → Continue to question 5

### 5. Can your CMS generate scheduled file exports?
- **Yes** → [Batch Mode](./batch-mode/) (simplest automated option)
- **No** → [Non-Integrated Mode](./non-integrated-mode/) (manual entry)

---

## Mode Comparison

| Attribute | ECF Mode | CIP Mode | Batch Mode | Non-Integrated |
|-----------|----------|----------|------------|----------------|
| **Update Frequency** | Real-time (seconds) | Real-time (seconds) | Scheduled (hours/days) | Manual entry |
| **CMS Development** | High | Medium | Low | None |
| **Technical Complexity** | High (SOAP APIs) | Medium (CIP config) | Low (file exports) | None |
| **Timeline** | 14-20 weeks | 4-8 months | 2-4 months | 2-4 weeks |
| **Prerequisites** | EFM integration | Enterprise Justice | File export capability | Staff training |
| **Typical Court Size** | 500+ e-filings/month | Any (if using EJ) | 100-500 filings/month | < 50 filings/month |
| **Maintenance** | High (certs, monitoring) | Medium (CIP updates) | Low (batch monitoring) | Minimal |
| **Recommended For** | Courts with EFM | EJ courts | Most courts | Very small courts |

---

## Mode Overview

### 🔄 ECF Mode - Real-Time API Integration
[View complete documentation →](./ecf-mode/)

**What it is:** Event-driven SOAP web service integration between your CMS and re:Search.

**How it works:**
- CMS sends event notifications when cases change
- re:Search calls GetCase to retrieve current data
- Changes appear in re:Search within seconds

**Best for:**
- Courts already using Tyler EFM for e-filing
- High e-filing volume (500+ filings/month)
- Real-time synchronization required
- CMS supports SOAP/XML web services

**Key requirements:**
- NotifyCaseEvent, GetCase, GetDocument APIs
- mTLS certificates
- Always-on connectivity
- SOAP/XML expertise

**Timeline:** 14-20 weeks

[Read ECF Mode details →](./ecf-mode/)

---

### 🏢 CIP Mode - Middleware Integration Platform
[View complete documentation →](./cip-mode/)

**What it is:** Court Integration Platform (CIP) acts as middleware between your CMS and re:Search.

**How it works:**
- CIP polls your CMS database for changes
- CIP transforms data and calls re:Search APIs
- Minimal CMS development required

**Best for:**
- Courts using Tyler Enterprise Justice
- CMS lacks web service capability
- Want real-time updates without building APIs
- Prefer Tyler-managed integration layer

**Key requirements:**
- Tyler Enterprise Justice deployment
- CIP middleware installation
- Database access from CIP to CMS
- Configuration and mapping

**Timeline:** 4-8 months (includes CIP setup)

[Read CIP Mode details →](./cip-mode/)

---

### 📦 Batch Mode - Scheduled File Exports
[View complete documentation →](./batch-mode/)

**What it is:** File-based integration using scheduled JSON snapshot deliveries.

**How it works:**
- CMS generates JSON Lines files (cases, filings, parties, documents)
- Files uploaded to S3 or SFTP on schedule (hourly, nightly, weekly)
- re:Search automatically processes and indexes files

**Best for:**
- Courts without EFM integration
- Prefer simplicity over real-time
- High-volume bulk ingestion needs
- Hourly or nightly updates acceptable

**Key requirements:**
- JSON Lines file generation capability
- AWS S3 or SFTP access
- File compression and checksum generation
- Scheduled upload automation

**Timeline:** 2-4 months

[Read Batch Mode details →](./batch-mode/)

---

### 📝 Non-Integrated Mode - Manual Portal Entry
[View complete documentation →](./non-integrated-mode/)

**What it is:** Court staff manually enter case data through re:Search web portal.

**How it works:**
- Staff log into re:Search portal
- Enter case metadata manually (no documents)
- Public can search entered cases

**Best for:**
- Very small courts (< 50 filings/month)
- Temporary solution during transition
- Pilot programs or evaluation periods
- Courts lacking technical resources

**Key requirements:**
- Staff training on portal
- User accounts and permissions
- Data entry procedures

**Timeline:** 2-4 weeks (mostly training)

[Read Non-Integrated Mode details →](./non-integrated-mode/)

---

## Choosing the Right Mode

### Decision Flowchart

```
Do you need real-time updates?
├─ YES → Do you have EFM integration?
│         ├─ YES → ECF Mode ⭐
│         └─ NO → Do you use Enterprise Justice?
│                  ├─ YES → CIP Mode
│                  └─ NO → Can you build SOAP APIs?
│                           ├─ YES → ECF Mode
│                           └─ NO → Use Batch Mode
│
└─ NO → How many filings per month?
         ├─ 100+ → Batch Mode ⭐ (recommended)
         ├─ 50-100 → Batch Mode or Non-Integrated
         └─ < 50 → Non-Integrated Mode
```

### Common Scenarios

**Scenario: Large court with EFM integration**
→ **ECF Mode** - You already have the infrastructure

**Scenario: Court using Enterprise Justice**
→ **CIP Mode** - Built-in integration capability

**Scenario: Medium-sized court, no EFM, wants automation**
→ **Batch Mode** - Best balance of simplicity and automation

**Scenario: Small rural court, minimal filings**
→ **Non-Integrated Mode** - Simple manual entry sufficient

**Scenario: Need quick temporary solution**
→ **Non-Integrated Mode** - Fast setup while planning full integration

---

## Migration Paths

### Moving Between Modes

**From Non-Integrated → Batch Mode:**
- Common upgrade path as court grows
- Timeline: 2-3 months
- Keeps manual data during transition

**From Batch Mode → ECF Mode:**
- Upgrade when real-time becomes requirement
- Timeline: 3-4 months
- Can run both during transition

**From ECF Mode → Batch Mode:**
- Simplification if real-time not needed
- Timeline: 6-8 weeks
- Reduces operational complexity

**From CIP Mode → ECF Mode:**
- Uncommon (usually stay with CIP)
- Only if removing Enterprise Justice
- Timeline: 4-6 months

---

## Getting Started

### 1. Review Mode Documentation
Start with the README for your selected mode:
- [ECF Mode README](./ecf-mode/) - Real-time integration
- [CIP Mode README](./cip-mode/) - Middleware integration  
- [Batch Mode README](./batch-mode/) - File-based integration
- [Non-Integrated Mode README](./non-integrated-mode/) - Manual entry

### 2. Contact Tyler
Reach out to your **Tyler Technical Project Manager (TPM)** to:
- Confirm mode selection
- Schedule kickoff meeting
- Review prerequisites
- Obtain credentials

### 3. Begin Implementation
Follow the implementation guide for your chosen mode:
- [ECF Implementation Guide](./ecf-mode/implementation-guide.md)
- [CIP Implementation Guide](./cip-mode/implementation-guide.md)
- [Batch Implementation Guide](./batch-mode/implementation-guide.md)
- [Non-Integrated Setup Guide](./non-integrated-mode/setup-guide.md)

---

## Frequently Asked Questions

**Q: Can we use multiple modes simultaneously?**
A: Not recommended. Choose one primary mode. During migration, you might temporarily run two modes in parallel.

**Q: Can we switch modes later?**
A: Yes. See [Migration Paths](#migration-paths) above. Most common is starting with Non-Integrated or Batch, then upgrading to ECF if needed.

**Q: What if our court is too small for integration?**
A: Non-Integrated Mode is designed for very small courts. Manual entry is perfectly acceptable for courts with minimal filings.

**Q: Do we have to choose now?**
A: You can start with Non-Integrated Mode while planning a more automated integration. This gives you time to assess needs and build resources.

**Q: What's the most common mode?**
A: **Batch Mode** is becoming the standard recommendation for new integrations. It balances simplicity with automation.

**Q: What's the most complex mode?**
A: **ECF Mode** requires SOAP expertise, certificate management, and real-time monitoring. Only choose if you need real-time updates or already have EFM.

**Q: Which mode is fastest to implement?**
A: **Non-Integrated Mode** can be operational in 2-4 weeks (just training). For automated integration, **Batch Mode** is fastest (2-4 months).

---

## Need Help?

**Still unsure which mode to choose?**
- Review the [Decision Flowchart](#decision-flowchart)
- Compare your situation to [Common Scenarios](#common-scenarios)
- Contact your Tyler Technical Project Manager
- Schedule a consultation call

**Ready to get started?**
- Pick your mode and click through to its README
- Review the implementation guide
- Contact Tyler TPM to begin onboarding

---

## Related Documentation

- [Onboarding Guide](../onboarding/README.md) - Implementation process for all modes
- [API Reference](../../api-reference/README.md) - Technical specifications (ECF/CIP modes)
- [Best Practices](../best-practices.md) - Implementation guidance
- [Troubleshooting](../../troubleshooting/README.md) - Common issues and solutions

---

**Last Updated:** December 2025