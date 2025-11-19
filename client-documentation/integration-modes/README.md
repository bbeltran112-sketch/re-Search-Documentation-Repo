# Integration Modes Overview

**Navigation:**  
[Home](/README.md) › [Client Documentation](./client-documentation/README.md) › Integration Modes

---
## 📖 Quick Navigation

<table>
<tr>
<td width="50%" valign="top">

### 🎯 Planning
- [Overview](#overview)
- [Quick Decision Guide](#quick-decision-guide)
- [Mode Selection Criteria](#mode-selection-criteria)
- [Can I Use Multiple Modes?](#can-i-use-multiple-modes)

### 📋 Mode Comparison
- [Modes Comparison Table](#integration-modes-comparison)
- [ECF Mode](#ecf-mode)
- [CIP Mode](#cip-mode)
- [Batch Mode](#batch-mode)
- [Non-Integrated Mode](#non-integrated-mode)

</td>
<td width="50%" valign="top">

### 🔍 Mode Details
- [Mode Details Section](#mode-details)
- [ECF Mode Overview](#ecf-mode)
- [CIP Mode Overview](#cip-mode)
- [Batch Mode Overview](#batch-mode)
- [Non-Integrated Mode Overview](#non-integrated-mode)

### 📚 Resources
- [Related Documentation](#-related-documentation)
- [Common Questions](#common-questions)
- [Need Help Deciding?](#need-help-deciding)
- [Next Steps](#️next-steps)

</td>
</tr>
</table>

## Overview

Integration modes define **how case, filing, and security data enter re:Search**. Choosing the right mode ensures accurate indexing, proper visibility handling, and predictable synchronization behavior across your deployment.

This page provides a comprehensive comparison of all supported integration modes, helping you understand which approach best fits your court's technical capabilities and operational requirements.

---

## Quick Decision Guide

**Not sure which mode you need?** Answer these questions:

1. **Does your court use an e-filing management system (EFM)?**
   - ✅ Yes → Consider **ECF Mode**
   - ❌ No → Continue to question 2

2. **Can your CMS send real-time event notifications?**
   - ✅ Yes → **ECF Mode** is ideal
   - ❌ No → Continue to question 3

3. **Are you currently using Enterprise Justice (EJ)?**
   - ✅ Yes → **CIP Mode** is your best path
   - ❌ No → Continue to question 4

4. **Can your CMS export data files on a schedule?**
   - ✅ Yes → **Batch Mode** will work
   - ❌ No → **Non-Integrated Mode** may be required

**Still unsure?** Use our [Mode Selection Decision Tree](./client-documentation/integration-modes/selection-decision-tree.md) for detailed guidance.

---

## Integration Modes Comparison

| Mode | Data Flow | Latency | Technical Requirements | Best For |
|------|-----------|---------|------------------------|----------|
| **[ECF Mode](./client-documentation/integration-modes/ecf-mode-overview.md)** | Real-time events via SOAP APIs | Immediate | ECF 4.x support, NotifyCaseEvent, GetCase/GetDocument | Modern courts with EFM integration |
| **[CIP Mode](./client-documentation/integration-modes/cip-mode-overview.md)** | REST API publishing from EJ | Near real-time | EJ platform, REST endpoints | Legacy EJ courts in transition |
| **[Batch Mode](./client-documentation/integration-modes/batch-mode-overview.md)** | Scheduled JSONL file uploads | Hours to daily | File export capability, S3/SFTP access | Bulk ingestion, historical migration |
| **[Non-Integrated Mode](./client-documentation/integration-modes/non-integrated-mode-overview.md)** | Manual or BIS-assisted | Variable | Minimal - web browser access | Small courts, no CMS integration |

---

## Mode Details

### ECF Mode
**Real-time, event-driven integration**

ECF Mode leverages the Electronic Court Filing (ECF) 4.x standard to enable real-time synchronization between your CMS, EFM, and re:Search. When case events occur (filings, dispositions, party changes), your CMS sends `NotifyCaseEvent` messages that trigger re:Search to pull updated case data via `GetCase` and `GetDocument` operations.

**Key Characteristics:**
- ⚡ Immediate updates (sub-second to minutes)
- 🔄 Bidirectional communication
- 🔒 Full security and sealed case support
- 📊 Event-type driven logic for precise updates

**[Read Full ECF Mode Documentation →](./client-documentation/integration-modes/ecf-mode-overview.md)**

---

### CIP Mode
**REST-based publishing for legacy systems**

CIP (Court Integration Platform) Mode provides a REST API endpoint for courts running Enterprise Justice to publish case data directly to re:Search. This mode was designed to bridge legacy EJ deployments with modern re:Search capabilities without requiring ECF implementation.

**Key Characteristics:**
- 🔌 Simple REST POST operations
- 📦 Full case payload publishing
- 🕒 Near real-time updates
- 🔄 Transition path from EJ to modern CMS

**[Read Full CIP Mode Documentation →](./client-documentation/integration-modes/cip-mode-overview.md)**

---

### Batch Mode
**Scheduled bulk data ingestion**

Batch Mode enables courts to export case data as JSONL files and upload them to S3 or SFTP locations for processing. re:Search ingests these files on a scheduled basis, making it ideal for historical migrations, bulk updates, or courts where real-time integration isn't feasible.

**Key Characteristics:**
- 📁 JSONL file format
- ☁️ S3 or SFTP delivery
- 📅 Scheduled processing (hourly, daily, etc.)
- 🔄 High-volume capable

**[Read Full Batch Mode Documentation →](./client-documentation/integration-modes/batch-mode-overview.md)**

---

### Non-Integrated Mode
**Manual or BIS-assisted publishing**

Non-Integrated Mode supports courts that lack CMS integration capabilities. Case data is published through BIS team tools, manual web interfaces, or assisted processes. While not automated, this mode ensures all courts can participate in re:Search regardless of technical infrastructure.

**Key Characteristics:**
- 👤 Manual or assisted workflows
- 🛠️ BIS team tooling support
- 📝 Web-based data entry options
- 🏛️ Suitable for small courts

**[Read Full Non-Integrated Mode Documentation →](./client-documentation/integration-modes/non-integrated-mode-overview.md)**

---

## Mode Selection Criteria

### Choose **ECF Mode** if:
- ✅ Your CMS supports ECF 4.x standards
- ✅ You have an active EFM integration
- ✅ Real-time updates are required
- ✅ Your technical team can implement SOAP web services
- ✅ You need comprehensive security/sealed case handling

### Choose **CIP Mode** if:
- ✅ You're currently running Enterprise Justice
- ✅ REST APIs are easier than SOAP for your team
- ✅ You're planning to migrate CMS platforms eventually
- ✅ Near real-time updates are acceptable

### Choose **Batch Mode** if:
- ✅ Your CMS can export data files but not send events
- ✅ Scheduled updates (not real-time) meet your needs
- ✅ You're performing historical data migration
- ✅ You need high-volume ingestion capability

### Choose **Non-Integrated Mode** if:
- ✅ Your court has no CMS or limited technical resources
- ✅ Case volume is low enough for manual processing
- ✅ BIS team assistance is acceptable
- ✅ Immediate automation isn't critical

---

## Can I Use Multiple Modes?

Yes! Many courts use **hybrid approaches**:

- **ECF + Batch**: Real-time for new data, batch for historical backfill
- **CIP + Batch**: Transition period from EJ while migrating historical cases
- **ECF + Non-Integrated**: Automated for most courts, manual for outliers

Discuss hybrid scenarios with your Technical Project Manager during planning.

---

## Related Documentation

| Resource | Purpose | Link |
|----------|---------|------|
| **Mode Selection Decision Tree** | Interactive guide to choose your mode | [View Tree →](./client-documentation/integration-modes/selection-decision-tree.md) |
| **API Reference** | Technical specifications for ECF/CIP APIs | [View APIs →](./technical-documentation/api-reference/README.md) |
| **Client Onboarding** | Step-by-step integration process | [View Guide →](./client-documentation/integration-modes/onboarding/README.md) |
| **Support Playbook** | Troubleshooting and system behavior | [View Playbook →](./technical-documentation/support-playbook/README.md) |

---

## Common Questions

**Q: Can I switch modes after go-live?**  
A: Yes, but it requires coordination with the BIS team. Mode changes typically happen during planned maintenance windows.

**Q: How long does each mode take to implement?**  
A: 
- ECF Mode: 8-12 weeks (most complex)
- CIP Mode: 4-6 weeks
- Batch Mode: 2-4 weeks
- Non-Integrated: 1-2 weeks (minimal technical work)

**Q: Which mode provides the best data quality?**  
A: ECF Mode generally provides the highest quality due to real-time validation and bidirectional communication, but all modes can achieve excellent results with proper configuration.

**Q: What if my CMS vendor doesn't support any of these modes?**  
A: Contact your TPM. The BIS team can work with vendors on implementation or explore custom solutions.

---

## Need Help Deciding?

**For Integration Questions:**  
Contact your assigned Technical Project Manager

**Internal Tyler Teams:**  
BIS Team – Bryce Beltran  
📧 [BIS Team](mailto:EFMInfo@tylertech.com)

---

## Next Steps

Ready to move forward? Here's your path:

1. **[Use the Decision Tree](./client-documentation/integration-modes/selection-decision-tree.md)** to confirm your mode selection
2. **Read the detailed mode documentation** for your chosen approach
3. **Review [Client Onboarding](./client-documentation/integration-modes/onboarding/README.md)** to understand the implementation process
4. **Contact your TPM** to schedule a kickoff meeting

---

[← Back to Client Documentation](./client-documentation/README.md) | [Mode Selection Decision Tree →](./client-documentation/integration-modes/selection-decision-tree.md)