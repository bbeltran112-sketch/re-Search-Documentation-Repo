# Batch Mode

## Quick Navigation
- **[How It Works](./how-it-works.md)** – Architecture and data flow
- **[Technical Setup](./technical-setup.md)** – S3/SFTP, schemas, validation
- **[Implementation Guide](./implementation-guide.md)** – Checklists and timeline
- **[Operations](./operations.md)** – Monitoring and troubleshooting

---

## What is Batch Mode?

Batch Mode is a **file-based integration method** that uses scheduled JSON snapshot deliveries to synchronize case, filing, party, and document metadata with re:Search. Unlike real-time API-driven modes, Batch Mode emphasizes predictability, high-volume throughput, and schema-driven data quality.

Think of it as a **data export pipeline**: your CMS generates files on a schedule (hourly, nightly, weekly), uploads them to secure storage (S3 or SFTP), and re:Search automatically processes them. No API development required—just file generation and upload automation.

**Key Characteristics:**
- Scheduled snapshot delivery via file uploads
- JSON Lines format for data files
- AWS S3 or SFTP transport
- Schema-driven validation
- Handles millions of records efficiently

---

## Is This Right for You?

### ✅ Use Batch Mode If:

- **You don't need real-time updates** – Hourly or nightly synchronization is acceptable
- **You handle high volumes** – Courts with 100-500+ filings/month
- **You lack API expertise** – Your CMS vendor is stronger with database exports than web services
- **You need historical backfills** – Loading decades of legacy data
- **You want predictable performance** – Scheduled processing windows work better than continuous API traffic
- **Your CMS has no ECF integration** – No existing EFM connection

### ❌ Don't Use Batch Mode If:

- **You need real-time updates** → Use [ECF Mode](../ecf-mode/README.md)
- **Your court has minimal filings** (< 50/month) → Use [Non-Integrated Mode](../non-integrated-mode.md)
- **Your CMS already integrates with EFM** → Use [ECF Mode](../ecf-mode/README.md)
- **You have budget for full API integration** → Consider [ECF Mode](../ecf-mode/README.md)

---

## At a Glance

| Attribute | Description |
|-----------|-------------|
| **Integration Pattern** | Scheduled snapshot delivery via file uploads |
| **Data Format** | JSON Lines (`.jsonl` / `.jsonl.gz`) |
| **Transport Method** | AWS S3 (preferred) or SFTP |
| **Update Frequency** | Configurable: hourly, nightly, weekly, or on-demand |
| **CMS Responsibility** | Export, package, validate, and upload snapshot files |
| **re:Search Responsibility** | Detect, validate, parse, and index uploaded files |
| **Typical Use Cases** | Historical backfills, bulk updates, non-ECF courts, high-volume ingestion |
| **Primary Advantage** | No API development required; handles millions of records efficiently |
| **Implementation Timeline** | 2-4 months typical |
| **Best For** | Courts with 100-500+ filings/month, bulk data needs |

---

## Key Benefits

### Low Technical Barrier
- **No API development required** – Avoids SOAP/REST complexity
- **Standard file formats** – Uses widely-supported JSON format
- **Flexible scheduling** – Run exports on your court's timeline

### High Performance
- **Scales to millions of records** – Proven with 10+ million record datasets
- **Parallel processing** – Multiple files processed simultaneously
- **Efficient resource usage** – Better for high-volume data transfer

### Data Quality Assurance
- **Schema-based validation** – Ensures data consistency before indexing
- **Checksum verification** – Detects corruption during transfer
- **Rollback capability** – Failed batches don't corrupt production data

### Operational Flexibility
- **Historical backfills supported** – Load decades of legacy data
- **Reprocessing capability** – Re-run batches if needed
- **Delta or full snapshots** – Choose incremental updates or complete snapshots
- **Works with any CMS** – Compatible with Odyssey, Tyler Eagle, custom systems

---

## Limitations and Considerations

