# ECF Mode (Electronic Court Filing)

## Quick Navigation
- **[How It Works](./how-it-works.md)** – Architecture and workflows
- **[Technical Setup](./technical-setup.md)** – APIs, security, infrastructure
- **[Implementation Guide](./implementation-guide.md)** – Timeline and requirements
- **[Operations](./operations.md)** – Monitoring and troubleshooting

---

## What is ECF Mode?

ECF Mode is a **real-time, event-driven integration** between your CMS and re:Search, designed specifically for courts that already use Tyler's Electronic Filing Manager (EFM) for electronic court filing.

Think of it as a **notification and retrieval system**: your CMS sends small event notifications when cases change, and re:Search responds by calling your CMS to retrieve the current case data. Changes appear in re:Search within seconds.

**Key Characteristics:**
- Event-driven: CMS pushes notifications when cases change
- Real-time: Changes appear in re:Search within seconds
- Pull-based data: re:Search calls GetCase to retrieve actual case data
- SOAP/XML: Uses ECF 4.x web service standards
- Requires: mTLS certificates, always-on connectivity

---

## Is This Right for You?

### ✅ Use ECF Mode If:

- **Your court already has EFM integration** – CMS is already connected to Tyler's Electronic Filing Manager
- **You need real-time synchronization** – Cases must appear in re:Search immediately after filing
- **Your CMS supports ECF standards** – Native SOAP/XML web service capabilities
- **You process significant e-filing volume** – 500+ e-filings per month with high automation needs
- **You have web services expertise** – Development team experienced with SOAP integration

### ❌ Don't Use ECF Mode If:

- **You don't have EFM integration** → Use [Batch Mode](../batch-mode/README.md) - simpler, file-based integration
- **You want simpler implementation** → Use [Batch Mode](../batch-mode/README.md) - easier to implement and maintain
- **Your CMS is legacy/custom-built** without web services → Use [CIP Mode](../cip-mode/README.md) - Court Integration Platform handles the web services
- **You need temporary solution** → Use [Non-Integrated Mode](../non-integrated-mode.md) - manual data entry
- **Real-time isn't critical** → Use [Batch Mode](../batch-mode/README.md) - scheduled processing is simpler

---

## At a Glance

| Attribute | Description |
|-----------|-------------|
| **Integration Pattern** | Real-time, event-driven SOAP web services |
| **Data Flow** | Push notifications + Pull data (GetCase) |
| **Update Frequency** | Immediate (seconds after change) |
| **Protocol** | SOAP/XML (ECF 4.x or 5.x standards) |
| **CMS Responsibility** | Send event notifications, respond to GetCase/GetDocument |
| **re:Search Responsibility** | Call GetCase when events received, index data |
| **Typical Use Cases** | Courts with EFM integration, high e-filing volume, real-time needs |
| **Primary Advantage** | Real-time synchronization with immediate updates |
| **Implementation Timeline** | 14-20 weeks typical |
| **Best For** | Courts with 500+ e-filings/month, existing EFM integration |

---

## How It Works (Quick Overview)

ECF Mode operates through two workflows:

### E-Filing Workflow (via EFM)
```
Attorney files → EFM → CMS (RecordFiling) → CMS dockets
                 ↓
           Notify re:Search → re:Search calls GetCase → Case indexed
```

### Direct Case Updates
```
CMS change happens → CMS sends NotifyCaseEvent → re:Search calls GetCase → Case updated
```

**The key concept:** Your CMS sends lightweight notifications (just case ID + event type), and re:Search responds by calling GetCase to retrieve the actual current data.

[Learn more about the workflows →](./how-it-works.md)

---

## Key Benefits

### Real-Time Synchronization
- Changes appear in re:Search within seconds
- No delay between filing and public visibility
- Security changes take effect immediately

### Leverages Existing Infrastructure
- Uses your existing EFM integration
- Reuses web service infrastructure
- Minimal additional development if EFM already implemented

### Event-Driven Efficiency
- Only updates when changes occur
- No polling or scheduled processing needed
- Efficient use of system resources

### Granular Control
- Specific event types for different change scenarios
- Fine-grained security controls
- Flexible document delivery options

---

## Key Limitations

| Limitation | Impact | Mitigation |
|------------|--------|------------|
| **Complex Implementation** | SOAP/XML expertise required, longer timeline | Use Batch Mode if complexity is a concern |
| **Always-On Requirement** | 99.9% uptime needed for real-time updates | Implement robust monitoring and failover |
| **Certificate Management** | mTLS certificates expire every 12-18 months | Set up expiration monitoring and renewal process |
| **Performance Requirements** | GetCase must respond in < 3 seconds | Optimize database queries, avoid N+1 problems |
| **Network Dependency** | Requires stable connectivity to Tyler endpoints | Plan for retry logic and temporary outages |
| **Higher Operational Overhead** | More monitoring, logging, and maintenance | Budget for ongoing operations support |

---

## Prerequisites

Before starting ECF Mode implementation:

### Technical Prerequisites
- [ ] CMS has SOAP/XML web services capability
- [ ] CMS already integrated with Tyler EFM (or integration underway)
- [ ] Development team experienced with web services
- [ ] Network connectivity to Tyler endpoints available
- [ ] CMS can respond to GetCase within 3 seconds

