# Batch Mode

## Table of Contents

- [Overview](#overview)
- [At a Glance](#at-a-glance)
- [How Batch Mode Works](#how-batch-mode-works)
  - [Integration Flow](#integration-flow)
  - [Step-by-Step Process](#step-by-step-process)
- [Key Benefits](#key-benefits)
  - [Low Technical Barrier](#low-technical-barrier)
  - [High Performance](#high-performance)
  - [Data Quality Assurance](#data-quality-assurance)
  - [Operational Flexibility](#operational-flexibility)
- [Limitations and Considerations](#limitations-and-considerations)
- [Strategic Context and Roadmap](#strategic-context-and-roadmap)
- [Transport and Authentication](#transport-and-authentication)
  - [AWS S3 (Recommended)](#aws-s3-recommended)
  - [SFTP (Alternative)](#sftp-alternative)
  - [Credential Management](#credential-management)
- [Manifest and Schema Requirements](#manifest-and-schema-requirements)
  - [Manifest Structure](#manifest-structure)
  - [Required Manifest Fields](#required-manifest-fields)
  - [Optional Manifest Fields](#optional-manifest-fields)
- [Data File Schema](#data-file-schema)
  - [Cases File Example](#cases-file-example)
  - [Required Case Fields](#required-case-fields)
- [Validation and Error Handling](#validation-and-error-handling)
  - [Validation Stages](#validation-stages)
  - [Error Reporting](#error-reporting)
- [Implementation Checklist](#implementation-checklist)
  - [For CMS Vendors](#for-cms-vendors)
  - [For Courts](#for-courts)
- [Monitoring and Operations](#monitoring-and-operations)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)
- [Related Documentation](#related-documentation)

---

## Overview

Batch Mode is a **file-based integration method** that uses scheduled JSON snapshot deliveries to synchronize case, filing, party, and document metadata with re:Search. Unlike real-time API-driven modes, Batch Mode emphasizes predictability, high-volume throughput, and schema-driven data quality.

This mode is ideal for courts and CMS vendors who need **bulk ingestion capabilities**, **historical data migration**, or prefer **offline processing** over real-time API integration.

**Key Characteristics:**
- Scheduled snapshot delivery via file uploads
- JSON Lines format for data files
- AWS S3 or SFTP transport
- Schema-driven validation
- Handles millions of records efficiently

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

## How Batch Mode Works

Batch Mode operates as a **one-way file delivery system** where your CMS generates complete or incremental snapshots of case data and uploads them to a designated storage location. re:Search automatically detects new batches, validates their contents, and indexes the data.

### Integration Flow

```
┌─────────────┐                    ┌─────────────┐                    ┌──────────┐
│    CMS      │                    │  S3/SFTP    │                    │re:Search │
│   System    │                    │   Storage   │                    │ Platform │
└─────────────┘                    └─────────────┘                    └──────────┘
      │                                    │                                 │
      │ 1. Generate JSON snapshot          │                                 │
      │    (cases, filings, parties)       │                                 │
      │                                    │                                 │
      │ 2. Create manifest.json            │                                 │
      │    with metadata                   │                                 │
      │                                    │                                 │
      │ 3. Compress files (.jsonl.gz)      │                                 │
      │                                    │                                 │
      │ 4. Calculate checksums (SHA-256)   │                                 │
      │                                    │                                 │
      │ 5. Upload manifest + data files    │                                 │
      ├───────────────────────────────────►│                                 │
      │                                    │                                 │
      │                                    │ 6. Detect new batch             │
      │                                    │    (polling/event)              │
      │                                    │◄────────────────────────────────┤
      │                                    │                                 │
      │                                    │ 7. Download and validate        │
      │                                    │    manifest                     │
      │                                    │◄────────────────────────────────┤
      │                                    │                                 │
      │                                    │ 8. Verify file integrity        │
      │                                    │    (checksums)                  │
      │                                    │◄────────────────────────────────┤
      │                                    │                                 │
      │                                    │                                 │ 9. Parse and validate
      │                                    │                                 │    JSON schema
      │                                    │                                 │
      │                                    │                                 │ 10. Apply business
      │                                    │                                 │     rules validation
      │                                    │                                 │
      │                                    │                                 │ 11. Index valid
      │                                    │                                 │     records
      │                                    │                                 │
      │                                    │ 12. Generate ingestion report   │
      │                                    │◄────────────────────────────────┤
      │                                    │                                 │
      │                                    │                                 │ 13. Data searchable
```

### Step-by-Step Process

1. **Export Generation**
   - Your CMS queries case data from database
   - Formats data according to Batch Mode schema
   - Applies court-specific filtering and rules

2. **File Creation**
   - Data written as JSON Lines files (one JSON object per line)
   - Large datasets split into multiple files for manageability
   - Each file type handled separately (cases, filings, parties, documents)

3. **Manifest Creation**
   - `manifest.json` file catalogs all data files
   - Includes checksums for integrity verification
   - Documents record counts and metadata

4. **Compression (Optional but Recommended)**
   - Files compressed using gzip (`.jsonl.gz`)
   - Reduces transfer time and bandwidth
   - Maintains data integrity during compression

5. **Upload**
   - All files uploaded to designated S3 bucket or SFTP directory
   - Atomic upload ensures complete batch delivery
   - Manifest uploaded last to signal batch completion

6. **Detection**
   - re:Search monitors storage location for new batches
   - Automatic detection via S3 events or periodic polling
   - Batch processing begins immediately upon detection

7. **Validation**
   - Schema validation ensures data structure correctness
   - Checksum verification detects corruption
   - Business rule checks validate data relationships

8. **Indexing**
   - Valid records indexed and made searchable
   - Invalid records logged in error report
   - Atomic indexing ensures data consistency

9. **Reporting**
   - Success/failure reports generated
   - Error details provided for troubleshooting
   - Metrics logged for monitoring

## Key Benefits

### Low Technical Barrier

**No API Development Required**
- Avoids SOAP/REST complexity
- No web service endpoints to maintain
- Simpler for CMS vendors without API expertise

**Standard File Formats**
- Uses widely-supported JSON format
- Standard compression (gzip)
- Compatible with any programming language

**Flexible Scheduling**
- Run exports on your court's timeline
- Batch during off-peak hours
- No real-time processing pressure

### High Performance

**Scales to Millions of Records**
- Handles large courts efficiently
- Proven with 10+ million record datasets
- Predictable performance characteristics

**Parallel Processing**
- Multiple files processed simultaneously
- Optimized for bulk operations
- Better throughput than API polling

**Efficient Resource Usage**
- Batch processing uses resources efficiently
- Lower overhead than continuous API calls
- Better for high-volume data transfer

### Data Quality Assurance

**Schema-Based Validation**
- Ensures data consistency before indexing
- Catches errors early in process
- Provides clear validation feedback

**Checksum Verification**
- Detects corruption during transfer
- Ensures file integrity
- Prevents partial data processing

**Rollback Capability**
- Failed batches don't corrupt production data
- Easy to reprocess corrected files
- Maintains data integrity

### Operational Flexibility

**Historical Backfills Supported**
- Load decades of legacy data
- Migrate from old systems
- Populate new installations quickly

**Reprocessing Capability**
- Re-run batches if needed
- Correct data errors and resubmit
- Iterative refinement during implementation

**Delta or Full Snapshots**
- Choose incremental updates (changes only)
- Or complete snapshots (all data)
- Flexibility based on court needs

**Works with Any CMS**
- Compatible with Odyssey, Tyler Eagle, custom systems
- No vendor-specific requirements
- Technology-agnostic approach

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

**When Not to Use Batch Mode:**

- Court requires real-time filing updates (use ECF Mode)
- Very small court with minimal filings (use Non-Integrated Mode)
- CMS already has ECF integration with EFM
- Budget for full API integration available

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

## Transport and Authentication

Batch Mode uses **secure file delivery** rather than API authentication. Two transport methods are supported:

### AWS S3 (Recommended)

**Why S3?**

- **Faster transfer speeds** – Optimized for large file uploads
- **Built-in encryption** – AES-256 encryption at rest
- **Automatic retry** – Built-in error handling and retry logic
- **Event-driven processing** – Instant batch detection and processing
- **Integrated monitoring** – CloudWatch integration for operational visibility
- **Scalability** – Handles any file size without configuration
- **Cost-effective** – Pay only for storage used

**Security Features:**

- IAM role-based access control
- Bucket policies restrict access to specific courts/vendors
- TLS 1.2+ encryption in transit
- Server-side encryption at rest (SSE-S3 or SSE-KMS)
- S3 versioning for audit trail
- MFA delete protection for production buckets

**S3 Path Structure:**

```text
s3://tx-research-ingest-prod/<CMSVendor>/<CourtCode>/<YYYY-MM-DD>/
  ├── manifest.json
  ├── cases_0001.jsonl.gz
  ├── cases_0002.jsonl.gz
  ├── filings_0001.jsonl.gz
  ├── parties_0001.jsonl.gz
  └── documents_0001.jsonl.gz
```

**Upload Methods:**

**AWS CLI:**
```bash
aws s3 cp batch/ s3://tx-research-ingest-prod/MyVendor/ELLIS-CC/2025-11-19/ --recursive
```

**Python (boto3):**
```python
import boto3

s3 = boto3.client('s3')
bucket = 'tx-research-ingest-prod'
prefix = 'MyVendor/ELLIS-CC/2025-11-19/'

s3.upload_file('manifest.json', bucket, f'{prefix}manifest.json')
s3.upload_file('cases_0001.jsonl.gz', bucket, f'{prefix}cases_0001.jsonl.gz')
```

**Java SDK:**
```java
AmazonS3 s3Client = AmazonS3ClientBuilder.defaultClient();
String bucket = "tx-research-ingest-prod";
String key = "MyVendor/ELLIS-CC/2025-11-19/manifest.json";

s3Client.putObject(bucket, key, new File("manifest.json"));
```

### SFTP (Alternative)

**When to Use SFTP:**

- Your CMS cannot integrate with AWS S3
- Corporate policy restricts cloud storage
- Existing SFTP automation already in place
- Air-gapped environments requiring file-based transfer

**Security Features:**

- SSH key-based authentication (password auth disabled)
- Chroot jail isolation per court
- TLS encryption for all transfers
- Connection logging and audit trails
- IP whitelisting for source systems

**SFTP Configuration:**

```text
Host: sftp.research.courts.state
Port: 22
Path: /research-ingest/<CMSVendor>/<CourtCode>/<YYYY-MM-DD>/
Authentication: SSH key (provided by re:Search team)
```

**Upload Example:**

```bash
# Using sftp command
sftp -i ~/.ssh/research_key user@sftp.research.courts.state
cd /research-ingest/MyVendor/ELLIS-CC/2025-11-19/
put manifest.json
put cases_*.jsonl.gz
put filings_*.jsonl.gz
put parties_*.jsonl.gz
put documents_*.jsonl.gz
bye

# Using scp for automation
scp -i ~/.ssh/research_key batch/* user@sftp.research.courts.state:/research-ingest/MyVendor/ELLIS-CC/2025-11-19/
```

**SFTP Best Practices:**

- Test connectivity before production
- Implement retry logic for network failures
- Monitor transfer completion
- Verify file sizes after upload
- Clean up old batches periodically

### Credential Management

**S3 Credentials:**

- Provided by re:Search team during onboarding
- IAM access key + secret key
- Rotate every 90 days (automated reminder)
- Scoped to specific buckets only
- Never commit credentials to source control
- Use environment variables or secret management systems

**SFTP Credentials:**

- SSH private key provided via secure channel
- Public key fingerprint verification required
- Key rotation every 180 days
- Store keys securely (encrypted, restricted permissions)
- Never share keys between environments

**Credential Storage Best Practices:**

- Use secrets management (AWS Secrets Manager, HashiCorp Vault)
- Restrict file permissions (chmod 600 for SSH keys)
- Audit credential access
- Implement credential rotation procedures
- Document recovery procedures

## Manifest and Schema Requirements

Every batch **must include a manifest.json file** that describes the batch contents. This file enables re:Search to validate file integrity before processing.

### Manifest Structure

```json
{
  "batchId": "ELLIS-2025-11-19-0100",
  "schemaVersion": "1.3.0",
  "generatedAt": "2025-11-19T01:05:12Z",
  "courtSystem": "ELLIS-COUNTY",
  "batchType": "FULL",
  "files": [
    {
      "type": "cases",
      "path": "cases_0001.jsonl.gz",
      "recordCount": 50000,
      "sha256": "a3d8f9e7c2b1a5d4e8f9c7b6a5d4e8f9c7b6a5d4e8f9c7b6a5d4e8f9c7b6a5d4",
      "sizeBytes": 15728640
    },
    {
      "type": "filings",
      "path": "filings_0001.jsonl.gz",
      "recordCount": 54000,
      "sha256": "b4e9f0a8d3c2b6e5f9a8d7c6b5e4f9a8d7c6b5e4f9a8d7c6b5e4f9a8d7c6b5e4",
      "sizeBytes": 18874368
    },
    {
      "type": "parties",
      "path": "parties_0001.jsonl.gz",
      "recordCount": 85000,
      "sha256": "d6a1c2f0b9e5d4c8f7a0b9e6d5c4f8a7b0e9d6c5f4a8b7e0d9c6f5a4b8e7d0c9",
      "sizeBytes": 8388608
    },
    {
      "type": "documents",
      "path": "documents_0001.jsonl.gz",
      "recordCount": 61000,
      "sha256": "c5f0a1b9e4d3c7f6a9b8e7d6c5f4a9b8e7d6c5f4a9b8e7d6c5f4a9b8e7d6c5f4",
      "sizeBytes": 12582912
    }
  ],
  "scope": {
    "courtCodes": ["ELLIS-CC", "ELLIS-DC", "ELLIS-JP1"],
    "asOfDate": "2025-11-18",
    "includesHistorical": false
  },
  "contact": {
    "vendor": "Acme CMS Inc.",
    "email": "support@acmecms.com",
    "phone": "+1-555-0100"
  }
}
```

### Required Manifest Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `batchId` | string | Yes | Unique identifier (format: `<COURT>-<DATE>-<SEQ>`) |
| `schemaVersion` | string | Yes | Must match current schema version (currently `1.3.0`) |
| `generatedAt` | ISO 8601 | Yes | Timestamp when batch was created (UTC recommended) |
| `files` | array | Yes | List of all data files with metadata |
| `files[].type` | enum | Yes | One of: `cases`, `filings`, `parties`, `documents` |
| `files[].path` | string | Yes | Relative filename (must exist in same directory) |
| `files[].recordCount` | integer | Yes | Number of JSON objects in file |
| `files[].sha256` | string | Yes | SHA-256 checksum of file contents (lowercase hex) |
| `files[].sizeBytes` | integer | Yes | File size in bytes |
| `scope.courtCodes` | array | Yes | Court identifiers covered by this batch |
| `scope.asOfDate` | date | Yes | Effective date of data snapshot (YYYY-MM-DD) |

### Optional Manifest Fields

| Field | Type | Description |
|-------|------|-------------|
| `batchType` | enum | `FULL` (complete snapshot) or `DELTA` (changes only) |
| `scope.includesHistorical` | boolean | `true` if batch contains backfilled historical data |
| `contact.vendor` | string | CMS vendor name for support purposes |
| `contact.email` | string | Support contact email for batch issues |
| `contact.phone` | string | Support contact phone number |
| `notes` | string | Additional information about this batch |

## Data File Schema

Each data file must be in **JSON Lines format** (one JSON object per line, no commas between objects, no outer array).

### Cases File Example

**File: cases_0001.jsonl**

```json
{"caseId":"2024-CV-00123","courtCode":"ELLIS-CC","caseNumber":"CV-2024-00123","caseTitle":"Smith v. Jones","filingDate":"2024-03-15","caseType":"Civil","caseStatus":"Active","security":{"level":"PUBLIC"}}
{"caseId":"2024-CR-00456","courtCode":"ELLIS-DC","caseNumber":"CR-2024-00456","caseTitle":"State v. Johnson","filingDate":"2024-03-16","caseType":"Criminal","caseStatus":"Pending","security":{"level":"PUBLIC"}}
{"caseId":"2024-FM-00789","courtCode":"ELLIS-CC","caseNumber":"FM-2024-00789","caseTitle":"In re: Marriage of Davis","filingDate":"2024-03-17","caseType":"Family","caseStatus":"Active","security":{"level":"CONFIDENTIAL"}}
```

### Required Case Fields

| Field | Type | Description |
|-------|------|-------------|
| `caseId` | string | Unique case identifier in CMS (primary key) |
| `courtCode` | string | Court jurisdiction code (must match scope) |
| `caseNumber` | string | Public docket number |
| `caseTitle` | string | Case caption (e.g., "Smith v. Jones") |
| `filingDate` | date | Date case was filed (YYYY-MM-DD) |
| `caseType` | string | Case type (Civil, Criminal, Family, Probate, etc.) |
| `caseStatus` | string | Current case status (Active, Closed, Pending, etc.) |
| `security.level` | enum | One of: `PUBLIC`, `CONFIDENTIAL`, `SEALED` |

**Optional Case Fields:**

- `judgeId`, `judgeName` - Assigned judge information
- `dispositionDate`, `dispositionType` - Case outcome
- `courtLocation`, `courtDivision` - Court assignment
- `caseCategory`, `caseSubtype` - Additional classification
- `parties` - Array of party objects (or separate parties file)
- `attorneys` - Array of attorney objects

**Full schema documentation:** Contact re:Search team for complete schema specifications

### Additional File Types

**Filings File:**
- Contains individual filing/document events
- Links to parent case via `caseId`
- Includes filer information and filing dates

**Parties File:**
- Contains party and attorney information
- Links to case via `caseId`
- Includes party roles and contact information

**Documents File:**
- Contains document metadata
- Links to filings and cases
- Includes document types and access controls

## Validation and Error Handling

### Validation Stages

**1. Manifest Validation**

- Schema version compatibility check
- Required fields presence validation
- File reference integrity (all listed files exist)
- BatchId uniqueness check
- Court code authorization verification

**2. File Integrity Validation**

- SHA-256 checksum verification
- File size validation against manifest
- Compression format check (if compressed)
- File accessibility and readability

**3. Schema Validation**

- JSON syntax parsing (valid JSON Lines format)
- Required field validation for each record
- Data type verification (strings, integers, dates)
- Enum value validation (security levels, case types)
- Date format compliance (ISO 8601)

**4. Business Rule Validation**

- Court code authorization for records
- Date range reasonableness checks
- Cross-reference integrity (filings reference valid cases)
- Party role validations
- Document security consistency

### Error Reporting

Failed batches generate detailed error reports uploaded to the reports directory:

```
s3://<bucket>/<path>/_reports/batch-<batchId>-errors.json
```

**Example Error Report:**

```json
{
  "batchId": "ELLIS-2025-11-19-0100",
  "status": "FAILED",
  "processedAt": "2025-11-19T02:15:00Z",
  "errors": [
    {
      "file": "cases_0001.jsonl.gz",
      "line": 1523,
      "severity": "ERROR",
      "error": "Missing required field: caseNumber",
      "record": {"caseId": "2024-CV-01523", "courtCode": "ELLIS-CC"}
    },
    {
      "file": "filings_0001.jsonl.gz",
      "line": 892,
      "severity": "ERROR",
      "error": "Invalid caseId reference: case 2024-CV-99999 not found in cases file",
      "record": {"filingId": "FIL-892", "caseId": "2024-CV-99999"}
    },
    {
      "file": "cases_0001.jsonl.gz",
      "line": 3456,
      "severity": "WARNING",
      "error": "Date in future: filingDate is after current date",
      "record": {"caseId": "2024-CV-03456", "filingDate": "2025-12-01"}
    }
  ],
  "summary": {
    "totalRecords": 165000,
    "validRecords": 164997,
    "errorRecords": 2,
    "warningRecords": 1,
    "processingTimeSeconds": 245
  }
}
```

**Error Severity Levels:**

- **ERROR** - Record rejected, not indexed
- **WARNING** - Record indexed with potential issues flagged
- **INFO** - Informational message, no action required

## Implementation Checklist

### For CMS Vendors

- [ ] **Obtain credentials** from re:Search team (S3 IAM keys or SFTP SSH keys)
- [ ] **Review schema documentation** (request from re:Search team)
- [ ] **Set up development environment** with test credentials
- [ ] **Develop export logic** to query case data from CMS database
- [ ] **Implement JSON Lines writer** with schema compliance
- [ ] **Build manifest generator** with SHA-256 checksum calculation
- [ ] **Test file compression** (gzip recommended for bandwidth)
- [ ] **Implement upload automation** (AWS CLI, SDK, or SFTP scripts)
- [ ] **Create pre-upload validation** to catch errors early
- [ ] **Set up monitoring** for export success/failure
- [ ] **Configure scheduling** (cron, task scheduler, or job orchestration)
- [ ] **Implement error handling** and retry logic
- [ ] **Document runbook** for operations team
- [ ] **Test with sample data** in test environment
- [ ] **Perform load testing** with production-sized datasets
- [ ] **Create rollback procedures** for failed batches

### For Courts

- [ ] **Coordinate with CMS vendor** on batch schedule and scope
- [ ] **Provide court codes** and jurisdiction mapping to vendor
- [ ] **Review data scope** (which case types, date ranges to include)
- [ ] **Plan historical backfill** if loading legacy data
- [ ] **Establish change window** for initial load (minimize disruption)
- [ ] **Designate technical contact** for troubleshooting and support
- [ ] **Schedule testing period** with re:Search team
- [ ] **Review security requirements** for sealed/confidential cases
- [ ] **Plan for public access** validation after go-live
- [ ] **Establish success metrics** and monitoring procedures
- [ ] **Train staff** on new processes and workflows
- [ ] **Communicate with stakeholders** about timeline and expectations

## Monitoring and Operations

### Success Metrics

Monitor these key performance indicators:

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| **Batch completion rate** | 99%+ | < 95% |
| **Record validation rate** | 99.9%+ | < 99% |
| **Processing latency** | < 2 hours | > 4 hours |
| **Error resolution time** | < 24 hours | > 48 hours |
| **Storage utilization** | < 80% | > 90% |
| **Upload success rate** | 99%+ | < 95% |

### Operational Dashboard (Coming Soon)

The re:Search team is developing a self-service dashboard showing:

- **Recent batch history** - Last 30 days of batches with status
- **Success/failure status** - Visual indicators for each batch
- **Record counts** - Cases, filings, parties, documents processed
- **Processing times** - How long each batch took
- **Error summaries** - Aggregated error types and counts
- **Trend analysis** - Historical performance metrics

**Current Monitoring:** Email notifications sent to vendor contact on batch completion/failure

### Log Retention

- Batch files retained for 90 days
- Error reports retained for 1 year
- Audit logs retained per court policy
- Successful batches archived after indexing

## Troubleshooting

### Common Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| **Batch not detected** | Incorrect file path or missing manifest | Verify upload location matches agreed path structure; ensure manifest uploaded last |
| **Checksum mismatch** | File corruption during transfer | Re-upload affected file; verify file integrity before upload |
| **Schema validation errors** | Invalid JSON or missing required fields | Run pre-upload validator; review schema documentation |
| **Permission denied (S3)** | IAM credentials expired or incorrect | Verify access key and secret; request credential rotation if needed |
| **Connection refused (SFTP)** | IP not whitelisted or wrong credentials | Provide source IP to re:Search team; verify SSH key |
| **Duplicate batchId** | Re-running same batch without new ID | Use unique batchId for each attempt; include sequence number |
| **Manifest file too large** | Too many files listed | Split batch into multiple smaller batches |
| **Processing timeout** | Batch too large or complex | Split into smaller batches; compress files; optimize JSON structure |
| **Missing cross-references** | Filings reference non-existent cases | Ensure cases file processed before filings; validate references |

### Debug Checklist

When troubleshooting batch failures:

1. **Check error report** in `_reports/` directory for detailed errors
2. **Verify manifest** structure and field values
3. **Validate checksums** match file contents
4. **Test JSON parsing** locally before upload
5. **Review file permissions** and accessibility
6. **Check network connectivity** to S3/SFTP
7. **Verify credentials** are current and valid
8. **Inspect logs** for upload and processing
9. **Contact support** with batchId and error report

### Getting Help

**For Batch Processing Issues:**

- Check error report in `_reports/` directory first
- Review [Troubleshooting Guide](../../troubleshooting/README.md)
- Contact your court liaison or technical project manager
- Include batchId, error report, and manifest in support request
- Provide sample data files (sanitized) if needed

**For Credential Issues:**

- Email re:Search technical support
- Request credential rotation or permission adjustment
- Verify IAM policy or SSH key configuration
- Test connectivity with provided credentials

**Support Resources:**

- Technical support portal (if available)
- Email: research-support@courts.state
- Phone support during business hours
- Escalation procedures for critical issues

## Next Steps

### Ready to Implement Batch Mode?

Follow these steps to begin implementation:

1. **Contact Your Court Liaison**
   - Schedule a kickoff meeting
   - Discuss implementation timeline
   - Review resource requirements
   - Establish project milestones

2. **Request Schema Documentation**
   - Obtain complete JSON schema specifications
   - Review sample files and manifests
   - Understand required and optional fields
   - Clarify court-specific requirements

3. **Review Onboarding Guide**
   - Follow [Onboarding Process](../onboarding/README.md)
   - Complete [Before You Begin](../onboarding/01-before-you-begin.md) checklist
   - Review [Technical Setup](../onboarding/04-technical-setup.md) guidance

4. **Obtain Test Credentials**
   - Request S3 or SFTP access for test environment
   - Set up credential storage and management
   - Test connectivity and upload procedures

5. **Begin Development**
   - Implement export logic in your CMS
   - Create manifest generation scripts
   - Build upload automation
   - Develop error handling and monitoring

6. **Test Thoroughly**
   - Upload sample batches to test environment
   - Validate processing and indexing
   - Review error reports and fix issues
   - Perform load testing with production-sized data

7. **Go Live**
   - Complete [Testing & Certification](../onboarding/05-testing-certification.md)
   - Schedule production cutover
   - Monitor initial batches closely
   - Adjust schedule and process as needed

**Have Questions?**

Contact re:Search technical support for assistance with implementation.

## Related Documentation

### Integration Modes
- [Integration Modes Overview](./README.md) – Compare all integration methods
- [ECF Mode](./ecf-mode.md) – Real-time API-driven integration
- [CIP Mode](./cip-mode.md) – Court Integration Platform for legacy systems
- [Non-Integrated Mode](./non-integrated-mode.md) – Manual portal-based processing

### Technical Resources
- [API Reference](../../api/README.md) – For ECF/CIP mode API details
- [Standards Guide](../../standards/README.md) – Data formats and requirements
- [Troubleshooting Guide](../../troubleshooting/README.md) – Common issues and solutions
- [FAQ](../../reference/faq.md) – Frequently asked questions

### Onboarding
- [Onboarding Guide](../onboarding/README.md) – Implementation process
- [Before You Begin](../onboarding/01-before-you-begin.md) – Prerequisites checklist
- [Environment Access](../onboarding/02-environment-access.md) – Getting credentials
- [CMS Preparation](../onboarding/03-cms-preparation.md) – System readiness
- [Technical Setup](../onboarding/04-technical-setup.md) – Development guidance
- [Testing & Certification](../onboarding/05-testing-certification.md) – Validation requirements

---

**Last Updated:** November 2025