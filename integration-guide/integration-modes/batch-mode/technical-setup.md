# Technical Setup

[← Back to Batch Mode Overview](./README.md)

## Quick Navigation
- [Transport and Authentication](#transport-and-authentication)
- [Manifest Requirements](#manifest-requirements)
- [Data Schema](#data-schema)
- [Validation Rules](#validation-rules)

---

## Transport and Authentication

Batch Mode uses **secure file delivery** rather than API authentication. Two transport methods are supported:

### AWS S3 (Recommended)

#### Why S3?

- **Faster transfer speeds** – Optimized for large file uploads
- **Built-in encryption** – AES-256 encryption at rest
- **Automatic retry** – Built-in error handling and retry logic
- **Event-driven processing** – Instant batch detection and processing
- **Integrated monitoring** – CloudWatch integration for operational visibility
- **Scalability** – Handles any file size without configuration
- **Cost-effective** – Pay only for storage used

#### Security Features

- IAM role-based access control
- Bucket policies restrict access to specific courts/vendors
- TLS 1.2+ encryption in transit
- Server-side encryption at rest (SSE-S3 or SSE-KMS)
- S3 versioning for audit trail
- MFA delete protection for production buckets

#### S3 Path Structure

```text
s3://tx-research-ingest-prod/<CMSVendor>/<CourtCode>/<YYYY-MM-DD>/
  ├── manifest.json
  ├── cases_0001.jsonl.gz
  ├── cases_0002.jsonl.gz
  ├── filings_0001.jsonl.gz
  ├── parties_0001.jsonl.gz
  └── documents_0001.jsonl.gz
```

**Path Components:**
- `<CMSVendor>`: Your CMS vendor name (e.g., "MyVendor", "Tyler", "Custom")
- `<CourtCode>`: Court identifier (e.g., "ELLIS-CC", "TRAVIS-DC")
- `<YYYY-MM-DD>`: Batch date in ISO format (e.g., "2025-11-19")

#### Upload Methods

**AWS CLI:**
```bash
# Upload entire batch directory
aws s3 cp batch/ s3://tx-research-ingest-prod/MyVendor/ELLIS-CC/2025-11-19/ --recursive

# Upload individual files
aws s3 cp manifest.json s3://tx-research-ingest-prod/MyVendor/ELLIS-CC/2025-11-19/manifest.json
aws s3 cp cases_0001.jsonl.gz s3://tx-research-ingest-prod/MyVendor/ELLIS-CC/2025-11-19/cases_0001.jsonl.gz
```

**Python (boto3):**
```python
import boto3

s3 = boto3.client('s3')
bucket = 'tx-research-ingest-prod'
prefix = 'MyVendor/ELLIS-CC/2025-11-19/'

# Upload data files first
s3.upload_file('cases_0001.jsonl.gz', bucket, f'{prefix}cases_0001.jsonl.gz')
s3.upload_file('filings_0001.jsonl.gz', bucket, f'{prefix}filings_0001.jsonl.gz')
s3.upload_file('parties_0001.jsonl.gz', bucket, f'{prefix}parties_0001.jsonl.gz')

# Upload manifest last to signal batch is ready
s3.upload_file('manifest.json', bucket, f'{prefix}manifest.json')
```

**Java SDK:**
```java
AmazonS3 s3Client = AmazonS3ClientBuilder.defaultClient();
String bucket = "tx-research-ingest-prod";
String prefix = "MyVendor/ELLIS-CC/2025-11-19/";

// Upload data files
s3Client.putObject(bucket, prefix + "cases_0001.jsonl.gz", new File("cases_0001.jsonl.gz"));
s3Client.putObject(bucket, prefix + "filings_0001.jsonl.gz", new File("filings_0001.jsonl.gz"));

// Upload manifest last
s3Client.putObject(bucket, prefix + "manifest.json", new File("manifest.json"));
```

#### S3 Credentials

- Provided by re:Search team during onboarding
- IAM access key + secret key
- Rotate every 90 days (automated reminder sent)
- Scoped to specific buckets only
- Never commit credentials to source control

**Credential Storage Best Practices:**
```bash
# Store in environment variables
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"

# Or use AWS credentials file (~/.aws/credentials)
[research-batch]
aws_access_key_id = your-access-key
aws_secret_access_key = your-secret-key
region = us-east-1
```

### SFTP (Alternative)

#### When to Use SFTP

- Your CMS cannot integrate with AWS S3
- Corporate policy restricts cloud storage
- Existing SFTP automation already in place
- Air-gapped environments requiring file-based transfer

#### Security Features

- SSH key-based authentication (password auth disabled)
- Chroot jail isolation per court
- TLS encryption for all transfers
- Connection logging and audit trails
- IP whitelisting for source systems

#### SFTP Configuration

```text
Host: sftp.research.courts.state
Port: 22
Path: /research-ingest/<CMSVendor>/<CourtCode>/<YYYY-MM-DD>/
Authentication: SSH key (provided by re:Search team)
```

#### Upload Example

```bash
# Using sftp command (interactive)
sftp -i ~/.ssh/research_key user@sftp.research.courts.state
cd /research-ingest/MyVendor/ELLIS-CC/2025-11-19/
put manifest.json
put cases_*.jsonl.gz
put filings_*.jsonl.gz
put parties_*.jsonl.gz
put documents_*.jsonl.gz
bye

# Using scp for automation
scp -i ~/.ssh/research_key batch/* \
  user@sftp.research.courts.state:/research-ingest/MyVendor/ELLIS-CC/2025-11-19/
```

#### SFTP Best Practices

- Test connectivity before production
- Implement retry logic for network failures
- Monitor transfer completion (check exit codes)
- Verify file sizes after upload
- Clean up old batches periodically (90-day retention)
- Restrict SSH key permissions (`chmod 600`)

#### SFTP Credentials

- SSH private key provided via secure channel
- Public key fingerprint verification required
- Key rotation every 180 days
- Store keys securely (encrypted, restricted permissions)
- Never share keys between environments

---

## Manifest Requirements

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

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `batchId` | string | Unique identifier for this batch | `"ELLIS-2025-11-19-0100"` |
| `schemaVersion` | string | Must match current schema version | `"1.3.0"` |
| `generatedAt` | ISO 8601 | Timestamp when batch was created (UTC) | `"2025-11-19T01:05:12Z"` |
| `files` | array | List of all data files with metadata | See file object below |
| `files[].type` | enum | File content type | `"cases"`, `"filings"`, `"parties"`, `"documents"` |
| `files[].path` | string | Relative filename | `"cases_0001.jsonl.gz"` |
| `files[].recordCount` | integer | Number of JSON objects in file | `50000` |
| `files[].sha256` | string | SHA-256 checksum (lowercase hex) | `"a3d8f9e7..."` |
| `files[].sizeBytes` | integer | File size in bytes | `15728640` |
| `scope.courtCodes` | array | Court identifiers in this batch | `["ELLIS-CC", "ELLIS-DC"]` |
| `scope.asOfDate` | date | Effective date of data snapshot | `"2025-11-18"` |

### Optional Manifest Fields

| Field | Type | Description |
|-------|------|-------------|
| `batchType` | enum | `"FULL"` (complete snapshot) or `"DELTA"` (changes only) |
| `scope.includesHistorical` | boolean | `true` if batch contains backfilled historical data |
| `contact.vendor` | string | CMS vendor name for support |
| `contact.email` | string | Support contact email |
| `contact.phone` | string | Support contact phone |
| `notes` | string | Additional information about this batch |

### Generating SHA-256 Checksums

**Linux/macOS:**
```bash
sha256sum cases_0001.jsonl.gz
# Output: a3d8f9e7c2b1... cases_0001.jsonl.gz
```

**Windows (PowerShell):**
```powershell
Get-FileHash cases_0001.jsonl.gz -Algorithm SHA256
```

**Python:**
```python
import hashlib

def calculate_sha256(filepath):
    sha256_hash = hashlib.sha256()
    with open(filepath, "rb") as f:
        for byte_block in iter(lambda: f.read(4096), b""):
            sha256_hash.update(byte_block)
    return sha256_hash.hexdigest()

checksum = calculate_sha256("cases_0001.jsonl.gz")
print(checksum)
```

**Java:**
```java
import java.security.MessageDigest;
import java.io.FileInputStream;

public static String calculateSHA256(String filepath) throws Exception {
    MessageDigest digest = MessageDigest.getInstance("SHA-256");
    FileInputStream fis = new FileInputStream(filepath);
    byte[] buffer = new byte[8192];
    int read;
    while ((read = fis.read(buffer)) > 0) {
        digest.update(buffer, 0, read);
    }
    fis.close();
    
    byte[] hash = digest.digest();
    StringBuilder hexString = new StringBuilder();
    for (byte b : hash) {
        String hex = Integer.toHexString(0xff & b);
        if (hex.length() == 1) hexString.append('0');
        hexString.append(hex);
    }
    return hexString.toString();
}
```

---

## Data Schema

Each data file must be in **JSON Lines format** (one JSON object per line).

### Cases File Schema

**File: cases_0001.jsonl**

```json
{"caseId":"2024-CV-00123","courtCode":"ELLIS-CC","caseNumber":"CV-2024-00123","caseTitle":"Smith v. Jones","filingDate":"2024-03-15","caseType":"Civil","caseStatus":"Active","security":{"level":"PUBLIC"}}
{"caseId":"2024-CR-00456","courtCode":"ELLIS-DC","caseNumber":"CR-2024-00456","caseTitle":"State v. Johnson","filingDate":"2024-03-16","caseType":"Criminal","caseStatus":"Pending","security":{"level":"PUBLIC"}}
```

#### Required Case Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `caseId` | string | Unique case identifier in CMS | `"2024-CV-00123"` |
| `courtCode` | string | Court jurisdiction code | `"ELLIS-CC"` |
| `caseNumber` | string | Public docket number | `"CV-2024-00123"` |
| `caseTitle` | string | Case caption | `"Smith v. Jones"` |
| `filingDate` | date | Date case was filed (YYYY-MM-DD) | `"2024-03-15"` |
| `caseType` | string | Case type | `"Civil"`, `"Criminal"`, `"Family"` |
| `caseStatus` | string | Current case status | `"Active"`, `"Closed"`, `"Pending"` |
| `security.level` | enum | Access level | `"PUBLIC"`, `"CONFIDENTIAL"`, `"SEALED"` |

#### Optional Case Fields

- `judgeId`, `judgeName` – Assigned judge information
- `dispositionDate`, `dispositionType` – Case outcome details
- `courtLocation`, `courtDivision` – Court assignment
- `caseCategory`, `caseSubtype` – Additional classification
- `parties` – Array of party objects (or use separate parties file)
- `attorneys` – Array of attorney objects

### Filings File Schema

**File: filings_0001.jsonl**

Contains individual filing/document events:
- Links to parent case via `caseId`
- Includes filer information and filing dates
- Documents filing types and statuses

### Parties File Schema

**File: parties_0001.jsonl**

Contains party and attorney information:
- Links to case via `caseId`
- Includes party roles (plaintiff, defendant, attorney, etc.)
- Contact information (if applicable)

### Documents File Schema

**File: documents_0001.jsonl**

Contains document metadata:
- Links to filings and cases
- Includes document types and access controls
- File locations and checksums (if binary files uploaded separately)

**Note:** For complete schema specifications including all optional fields and nested objects, contact the re:Search team for the full schema documentation package.

---

## Validation Rules

re:Search validates batches in multiple stages to ensure data quality.

### Stage 1: Manifest Validation

**Checks Performed:**
- Schema version compatibility (must be `1.3.0` currently)
- Required fields presence
- File reference integrity (all listed files must exist)
- BatchId uniqueness (no duplicate batch IDs)
- Court code authorization (your credentials allow these courts)

**Common Errors:**
- Missing required fields in manifest
- Files listed in manifest but not uploaded
- Duplicate batchId from previous submission
- Unauthorized court codes

### Stage 2: File Integrity Validation

**Checks Performed:**
- SHA-256 checksum verification (matches manifest)
- File size validation (matches manifest)
- Compression format check (valid gzip if `.gz` extension)
- File accessibility and readability

**Common Errors:**
- Checksum mismatch (file corrupted during upload)
- File size mismatch
- Invalid gzip compression
- File permissions issues

### Stage 3: Schema Validation

**Checks Performed:**
- JSON syntax parsing (valid JSON Lines format)
- Required field validation for each record
- Data type verification (strings, integers, dates, booleans)
- Enum value validation (security levels, case types, statuses)
- Date format compliance (ISO 8601: YYYY-MM-DD)

**Common Errors:**
- Invalid JSON syntax (missing quotes, commas, brackets)
- Missing required fields (caseId, courtCode, caseNumber, etc.)
- Wrong data types (string where integer expected)
- Invalid enum values (security level "private" instead of "CONFIDENTIAL")
- Invalid date formats (03/15/2024 instead of 2024-03-15)

### Stage 4: Business Rule Validation

**Checks Performed:**
- Court code authorization (records match manifest scope)
- Date range reasonableness (filing dates not in future, reasonable history)
- Cross-reference integrity (filings reference valid cases in same batch)
- Party role validations (valid party types)
- Document security consistency (document security ≤ case security)

**Common Errors:**
- Filing references case not in batch or database
- Future filing dates
- Sealed document in public case
- Invalid party role codes

### Error Severity Levels

- **ERROR** – Record rejected, not indexed, must be corrected
- **WARNING** – Record indexed but flagged for review
- **INFO** – Informational message, no action required

### Error Reports

Failed batches generate detailed error reports:

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
      "error": "Invalid caseId reference: case 2024-CV-99999 not found",
      "record": {"filingId": "FIL-892", "caseId": "2024-CV-99999"}
    }
  ],
  "summary": {
    "totalRecords": 165000,
    "validRecords": 164998,
    "errorRecords": 2,
    "warningRecords": 0,
    "processingTimeSeconds": 245
  }
}
```

---

## Pre-Upload Validation (Recommended)

To catch errors before upload, implement local validation:

### Validation Checklist

**Before Upload:**
- [ ] All JSON files are valid JSON Lines format
- [ ] Required fields present in all records
- [ ] Date fields use ISO 8601 format (YYYY-MM-DD)
- [ ] Security levels use valid enum values
- [ ] Court codes match authorized list
- [ ] File compression successful (if using gzip)
- [ ] Manifest includes all data files
- [ ] SHA-256 checksums calculated correctly
- [ ] File sizes in manifest match actual files
- [ ] BatchId is unique and follows naming convention

### Validation Tools

**JSON Lines Validator (Python):**
```python
import json

def validate_jsonl(filepath):
    errors = []
    with open(filepath, 'r') as f:
        for line_num, line in enumerate(f, 1):
            try:
                record = json.loads(line.strip())
                # Add your field validation here
                if 'caseId' not in record:
                    errors.append(f"Line {line_num}: Missing caseId")
            except json.JSONDecodeError as e:
                errors.append(f"Line {line_num}: Invalid JSON - {e}")
    return errors
```

**Schema Validator:**
Request the JSON Schema definitions from the re:Search team and use standard validators like:
- Python: `jsonschema` library
- Java: `everit-org/json-schema`
- Node.js: `ajv` (Another JSON Schema Validator)

---

## Next Steps

- **Review implementation process** → Read [Implementation Guide](./implementation-guide.md)
- **Learn about monitoring** → Read [Operations](./operations.md)
- **Understand the data flow** → Read [How It Works](./how-it-works.md)
- **Return to overview** → [Batch Mode Overview](./README.md)

---

**Last Updated:** November 2025