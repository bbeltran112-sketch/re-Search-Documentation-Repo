# Implementation Guide

[← Back to Batch Mode Overview](./README.md)

## Quick Navigation
- [Implementation Timeline](#implementation-timeline)
- [CMS Vendor Checklist](#cms-vendor-checklist)
- [Court Checklist](#court-checklist)
- [Testing Requirements](#testing-requirements)
- [Go-Live Process](#go-live-process)

---

## Implementation Timeline

### Typical Timeline: 2-4 Months

```
Month 1: Planning & Setup
├─ Week 1-2: Kickoff, credentials, schema review
├─ Week 3-4: Development environment setup
└─ Deliverable: Test environment configured, sample files generated

Month 2: Development
├─ Week 1-2: Export logic development
├─ Week 3: Manifest generation and upload automation
└─ Week 4: Initial test batch submission
    Deliverable: First successful test batch processed

Month 3: Testing & Refinement
├─ Week 1-2: Schema compliance testing
├─ Week 3: Load testing with production-sized data
└─ Week 4: Error handling and edge case testing
    Deliverable: Certification testing passed

Month 4: Production Deployment
├─ Week 1: Production credentials and final configuration
├─ Week 2: Historical backfill (if applicable)
├─ Week 3: Parallel run (shadow mode)
└─ Week 4: Go-live and monitoring
    Deliverable: Production batch running successfully
```

### Timeline Factors

**Faster Implementation (6-8 weeks):**
- CMS has existing batch export capabilities
- Small court with simple data model
- No historical backfill required
- Experienced vendor team

**Slower Implementation (4-6 months):**
- Custom CMS with no export framework
- Large court with complex data (1M+ cases)
- Extensive historical backfill (decades of data)
- Multiple court jurisdictions in scope
- Complex data transformations needed

---

## CMS Vendor Checklist

### Phase 1: Planning and Setup

- [ ] **Schedule kickoff meeting** with re:Search team and court
  - Review project scope and timeline
  - Identify key stakeholders and roles
  - Establish communication channels

- [ ] **Obtain test environment credentials**
  - Request S3 IAM keys or SFTP SSH keys
  - Verify credential access and permissions
  - Test connectivity to storage location

- [ ] **Review schema documentation**
  - Request complete JSON schema specifications
  - Understand required vs. optional fields
  - Clarify court-specific requirements
  - Document any custom fields or extensions

- [ ] **Assess CMS capabilities**
  - Identify data sources (database tables, views)
  - Map CMS fields to schema fields
  - Document data transformation requirements
  - Identify any data quality issues

### Phase 2: Development

- [ ] **Develop export logic**
  - Create SQL queries or API calls to extract data
  - Implement incremental vs. full snapshot logic
  - Apply court-specific filtering (jurisdiction, case types)
  - Handle security classifications correctly
  - Optimize query performance for large datasets

- [ ] **Implement JSON Lines writer**
  - Generate one JSON object per line
  - Ensure schema compliance (required fields, data types)
  - Handle character encoding (UTF-8)
  - Escape special characters properly
  - Split large datasets into multiple files (50K records recommended)

- [ ] **Build manifest generator**
  - Calculate SHA-256 checksums for each file
  - Collect file metadata (size, record count)
  - Generate unique batchId
  - Include all required manifest fields
  - Output valid JSON format

- [ ] **Implement file compression**
  - Use gzip compression (`.jsonl.gz`)
  - Verify compressed files are readable
  - Test decompression locally

- [ ] **Build upload automation**
  - Implement S3 upload (AWS SDK) or SFTP upload
  - Add retry logic for network failures
  - Verify upload success (check return codes)
  - Upload manifest last to signal batch ready
  - Log upload status and timing

- [ ] **Create pre-upload validation**
  - Validate JSON syntax locally
  - Check required fields presence
  - Verify date formats and enum values
  - Test checksum calculation
  - Catch errors before upload

- [ ] **Set up scheduling**
  - Configure cron job, task scheduler, or orchestration tool
  - Set appropriate batch frequency (hourly, nightly, weekly)
  - Schedule during off-peak hours if possible
  - Implement failure notifications

- [ ] **Implement error handling**
  - Parse error reports from re:Search
  - Log validation failures
  - Alert on batch failures
  - Create retry mechanisms
  - Document troubleshooting steps

- [ ] **Create operational runbook**
  - Document batch generation process
  - Include troubleshooting procedures
  - List key contacts and escalation paths
  - Provide example commands and scripts

### Phase 3: Testing

- [ ] **Test with sample data** (10-100 records)
  - Verify basic schema compliance
  - Test manifest generation
  - Confirm upload process works
  - Review processing results

- [ ] **Test with realistic data** (1,000-10,000 records)
  - Include all case types and statuses
  - Test security classifications
  - Verify cross-references (cases → filings)
  - Check character encoding edge cases

- [ ] **Perform load testing** (production-sized datasets)
  - Test with full production volume
  - Measure export time and resource usage
  - Verify file compression ratios
  - Test upload reliability
  - Monitor processing times

- [ ] **Test error scenarios**
  - Submit batch with invalid JSON
  - Submit batch with missing required fields
  - Submit batch with wrong checksums
  - Submit batch with missing files
  - Verify error reports are clear and actionable

- [ ] **Test incremental updates** (if using delta batches)
  - Verify only changed records exported
  - Test addition, modification, deletion scenarios
  - Confirm delta logic correctness

### Phase 4: Production Deployment

- [ ] **Obtain production credentials**
  - Request production S3/SFTP access
  - Configure credential storage securely
  - Test production connectivity

- [ ] **Perform historical backfill** (if applicable)
  - Plan backfill window (nights/weekends)
  - Split into manageable batches
  - Monitor progress closely
  - Verify historical data indexed correctly

- [ ] **Configure production scheduling**
  - Set final batch frequency
  - Adjust timing based on court needs
  - Enable monitoring and alerts
  - Document schedule for operations team

- [ ] **Execute parallel run** (optional but recommended)
  - Run new batch process alongside existing system
  - Compare results for consistency
  - Identify and fix any discrepancies
  - Build confidence before cutover

- [ ] **Perform production cutover**
  - Coordinate with court and re:Search team
  - Submit first production batch
  - Monitor processing closely
  - Verify data searchable in re:Search portal

- [ ] **Establish ongoing monitoring**
  - Set up batch success/failure alerts
  - Monitor processing times and trends
  - Review error reports regularly
  - Track data quality metrics

---

## Court Checklist

### Phase 1: Planning

- [ ] **Coordinate with CMS vendor**
  - Agree on batch schedule and scope
  - Define data coverage (case types, date ranges)
  - Discuss historical backfill needs
  - Establish project timeline

- [ ] **Provide court-specific information**
  - Supply court codes and jurisdiction mapping
  - Define security classification policies
  - Clarify any custom data requirements
  - Identify sealed/confidential case handling

- [ ] **Designate project contacts**
  - Assign technical contact for CMS coordination
  - Assign business contact for scope decisions
  - Identify support contact for troubleshooting
  - Establish escalation procedures

- [ ] **Review data scope**
  - Determine which case types to include
  - Define date range for historical data
  - Identify any exclusions or special cases
  - Plan for ongoing updates vs. historical backfill

### Phase 2: Preparation

- [ ] **Plan historical backfill** (if applicable)
  - Determine backfill date range
  - Estimate data volume and batch count
  - Schedule backfill window (nights/weekends)
  - Coordinate with CMS vendor on timing

- [ ] **Establish change window**
  - Minimize court operations disruption
  - Communicate to staff and stakeholders
  - Plan for validation after go-live
  - Prepare rollback procedures if needed

- [ ] **Schedule testing period**
  - Coordinate with re:Search team
  - Allocate staff time for validation
  - Plan for sample case verification
  - Establish acceptance criteria

- [ ] **Review security requirements**
  - Confirm sealed case handling procedures
  - Verify confidential case protections
  - Test access controls in re:Search portal
  - Document security classification mapping

### Phase 3: Testing and Validation

- [ ] **Validate test data**
  - Review sample cases in re:Search portal
  - Verify case metadata accuracy
  - Test search functionality
  - Confirm security filters working

- [ ] **Test user access**
  - Verify public users see appropriate cases
  - Test internal staff access levels
  - Confirm sealed cases are protected
  - Validate search results accuracy

- [ ] **Review error reports**
  - Work with CMS vendor to resolve validation errors
  - Prioritize critical vs. minor issues
  - Verify corrections in subsequent batches

### Phase 4: Go-Live

- [ ] **Communicate with stakeholders**
  - Notify staff of timeline and changes
  - Update public-facing documentation
  - Prepare for questions and support requests

- [ ] **Monitor initial production batches**
  - Review first batch results closely
  - Verify data accuracy and completeness
  - Check processing times meet expectations
  - Address any issues immediately

- [ ] **Establish success metrics**
  - Define KPIs (batch success rate, processing time, error rate)
  - Set up monitoring dashboards
  - Schedule regular review meetings
  - Document lessons learned

- [ ] **Train staff**
  - Provide overview of new batch process
  - Explain monitoring and troubleshooting
  - Document support procedures
  - Establish escalation paths

---

## Testing Requirements

### Test Environment

**What You'll Receive:**
- Test S3 bucket or SFTP directory
- Test credentials (scoped to test environment only)
- Access to test re:Search portal for validation
- Test error report directory

**Test Data Requirements:**
- Realistic sample data (anonymized if needed)
- All case types represented
- Various security levels (public, confidential, sealed)
- Edge cases (special characters, long text, null values)
- Cross-references (cases with filings, parties, documents)

### Certification Testing

Before production deployment, you must pass certification testing:

#### 1. Schema Compliance Test
- Submit batch with all required fields
- Include all case types and statuses
- Zero validation errors
- All records successfully indexed

#### 2. Volume Test
- Submit batch with production-like volume
- Processing completes within acceptable timeframe
- No performance degradation
- All records processed successfully

#### 3. Error Handling Test
- Submit batch with intentional errors
- Verify error reports are generated correctly
- Confirm invalid records rejected appropriately
- Validate error messages are clear

#### 4. Security Test
- Submit cases with various security levels
- Verify sealed cases not visible to public
- Confirm confidential cases have restricted access
- Test security filter enforcement

#### 5. End-to-End Test
- Submit complete batch from CMS
- Verify processing pipeline works end-to-end
- Confirm data searchable in portal
- Validate data accuracy and completeness

### Acceptance Criteria

- **Batch Success Rate:** 95%+ of test batches processed successfully
- **Data Quality:** 99%+ of records pass validation
- **Processing Time:** Within agreed SLA (typically 2 hours for nightly batch)
- **Error Handling:** Errors reported clearly with actionable messages
- **Security:** All access controls enforced correctly

---

## Go-Live Process

### Pre-Go-Live Checklist

- [ ] Certification testing passed
- [ ] Production credentials obtained and tested
- [ ] Batch schedule configured for production
- [ ] Monitoring and alerts set up
- [ ] Runbook documented and reviewed
- [ ] Stakeholders notified of go-live date
- [ ] Rollback plan prepared
- [ ] Support contacts confirmed

### Go-Live Steps

#### 1. Historical Backfill (if applicable)

**Week 1-2: Backfill Preparation**
- Generate historical batches in chunks (e.g., by year)
- Test first historical batch in production
- Verify processing and indexing
- Adjust batch size if needed

**Week 3-4: Backfill Execution**
- Upload historical batches sequentially
- Monitor each batch closely
- Address any errors before proceeding
- Verify cumulative data in portal

#### 2. Parallel Run (Recommended)

**Week 1-2: Shadow Mode**
- Run new batch process alongside existing system
- Do not disable old system yet
- Compare results for consistency
- Identify and resolve discrepancies

**Week 3: Validation**
- Verify data completeness in re:Search
- Test search functionality thoroughly
- Confirm with court staff that data is accurate
- Address any issues found

#### 3. Production Cutover

**Day 1: First Production Batch**
- Submit first production batch
- Monitor processing in real-time
- Verify successful completion
- Check data in re:Search portal

**Day 2-7: Close Monitoring**
- Review each batch carefully
- Check error reports daily
- Monitor processing times
- Address issues immediately

**Week 2-4: Stabilization**
- Continue daily monitoring
- Fine-tune batch schedule if needed
- Optimize performance
- Document any issues and resolutions

#### 4. Ongoing Operations

**Daily:**
- Monitor batch success/failure
- Review error reports for recurring issues
- Verify processing times within SLA

**Weekly:**
- Analyze trends (error rates, processing times)
- Review data quality metrics
- Check for any patterns in failures

**Monthly:**
- Review overall batch performance
- Evaluate optimization opportunities
- Update documentation as needed
- Coordinate with re:Search team on improvements

---

## Common Implementation Challenges

### Challenge: Large Historical Backfill

**Solution:**
- Split into manageable batches (1-2 years per batch)
- Run during off-peak hours/weekends
- Monitor disk space on CMS and re:Search
- Use incremental approach (oldest to newest)

### Challenge: Complex Data Transformations

**Solution:**
- Document transformation logic clearly
- Create reusable transformation functions
- Test edge cases thoroughly
- Consider staging tables for complex transformations

### Challenge: Network Reliability

**Solution:**
- Implement automatic retry logic
- Use compression to reduce transfer time
- Monitor upload success rates
- Consider batch splitting for large files

### Challenge: Schema Changes

**Solution:**
- Subscribe to schema change notifications
- Test schema updates in test environment first
- Implement version compatibility checks
- Maintain backward compatibility when possible

### Challenge: Error Rate Too High

**Solution:**
- Analyze error patterns (common field issues?)
- Improve pre-upload validation
- Fix systemic data quality issues in CMS
- Work with re:Search team on schema clarifications

---

## Next Steps

- **Review technical details** → Read [Technical Setup](./technical-setup.md)
- **Learn about operations** → Read [Operations](./operations.md)
- **Understand the architecture** → Read [How It Works](./how-it-works.md)
- **Return to overview** → [Batch Mode Overview](./README.md)

---

**Need Help?**

Contact your re:Search court liaison or technical project manager for assistance with implementation planning.

**Last Updated:** November 2025