### Operational Prerequisites
- [ ] Court processes 500+ e-filings per month (or expects to)
- [ ] Real-time synchronization is a business requirement
- [ ] Resources available for 14-20 week implementation
- [ ] Operations team can monitor and maintain integration

### Infrastructure Prerequisites
- [ ] Publicly accessible HTTPS endpoint for GetCase
- [ ] Firewall allows outbound HTTPS to Tyler endpoints
- [ ] Static IP or domain name for CMS endpoint
- [ ] Certificate management process in place

---

## Strategic Context

### Current State (2025)

- ECF Mode is **recommended for courts with existing EFM integration**
- Used by courts processing high e-filing volumes
- Required for real-time filing visibility
- Established standard with proven implementations

### When to Choose ECF Over Batch

**Choose ECF Mode when:**
- You already have EFM integration (infrastructure exists)
- Real-time updates are critical to operations
- E-filing volume justifies the complexity
- Development resources support SOAP integration

**Choose Batch Mode when:**
- You don't have EFM integration
- Hourly/nightly updates are acceptable
- Simpler implementation is preferred
- Development team lacks SOAP expertise

### Future Considerations

**ECF 5.x Available:**
- Improved error handling
- Better performance characteristics
- Enhanced security features
- More flexible event types
- Contact your CMS vendor for upgrade path

---

## Quick Start Path

Ready to implement ECF Mode? Here's your roadmap:

1. **Verify prerequisites** → Review checklist above
2. **Understand the workflows** → Read [How It Works](./how-it-works.md)
3. **Review technical requirements** → Read [Technical Setup](./technical-setup.md)
4. **Plan your implementation** → Read [Implementation Guide](./implementation-guide.md)
5. **Contact your Tyler TPM** → Schedule kickoff meeting
6. **Request certificates** → Get test environment credentials
7. **Begin development** → Follow implementation checklist
8. **Test and certify** → Work with Tyler team for certification
9. **Deploy to production** → Go-live with monitoring

---

## Next Steps

### For Decision Makers
- Review this overview page
- Compare with [Batch Mode](../batch-mode/README.md) if EFM integration doesn't exist
- Evaluate [Prerequisites](#prerequisites) against current capabilities
- Contact your Tyler Technical Project Manager to discuss

### For Technical Teams
- Read [How It Works](./how-it-works.md) to understand the architecture
- Review [Technical Setup](./technical-setup.md) for API and security details
- Check [Implementation Guide](./implementation-guide.md) for timeline and resources
- Request ECF standards documentation from Tyler

### For Project Managers
- Review [Implementation Guide](./implementation-guide.md) for project planning
- Assess resource requirements (14-20 weeks, dedicated team)
- Coordinate with Tyler TPM on timeline and milestones
- Plan for certification testing phase

### For Operations Teams
- Bookmark [Operations](./operations.md) for monitoring and troubleshooting
- Review key metrics and alerting requirements
- Understand certificate management lifecycle
- Set up logging and monitoring infrastructure

---

## Migration Options

### Moving from ECF to Batch Mode

If you're currently using ECF Mode but want to simplify operations:

**Why migrate to Batch Mode?**
- Simpler file-based integration (no SOAP/XML)
- No certificates to manage
- Scheduled processing instead of real-time
- Lower operational overhead

**Migration timeline:** 6-8 weeks with re:Search support

[Learn more about Batch Mode →](../batch-mode/README.md)

### Upgrading to ECF 5.x

If you're on ECF 4.x and want improved features:

**Benefits of ECF 5.x:**
- Improved error handling
- Better performance
- Enhanced security
- More flexible event types

Contact your CMS vendor for upgrade path.

---

## Related Documentation

### Other Integration Modes
- [Integration Modes Overview](../README.md) – Compare all integration methods
- [Batch Mode](../batch-mode/README.md) – Simpler file-based integration
- [CIP Mode](../cip-mode/README.md) – Court Integration Platform for legacy systems
- [Non-Integrated Mode](../non-integrated-mode.md) – Manual portal-based processing

### API Reference
- [NotifyCaseEvent API](../../api-reference/notifycaseevent.md) – Event notification specification
- [GetCase API](../../api-reference/getcase.md) – Case retrieval specification
- [GetDocument API](../../api-reference/getdocument.md) – Document retrieval specification

### Implementation Resources
- [Event Type Logic](../../implementation-playbook/event-type-logic.md) – Understanding event types
- [Security Logic](../../implementation-playbook/security-logic.md) – Security enforcement
- [Best Practices](../best-practices.md) – Implementation guidance
- [Troubleshooting Guide](../../troubleshooting/README.md) – Common issues

### Onboarding
- [Onboarding Guide](../../onboarding/README.md) – Implementation process
- [Before You Begin](../../onboarding/01-before-you-begin.md) – Prerequisites checklist
- [Testing & Certification](../../onboarding/05-testing-certification.md) – Certification process

---

**Have Questions?**

Contact your Tyler Technical Project Manager for assistance with ECF Mode implementation.

**Last Updated:** December 2025