# Operations and Troubleshooting

[← Back to Batch Mode Overview](./README.md)

## Quick Navigation
- [Monitoring and Metrics](#monitoring-and-metrics)
- [Common Issues](#common-issues)
- [Troubleshooting Guide](#troubleshooting-guide)
- [Getting Help](#getting-help)

---

## Monitoring and Metrics

### Key Performance Indicators

Monitor these metrics to ensure healthy batch operations:

| Metric | Target | Alert Threshold | Action if Exceeded |
|--------|--------|-----------------|-------------------|
| **Batch completion rate** | 99%+ | < 95% | Investigate recurring failures |
| **Record validation rate** | 99.9%+ | < 99% | Review data quality in CMS |
| **Processing latency** | < 2 hours | > 4 hours | Check batch size and complexity |
| **Error resolution time** | < 24 hours | > 48 hours | Escalate to CMS vendor/support |
| **Storage utilization** | < 80% | > 90% | Clean up old batches |
| **Upload success rate** | 99%+ | < 95% | Check network and credentials |

### Daily Monitoring Checklist

**Morning:**
- [ ] Check overnight batch status (success/failure)
- [ ] Review error reports for any failed batches
- [ ] Verify processing completed within SLA
- [ ] Check storage utilization

**Throughout Day:**
- [ ] Monitor real-time batch processing (if running multiple times daily)
- [ ] Address any failed batch alerts immediately
- [ ] Review error patterns for recurring issues

**Weekly:**
- [ ] Analyze batch performance trends
- [ ] Review error rate changes over time
- [ ] Check for schema version updates
- [ ] Verify credential expiration dates

**Monthly:**
- [ ] Generate batch performance reports
- [ ] Review optimization opportunities
- [ ] Update documentation based on lessons learned
- [ ] Coordinate with re:Search team on improvements

### Operational Dashboard (Coming Soon)

The re:Search team is developing a self-service dashboard showing:

- **Recent batch history** – Last 30 days with status indicators
- **Success/failure status** – Visual timeline of batch results
- **Record counts** – Cases, filings, parties, documents processed
- **Processing times** – Historical performance metrics
- **Error summaries** – Aggregated error types and counts
- **Trend analysis** – Week-over-week and month-over-month comparisons

**Current Monitoring:**
- Email notifications sent to vendor contact on batch completion/failure
- Error reports uploaded to `_reports/` directory in S3/SFTP
- Check error reports manually for detailed diagnostics

### Log Retention

| Item | Retention Period | Purpose |
|------|------------------|---------|
| **Batch files** | 90 days | Reprocessing, troubleshooting |
| **Error reports** | 1 year | Trend analysis, auditing |
| **Audit logs** | Per court policy | Compliance, security review |
| **Successful batches** | Archived after indexing | Historical reference |

---

## Common Issues

### Batch Not Detected

**Symptoms:**
- Batch uploaded but no processing notification
- Files visible in S3/SFTP but not processed
- No error report generated

**Causes:**
- Incorrect upload path (wrong vendor/court/date structure)
- Manifest not uploaded or uploaded first
- File permissions issues (S3 IAM policy, SFTP permissions)

**Solutions:**
1. Verify upload path matches agreed structure:
   ```
   s3://bucket/Vendor/CourtCode/YYYY-MM-DD/manifest.json
   ```
2. Ensure manifest uploaded **last** to signal batch ready
3. Check file permissions and IAM policies
4. Verify credentials are active and not expired
5. Contact re:Search team if path structure is unclear

---

### Checksum Mismatch

**Symptoms:**
- Error report shows "SHA-256 checksum verification failed"
- Batch rejected at file integrity validation stage

**Causes:**
- File corrupted during upload (network issue)
- File modified after checksum calculated
- Checksum calculated on wrong file (compressed vs. uncompressed)
- Line ending conversion (Windows CRLF vs. Unix LF)

**Solutions:**
1. Re-calculate checksum and verify it matches manifest
2. Re-upload affected file
3. Ensure checksum calculated on final file (after compression)
4. Use binary mode for file operations (avoid text mode conversions)
5. Verify no file modifications between checksum and upload

**Prevention:**
```python
# Good: Calculate checksum on compressed file
import gzip
import hashlib

# Create compressed file
with open('cases.jsonl', 'rb') as f_in:
    with gzip.open('cases.jsonl.gz', 'wb') as f_out:
        f_out.writelines(f_in)

# Calculate checksum on compressed file
sha256 = hashlib.sha256()
with open('cases.jsonl.gz', 'rb') as f:
    for chunk in iter(lambda: f.read(4096), b""):
        sha256.update(chunk)
checksum = sha256.hexdigest()
```

---

### Schema Validation Errors

**Symptoms:**
- Error report shows "Missing required field" or "Invalid data type"
- Records rejected during schema validation stage
- High error rate in error report summary

**Causes:**
- Missing required fields (caseId, courtCode, caseNumber, etc.)
- Wrong data types (string where integer expected, vice versa)
- Invalid enum values (security level "private" instead of "CONFIDENTIAL")
- Invalid date formats (MM/DD/YYYY instead of YYYY-MM-DD)
- Special characters not properly escaped

**Solutions:**
1. Review error report for specific field issues
2. Implement pre-upload validation using JSON schema
3. Fix systemic issues in CMS export logic
4. Test with small sample before full batch
5. Request schema documentation if unclear

**Common Field Issues:**

| Field | Common Error | Correct Format |
|-------|--------------|----------------|
| `filingDate` | `"03/15/2024"` | `"2024-03-15"` |
| `security.level` | `"private"` | `"CONFIDENTIAL"` |
| `caseNumber` | Missing | Always required |
| `courtCode` | `"Ellis County"` | `"ELLIS-CC"` (code, not name) |

---

### Permission Denied (S3)

**Symptoms:**
- Upload fails with "Access Denied" error
- AWS CLI shows 403 Forbidden
- Cannot list bucket contents

**Causes:**
- IAM credentials expired or incorrect
- Wrong AWS access key ID or secret key
- IAM policy doesn't allow writes to this path
- Bucket policy restricts access by IP or condition

**Solutions:**
1. Verify credentials are current:
   ```bash
   aws sts get-caller-identity
   ```
2. Request credential rotation if expired
3. Verify bucket and path in IAM policy
4. Check source IP if bucket restricts by IP
5. Test with simple upload command:
   ```bash
   echo "test" | aws s3 cp - s3://bucket/path/test.txt
   ```

---

### Connection Refused (SFTP)

**Symptoms:**
- SFTP connection times out or refused
- SSH key authentication fails
- Cannot connect to SFTP server

**Causes:**
- Source IP not whitelisted
- Wrong SSH key or incorrect key permissions
- SFTP server address or port incorrect
- Network firewall blocking outbound SFTP

**Solutions:**
1. Verify SFTP server address and port (typically 22)
2. Check SSH key file permissions:
   ```bash
   chmod 600 ~/.ssh/research_key
   ```
3. Test SSH key authentication:
   ```bash
   ssh -i ~/.ssh/research_key user@sftp.server.address
   ```
4. Provide source IP to re:Search team for whitelisting
5. Check network/firewall allows outbound port 22

---

### Duplicate BatchId

**Symptoms:**
- Error report shows "Duplicate batchId"
- Batch rejected at manifest validation stage

**Causes:**
- Re-running same batch without changing batchId
- BatchId generation logic not unique
- Manually using same batchId as previous submission

**Solutions:**
1. Use unique batchId for each attempt
2. Include sequence number in batchId:
   ```
   ELLIS-2025-11-19-0100  (first attempt)
   ELLIS-2025-11-19-0101  (second attempt)
   ```
3. Implement batchId generation with timestamp:
   ```python
   from datetime import datetime
   batch_id = f"ELLIS-{datetime.now().strftime('%Y-%m-%d-%H%M')}"
   ```
4. Avoid hardcoding batchIds in scripts

---

### Processing Timeout

**Symptoms:**
- Batch processing takes longer than expected
- Processing appears stuck or incomplete
- Timeout error in processing logs

**Causes:**
- Batch too large (millions of records)
- Complex nested data structures
- High system load on re:Search platform
- Network latency during file download

**Solutions:**
1. Split large batches into smaller chunks:
   - Recommended: 50,000-100,000 records per file
   - Maximum: 1,000,000 records per batch
2. Simplify data structures (avoid deep nesting)
3. Compress files to reduce download time
4. Schedule during off-peak hours
5. Contact re:Search team if timeouts persist

---

### Missing Cross-References

**Symptoms:**
- Error report shows "Invalid caseId reference"
- Filings/parties reference cases not found
- Cross-reference validation failures

**Causes:**
- Filings file references cases not in cases file
- Cases and filings processed out of order
- Case filtering excludes cases referenced by filings
- CaseId mismatch between files

**Solutions:**
1. Ensure cases file includes all referenced cases
2. Process files in order: cases → filings → parties → documents
3. Apply consistent filtering across all file types
4. Validate cross-references before upload:
   ```python
   case_ids = set()
   # Read all case IDs from cases file
   for case in cases:
       case_ids.add(case['caseId'])
   
   # Validate filings reference valid cases
   for filing in filings:
       if filing['caseId'] not in case_ids:
           print(f"Invalid reference: {filing['caseId']}")
   ```

---

## Troubleshooting Guide

### Debug Checklist

When troubleshooting batch failures, follow this systematic approach:

1. **Check error report** in `_reports/` directory
   - Review error summary for counts and types
   - Examine specific error details
   - Note error severity (ERROR vs. WARNING)

2. **Verify manifest structure**
   - Confirm all required fields present
   - Check file paths match actual files
   - Verify checksums calculated correctly
   - Ensure batchId is unique

3. **Validate file integrity**
   - Re-calculate checksums locally
   - Compare with manifest values
   - Check file sizes match manifest
   - Verify files are readable/not corrupted

4. **Test JSON parsing locally**
   - Validate JSON Lines format
   - Check for syntax errors
   - Verify character encoding (UTF-8)
   - Test with JSON validator

5. **Review file permissions**
   - Check S3 IAM policies
   - Verify SFTP directory permissions
   - Test write access to upload location

6. **Check network connectivity**
   - Test S3/SFTP connectivity
   - Verify firewall allows outbound traffic
   - Check for proxy or VPN issues

7. **Verify credentials**
   - Confirm credentials are current
   - Check expiration dates
   - Test with simple upload/list command

8. **Inspect upload logs**
   - Review CMS upload logs
   - Check for upload errors or warnings
   - Verify upload completion

9. **Contact support** if issue persists
   - Provide batchId
   - Include error report
   - Share sample data (sanitized)
   - Describe troubleshooting steps taken

### Error Report Analysis

**High-Priority Issues (Address Immediately):**
- Batch-level failures (manifest errors, missing files)
- >1% error rate in record validation
- Security classification errors (sealed case exposed)
- Cross-reference integrity issues

**Medium-Priority Issues (Address Within 24 Hours):**
- Individual record validation errors (<1% rate)
- Data format inconsistencies
- Warning-level issues that may become errors

**Low-Priority Issues (Monitor and Plan Fix):**
- INFO-level messages
- Performance optimization opportunities
- Non-critical data quality issues

---

## Getting Help

### Support Resources

**For Batch Processing Issues:**
1. Check error report in `_reports/` directory first
2. Review this troubleshooting guide
3. Check [FAQ](../../reference/faq.md) for common questions
4. Contact your court liaison or technical project manager

**For Credential Issues:**
- Email re:Search technical support
- Request credential rotation or permission adjustment
- Verify IAM policy or SSH key configuration

**For Schema Questions:**
- Review schema documentation provided during onboarding
- Request clarification from re:Search technical team
- Test with small sample in test environment

### Support Ticket Information

When contacting support, provide:

- **BatchId** – Unique identifier for the failed batch
- **Error report** – Full JSON error report from `_reports/`
- **Manifest** – The manifest.json file used
- **Sample data** – Sanitized sample records (if applicable)
- **Upload logs** – CMS logs showing upload attempts
- **Troubleshooting steps** – What you've tried already

**Example Support Request:**

```
Subject: Batch Validation Failure - ELLIS-2025-11-19-0100

BatchId: ELLIS-2025-11-19-0100
Issue: Batch failed schema validation with 1,234 errors (2% error rate)

Error Summary:
- 1,150 records: Missing required field "caseNumber"
- 84 records: Invalid date format in "filingDate"

Troubleshooting Completed:
- Verified manifest structure is correct
- Checked sample records locally - unable to identify pattern
- Reviewed schema documentation

Attached:
- batch-ELLIS-2025-11-19-0100-errors.json
- manifest.json
- sample_cases.jsonl (10 failed records, sanitized)

Request: Assistance identifying why caseNumber field is missing from 
exported records. Field is present in CMS database.
```

### Escalation Procedures

**Normal Priority (Response within 1 business day):**
- Non-critical batch failures
- Data quality questions
- Schema clarifications

**High Priority (Response within 4 hours):**
- Production batch failures affecting operations
- Security issues (sealed cases exposed)
- Credential/access problems blocking uploads

**Critical (Immediate response):**
- Data breach or security incident
- System-wide failures affecting multiple courts
- Issues requiring immediate rollback

### Contact Information

- **Technical Support Portal:** (if available)
- **Email:** research-support@courts.state
- **Phone:** During business hours (8am-5pm CT)
- **Emergency Hotline:** For critical issues only

---

## Best Practices

### Proactive Monitoring

- Set up automated alerts for batch failures
- Monitor error rates and trends weekly
- Review processing times regularly
- Test with sample data before major changes

### Data Quality

- Implement pre-upload validation
- Fix systemic issues in CMS, not just symptoms
- Document data transformation logic
- Maintain consistency across batches

### Operational Excellence

- Document all processes and procedures
- Keep runbooks updated with lessons learned
- Train multiple team members on operations
- Establish clear escalation paths

### Performance Optimization

- Compress files to reduce transfer time
- Split large batches for faster processing
- Schedule during off-peak hours
- Monitor and optimize CMS export queries

---

## Next Steps

- **Plan your implementation** → Read [Implementation Guide](./implementation-guide.md)
- **Review technical details** → Read [Technical Setup](./technical-setup.md)
- **Understand the architecture** → Read [How It Works](./how-it-works.md)
- **Return to overview** → [Batch Mode Overview](./README.md)

---

**Last Updated:** November 2025