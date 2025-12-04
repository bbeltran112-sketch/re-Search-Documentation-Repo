# How Batch Mode Works

[← Back to Batch Mode Overview](./README.md)

## Quick Navigation
- [Integration Flow](#integration-flow)
- [Step-by-Step Process](#step-by-step-process)
- [File Formats and Structure](#file-formats-and-structure)
- [What Happens to Your Data](#what-happens-to-your-data)

---

## Integration Flow

Batch Mode operates as a **one-way file delivery system** where your CMS generates complete or incremental snapshots of case data and uploads them to a designated storage location. re:Search automatically detects new batches, validates their contents, and indexes the data.

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

---

## Step-by-Step Process

### Phase 1: Data Export (CMS Responsibility)

#### 1. Export Generation
Your CMS queries case data from its database and prepares it for export:
- Query cases, filings, parties, documents based on date range or criteria
- Apply court-specific filtering (jurisdiction, case types, security levels)
- Format data according to Batch Mode schema specifications
- Handle incremental vs. full snapshot logic

**Example Query Logic:**
```sql
-- Example: Get cases filed/updated in last 24 hours
SELECT * FROM cases 
WHERE court_id = 'ELLIS-CC' 
  AND (filed_date >= DATEADD(day, -1, GETDATE()) 
       OR modified_date >= DATEADD(day, -1, GETDATE()))
```

#### 2. File Creation
Data is written to JSON Lines files:
- **One JSON object per line** (no commas between objects, no outer array)
- Large datasets split into multiple files (recommended: 50,000 records per file)
- Each entity type in separate files (cases, filings, parties, documents)
- Files can be compressed (gzip recommended for ~70% size reduction)

**Example File:**
```json
{"caseId":"2024-CV-00123","courtCode":"ELLIS-CC","caseNumber":"CV-2024-00123","caseTitle":"Smith v. Jones","filingDate":"2024-03-15","caseType":"Civil","caseStatus":"Active","security":{"level":"PUBLIC"}}
{"caseId":"2024-CR-00456","courtCode":"ELLIS-DC","caseNumber":"CR-2024-00456","caseTitle":"State v. Johnson","filingDate":"2024-03-16","caseType":"Criminal","caseStatus":"Pending","security":{"level":"PUBLIC"}}
```

#### 3. Manifest Creation
A `manifest.json` file catalogs all data files:
- Lists each file with metadata (type, path, record count)
- Includes SHA-256 checksums for integrity verification
- Documents batch metadata (ID, timestamp, court codes)
- Acts as the "table of contents" for the batch

#### 4. Compression (Optional but Recommended)
Files are compressed using gzip:
- Reduces transfer time by ~70%
- Maintains data integrity during compression
- Standard `.jsonl.gz` extension

#### 5. Upload
All files uploaded to designated storage:
- Upload to S3 bucket or SFTP directory
- Use atomic upload to ensure complete batch delivery
- **Upload manifest last** to signal batch is ready for processing

### Phase 2: Detection and Validation (re:Search Responsibility)

#### 6. Batch Detection
re:Search monitors storage location for new batches:
- S3: Event-driven notification when new manifest appears
- SFTP: Periodic polling (every 5-15 minutes)
- Processing begins immediately upon detection

#### 7. Manifest Validation
First validation layer checks manifest integrity:
- Schema version compatibility
- Required fields presence
- File reference integrity (all listed files exist)
- BatchId uniqueness
- Court code authorization

#### 8. File Integrity Verification
Files are downloaded and verified:
- SHA-256 checksums compared against manifest
- File sizes validated
- Compression format checked
- Files extracted if compressed

### Phase 3: Processing and Indexing (re:Search Responsibility)

#### 9. Schema Validation
Each record validated against schema:
- JSON syntax parsing
- Required field validation
- Data type verification (strings, integers, dates)
- Enum value validation (security levels, case types)
- Date format compliance (ISO 8601)

#### 10. Business Rule Validation
Additional validation logic applied:
- Court code authorization for records
- Date range reasonableness checks
- Cross-reference integrity (filings reference valid cases)
- Party role validations
- Document security consistency

#### 11. Indexing
Valid records are indexed and made searchable:
- Records inserted into search index
- Relationships established (cases → filings → documents)
- Security filters applied
- Full-text search enabled
- **Atomic operation** – batch either fully succeeds or fully rolls back

#### 12. Reporting
Processing results documented:
- Success/failure status
- Record counts (total, valid, errors, warnings)
- Detailed error report generated for failed records
- Report uploaded to reports directory
- Email notification sent to vendor contact

#### 13. Data Available
Successfully indexed data becomes searchable:
- Public users can search case metadata
- Security filters enforced based on record-level settings
- Data appears in re:Search portal immediately

---

## File Formats and Structure

### Batch Directory Structure

```
s3://tx-research-ingest-prod/MyVendor/ELLIS-CC/2025-11-19/
├── manifest.json              # Required: Batch metadata and file catalog
├── cases_0001.jsonl.gz       # Case records (compressed)
├── cases_0002.jsonl.gz       # More case records if needed
├── filings_0001.jsonl.gz     # Filing records
├── parties_0001.jsonl.gz     # Party and attorney records
└── documents_0001.jsonl.gz   # Document metadata records
```

### File Naming Conventions

- **Manifest:** Always `manifest.json` (exact name required)
- **Data files:** `<type>_<sequence>.jsonl[.gz]`
  - Type: `cases`, `filings`, `parties`, or `documents`
  - Sequence: 4-digit zero-padded number (0001, 0002, etc.)
  - Extension: `.jsonl` (uncompressed) or `.jsonl.gz` (compressed)

### JSON Lines Format

**Key Characteristics:**
- One JSON object per line
- No commas between objects
- No outer array wrapper
- Each line is independently parseable
- Newline character (LF) as delimiter

**Why JSON Lines?**
- Streamable: Process files without loading entire dataset into memory
- Fault-tolerant: Parse errors affect only one line, not entire file
- Appendable: Easy to generate incrementally
- Tool-friendly: Many tools support JSON Lines natively

---

## What Happens to Your Data

### Data Lifecycle

1. **Generation:** Your CMS creates files from database queries
2. **Staging:** Files written to local disk and compressed
3. **Upload:** Files transferred to S3/SFTP (encrypted in transit)
4. **Validation:** Files verified and parsed by re:Search
5. **Indexing:** Valid records inserted into search index
6. **Availability:** Data becomes searchable in re:Search portal
7. **Retention:** Original batch files retained for 90 days
8. **Archival:** Successful batches archived after indexing

### Data Freshness

| Batch Schedule | Data Freshness | Typical Use Case |
|----------------|----------------|------------------|
| **Hourly** | 0-60 minutes old | High-volume courts needing near-real-time |
| **Nightly** | 0-24 hours old | Most common; balances freshness and overhead |
| **Weekly** | 0-7 days old | Small courts with minimal changes |
| **On-demand** | Variable | Historical backfills, special migrations |

### Data Completeness

**Full Snapshots:**
- Entire case database exported each batch
- Ensures consistency but higher overhead
- Best for: Initial loads, small courts, monthly updates

**Delta Batches (Coming Soon):**
- Only changed/new records since last batch
- Much smaller file sizes
- Best for: High-volume courts, frequent updates

### Error Handling

**Record-Level Errors:**
- Invalid records rejected and logged
- Valid records from same batch still indexed
- Error report details why each record failed
- Re-upload corrected records in next batch

**Batch-Level Errors:**
- Critical errors (manifest issues, missing files) reject entire batch
- No partial indexing occurs
- Original data remains unchanged
- Fix issues and re-upload entire batch

---

## Performance Characteristics

### Typical Processing Times

| Batch Size | Processing Time | Notes |
|------------|-----------------|-------|
| 10,000 records | 2-5 minutes | Small court nightly update |
| 50,000 records | 5-15 minutes | Medium court nightly update |
| 100,000 records | 15-30 minutes | Large court nightly update |
| 1,000,000 records | 2-4 hours | Historical backfill |
| 10,000,000 records | 8-12 hours | Major migration project |

**Factors Affecting Speed:**
- Record complexity (number of fields, nested objects)
- File compression ratio
- Network bandwidth
- Validation rule complexity
- Current system load

### Scalability

Batch Mode has been proven at scale:
- ✅ Single batches up to 10+ million records
- ✅ Daily batches processing 500,000+ records
- ✅ Multiple courts processed in parallel
- ✅ Consistent performance across varying loads

---

## Next Steps

- **Understand transport options** → Read [Technical Setup](./technical-setup.md)
- **Plan your implementation** → Read [Implementation Guide](./implementation-guide.md)
- **Learn about operations** → Read [Operations](./operations.md)
- **Return to overview** → [Batch Mode Overview](./README.md)

---

**Last Updated:** November 2025