| Limitation | Impact | Mitigation |
|------------|--------|------------|
| **Not Real-Time** | Updates delayed by batch schedule (hours to days) | Use ECF Mode if real-time updates are critical |
| **Full Export Overhead** | Large courts may require significant export time | Implement delta batches to send only changes |
| **Manifest Accuracy Critical** | Incorrect manifest causes batch rejection | Validate manifest before upload; use automation |
| **Schema Strictness** | Invalid JSON causes record rejection | Pre-validate files against schema before upload |
| **Document Binary Handling** | PDFs/images require separate delivery mechanism | Use S3 multipart uploads or coordinate with re:Search team |
| **Error Feedback Delay** | Validation errors discovered after upload | Monitor ingestion reports; implement pre-upload validation |
| **Storage Requirements** | Need space for file generation and staging | Plan for 2-3x data size for working space |
| **Network Dependency** | Upload requires reliable internet connection | Use compression; schedule during stable network periods |

---

## Strategic Context and Roadmap

### Current State (2025)

- Batch Mode is the **recommended standard** for new re:Search integrations
- Actively replacing CIP Mode for courts migrating from legacy systems
- Preferred over ECF Mode for courts without existing EFM integration
- Proven at scale with multiple large county implementations

### Adoption Timeline

- **CIP Mode migrations:** Expected completion by mid-2026
- **ECF Mode:** Reserved for courts with established EFM integrations
- **New implementations:** Default to Batch Mode unless real-time required
- **Historical migrations:** Ongoing through 2025-2026

### Future Enhancements

The re:Search team is developing:

- **Delta batch support** – Send only changed records for efficiency
- **Streaming validators** – Pre-upload validation tools to catch errors early
- **Enhanced diagnostics** – Real-time ingestion monitoring dashboard
- **Incremental indexing** – Faster processing for large batches
- **Automated scheduling** – Self-service batch scheduling and monitoring
- **Advanced error recovery** – Partial batch processing and automatic retry

---

## Quick Start Path

Ready to implement Batch Mode? Here's your roadmap:

1. **Understand the architecture** → Read [How It Works](./how-it-works.md)
2. **Review technical requirements** → Read [Technical Setup](./technical-setup.md)
3. **Plan your implementation** → Read [Implementation Guide](./implementation-guide.md)
4. **Contact your court liaison** → Schedule a kickoff meeting
5. **Request credentials** → Get S3 or SFTP access for test environment
6. **Begin development** → Follow the implementation checklist
7. **Test thoroughly** → Upload sample batches to test environment
8. **Go live** → Schedule production cutover and monitor

---

## Next Steps

### For Decision Makers
- Review this overview page
- Evaluate [Limitations and Considerations](#limitations-and-considerations)
- Compare with [ECF Mode](../ecf-mode/README.md) if real-time is needed
- Contact your court liaison to discuss feasibility

### For Technical Teams
- Read [How It Works](./how-it-works.md) to understand the architecture
- Review [Technical Setup](./technical-setup.md) for implementation details
- Check [Implementation Guide](./implementation-guide.md) for timeline and checklist
- Request schema documentation from re:Search team

### For Project Managers
- Review [Implementation Guide](./implementation-guide.md) for project planning
- Coordinate with CMS vendor on batch schedule and scope
- Schedule kickoff meeting with re:Search team
- Establish project milestones and success metrics

### For Support Teams
- Bookmark [Operations](./operations.md) for troubleshooting
- Review monitoring and success metrics
- Understand common issues and solutions
- Set up alerts and notification systems

---

## Related Documentation

### Other Integration Modes
- [Integration Modes Overview](../README.md) – Compare all integration methods
- [ECF Mode](../ecf-mode/README.md) – Real-time API-driven integration
- [CIP Mode](../cip-mode/README.md) – Court Integration Platform for legacy systems
- [Non-Integrated Mode](../non-integrated-mode.md) – Manual portal-based processing

### Onboarding Resources
- [Onboarding Guide](../../onboarding/README.md) – Implementation process
- [Before You Begin](../../onboarding/01-before-you-begin.md) – Prerequisites checklist
- [Environment Access](../../onboarding/02-environment-access.md) – Getting credentials
- [Testing & Certification](../../onboarding/05-testing-certification.md) – Validation requirements

### Technical Resources
- [API Reference](../../api/README.md) – For ECF/CIP mode API details (not used in Batch Mode)
- [Standards Guide](../../standards/README.md) – Data formats and requirements
- [Troubleshooting Guide](../../troubleshooting/README.md) – Common issues and solutions
- [FAQ](../../reference/faq.md) – Frequently asked questions

---

**Have Questions?**

Contact re:Search technical support for assistance with implementation.

**Last Updated:** November 2025