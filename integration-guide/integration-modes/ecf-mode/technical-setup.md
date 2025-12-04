# Technical Setup

[← Back to ECF Mode Overview](./README.md)

## Quick Navigation
- [APIs You'll Implement](#apis-youll-implement)
- [Infrastructure Requirements](#infrastructure-requirements)
- [Security Requirements](#security-requirements)
- [What CMS Must Provide](#what-cms-must-provide)

---

## APIs You'll Implement

ECF Mode requires your CMS to implement specific SOAP web service APIs. Some you'll call (outbound), others you'll expose (inbound).

### API Overview

| API | Direction | Required? | Purpose |
|-----|-----------|-----------|---------|
| **NotifyCaseEvent** | CMS → re:Search | Yes | Notify when case changes |
| **GetCase** | re:Search → CMS | Yes | Retrieve current case data |
| **RecordFiling** | EFM → CMS | Yes (if e-filing) | Receive e-filing from EFM |
| **NotifyDocketingComplete** | CMS → EFM | Yes (if e-filing) | Confirm filing docketed |
| **GetDocument** | re:Search → CMS | Optional | Retrieve document binary |

---

## NotifyCaseEvent (CMS → re:Search)

### Purpose
Notify re:Search that a case has changed so it can retrieve fresh data.

### When to Send
**Send immediately after:**
- Party or attorney added/changed/removed
- Case security changed (sealed/unsealed) ⚠️ CRITICAL
- Document security changed ⚠️ CRITICAL
- Disposition entered
- Judge or division reassigned
- Document added outside e-filing workflow
- Case status changed
- Any other material case change

**CRITICAL:** Security events (CaseSecurity, DocumentSecurity) must be sent immediately - never batch or delay these.

### Request Format

**Endpoint:** Provided by Tyler (e.g., `https://researchXX.tylerhost.net/NotifyCaseEvent`)

**Example:**
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <NotifyCaseEvent xmlns="urn:tyler:ecf:research">
      <CaseTrackingID>CV-2024-001234</CaseTrackingID>
      <EventType>CaseParty</EventType>
    </NotifyCaseEvent>
  </soap:Body>
</soap:Envelope>
```

### Event Types

| Event Type | When to Use | What Gets Refreshed |
|------------|-------------|---------------------|
| `CaseParty` | Party added/changed | Party list |
| `CasePartyAttorney` | Attorney added/changed | Attorney list and associations |
| `CaseSecurity` | Case sealed/unsealed | Security level, access controls |
| `DocumentSecurity` | Document sealed | Document access controls |
| `CaseDisposition` | Disposition entered | Disposition data, case status |
| `CaseReassigned` | Judge/division changed | Judge assignment |
| `DocumentFiling` | Document added (non-e-filing) | Document list |
| `CaseHearing` | Hearing scheduled | Event/hearing list |
| `CaseStatus` | Status changed | Case status |
| `CaseInitiated` | New case created | Entire case (initial load) |

[Complete event type reference →](../../implementation-playbook/event-type-logic.md)

### Response

**Success:**
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <NotifyCaseEventResponse xmlns="urn:tyler:ecf:research">
      <Success>true</Success>
    </NotifyCaseEventResponse>
  </soap:Body>
</soap:Envelope>
```

**HTTP 200** indicates success - re:Search will call GetCase to retrieve fresh data.

### Error Handling

**If NotifyCaseEvent fails:**
1. Log error with case ID and event type
2. Retry with exponential backoff (1s, 2s, 4s, 8s, 16s)
3. Alert operations team if retries exhausted
4. Don't block CMS operations waiting for response

**Best Practice:** Use asynchronous message queue for reliability:
```
CMS transaction commits → Message to queue → Background worker sends event
```

### Implementation Example (C#)

```csharp
public async Task SendCaseEventAsync(string caseId, string eventType)
{
    var client = new ResearchClient();
    var request = new NotifyCaseEventRequest
    {
        CaseTrackingID = caseId,
        EventType = eventType
    };
    
    try
    {
        var response = await client.NotifyCaseEventAsync(request);
        _logger.LogInformation($"Event sent: {caseId} - {eventType}");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, $"Failed to send event: {caseId} - {eventType}");
        // Queue for retry
        await _retryQueue.EnqueueAsync(caseId, eventType);
    }
}
```

---

## GetCase (re:Search → CMS)

### Purpose
Retrieve complete current case data when re:Search receives a notification.

### When Called
- After receiving NotifyCaseEvent
- After receiving NotifyDocketingComplete (from EFM)
- On-demand (rarely, for troubleshooting)

### Request Format

**Endpoint:** Your CMS must expose this (e.g., `https://yourcms.court.gov/services/GetCase`)

**Example:**
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetCaseRequest xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:GetCase-4.0">
      <CaseTrackingID>CV-2024-001234</CaseTrackingID>
    </GetCaseRequest>
  </soap:Body>
</soap:Envelope>
```

### Response Format

**Must include:**
- Case metadata (number, title, type, status, dates)
- All parties with roles and contact information
- All attorneys with party associations and bar numbers
- All documents with metadata (NO binaries)
- Current security settings (case-level and document-level)
- Docket entries
- Case events/hearings

**Example (abbreviated):**
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetCaseResponse xmlns="urn:oasis:names:tc:legalxml-courtfiling:schema:xsd:GetCase-4.0">
      <Case>
        <CaseTrackingID>CV-2024-001234</CaseTrackingID>
        <CaseNumber>CV-2024-001234</CaseNumber>
        <CaseTitle>Smith v. Jones</CaseTitle>
        <FilingDate>2024-03-15</FilingDate>
        <CaseType>Civil</CaseType>
        <CaseStatus>Active</CaseStatus>
        <Security>Public</Security>
        
        <CaseParticipant>
          <EntityPerson>
            <PersonName>
              <FirstName>John</FirstName>
              <LastName>Smith</LastName>
            </PersonName>
          </EntityPerson>
          <CasePartyRole>Plaintiff</CasePartyRole>
        </CaseParticipant>
        
        <CaseParticipant>
          <EntityPerson>
            <PersonName>
              <FirstName>Jane</FirstName>
              <LastName>Jones</LastName>
            </PersonName>
          </EntityPerson>
          <CasePartyRole>Defendant</CasePartyRole>
        </CaseParticipant>
        
        <CaseAttorney>
          <AttorneyID>ATT-123</AttorneyID>
          <PersonName>
            <FirstName>Robert</FirstName>
            <LastName>Attorney</LastName>
          </PersonName>
          <BarNumber>12345678</BarNumber>
          <RepresentsParty>
            <PartyReference>John Smith</PartyReference>
          </RepresentsParty>
        </CaseAttorney>
        
        <FiledDocument>
          <DocumentID>DOC-001</DocumentID>
          <DocumentTitle>Complaint</DocumentTitle>
          <FilingDate>2024-03-15</FilingDate>
          <DocumentType>Complaint</DocumentType>
          <Security>Public</Security>
        </FiledDocument>
        
        <!-- More documents, docket entries, etc. -->
      </Case>
    </GetCaseResponse>
  </soap:Body>
</soap:Envelope>
```

[Complete GetCase schema →](../../api-reference/getcase.md)

### Performance Requirements

| Case Complexity | Target Response Time | Maximum |
|----------------|---------------------|---------|
| **Simple** (1-5 parties, 1-10 docs) | < 1 second | 2 seconds |
| **Average** (5-15 parties, 10-50 docs) | < 2 seconds | 3 seconds |
| **Complex** (15+ parties, 50+ docs) | < 3 seconds | 5 seconds |

**If GetCase times out:**
- re:Search will retry (3 attempts)
- Case update will fail
- Operations team will be alerted
- Your CMS performance needs optimization

### Common Performance Issues

**N+1 Query Problem:**
```csharp
// ❌ BAD: Query once per party (N+1 problem)
var case = GetCase(caseId);
foreach (var party in case.Parties)
{
    party.Attorney = GetAttorney(party.AttorneyId); // N queries!
}

// ✅ GOOD: Query everything at once
var case = GetCaseWithAllData(caseId); // 1 query with joins
```

**Missing Database Indexes:**
```sql
-- Add indexes on frequently queried fields
CREATE INDEX idx_case_tracking_id ON cases(case_tracking_id);
CREATE INDEX idx_parties_case_id ON parties(case_id);
CREATE INDEX idx_attorneys_case_id ON attorneys(case_id);
CREATE INDEX idx_documents_case_id ON documents(case_id);
```

**Caching Pitfall:**
```csharp
// ❌ BAD: Returning cached data
return _cache.Get(caseId); // Might be stale!

// ✅ GOOD: Always return current data
return _database.GetCurrentCase(caseId); // Fresh data
```

### Implementation Example (C#)

```csharp
public async Task<GetCaseResponse> GetCaseAsync(string caseId)
{
    var stopwatch = Stopwatch.StartNew();
    
    try
    {
        // Single query with all joins - avoid N+1 problem
        var caseData = await _db.Cases
            .Include(c => c.Parties)
            .Include(c => c.Attorneys)
            .Include(c => c.Documents)
            .Include(c => c.DocketEntries)
            .FirstOrDefaultAsync(c => c.CaseTrackingID == caseId);
            
        if (caseData == null)
        {
            throw new CaseNotFoundException(caseId);
        }
        
        // Map to ECF response format
        var response = MapToGetCaseResponse(caseData);
        
        stopwatch.Stop();
        _logger.LogInformation(
            $"GetCase completed: {caseId} in {stopwatch.ElapsedMilliseconds}ms");
            
        return response;
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, $"GetCase failed: {caseId}");
        throw;
    }
}
```

---

## RecordFiling (EFM → CMS)

### Purpose
Receive e-filing package from EFM when attorney submits filing.

### When Called
- Attorney submits filing through EFSP
- EFM validates and routes to CMS
- Part of EFM integration (separate from re:Search)

**Note:** This API is part of your EFM integration, not specific to re:Search. Documented here for completeness since it triggers the e-filing workflow.

### Process
1. EFM calls RecordFiling on your CMS
2. CMS accepts filing, stores documents
3. CMS returns receipt to EFM
4. CMS dockets filing (background process)
5. CMS sends NotifyDocketingComplete to EFM
6. EFM notifies re:Search, triggering GetCase

[EFM integration documentation →](https://efm.tylertech.com/documentation)

---

## NotifyDocketingComplete (CMS → EFM)

### Purpose
Notify EFM (and indirectly re:Search) that filing has been docketed.

### When to Send
- After clerk approves filing
- After filing assigned docket number
- After filing officially added to case record

### Process
1. CMS completes docketing
2. CMS sends NotifyDocketingComplete to EFM
3. EFM notifies re:Search
4. re:Search calls GetCase to retrieve docketed filing

**Critical:** Send this promptly after docketing - delays mean public can't see filings.

---

## GetDocument (re:Search → CMS) [Optional]

### Purpose
Retrieve document binary (PDF) when user clicks to view/download in re:Search.

### Alternative Approach
Instead of implementing GetDocument, you can provide document URLs in GetCase response:
```xml
<FiledDocument>
  <DocumentID>DOC-001</DocumentID>
  <DocumentTitle>Complaint</DocumentTitle>
  <DocumentURL>https://yourcms.court.gov/documents/DOC-001</DocumentURL>
</FiledDocument>
```

re:Search will link directly to your document viewer, and your CMS handles security enforcement.

### If Implementing GetDocument

**Request:**
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetDocumentRequest xmlns="urn:tyler:ecf:research">
      <DocumentID>DOC-001</DocumentID>
    </GetDocumentRequest>
  </soap:Body>
</soap:Envelope>
```

**Response:**
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetDocumentResponse xmlns="urn:tyler:ecf:research">
      <DocumentBinary>JVBERi0xLjQKJeLjz9MKMSAwIG9iag...</DocumentBinary>
      <MimeType>application/pdf</MimeType>
    </GetDocumentResponse>
  </soap:Body>
</soap:Envelope>
```

**Security:** Always check document security before returning - never return sealed document to unauthorized user.

---

## Infrastructure Requirements

### Network Connectivity

**Outbound (CMS → Tyler):**
- HTTPS to re:Search endpoints (provided by Tyler TPM)
- HTTPS to EFM endpoints (if e-filing)
- Port 443 outbound
- Always-on connectivity (99.9% uptime recommended)

**Inbound (Tyler → CMS):**
- Publicly accessible HTTPS endpoint for GetCase
- Optional: Endpoint for GetDocument
- Port 443 inbound
- Static IP or domain name
- Proper SSL/TLS certificate

**Firewall Requirements:**
```
Allow outbound HTTPS to:
- *.tylerhost.net (re:Search and EFM)

Allow inbound HTTPS from:
- Tyler IP ranges (provided by Tyler TPM)
```

### CMS Endpoint Requirements

**Your CMS must provide:**
- Publicly accessible HTTPS endpoint
- Valid SSL/TLS certificate (not self-signed)
- TLS 1.2 or higher
- Strong cipher suites
- Response within 3 seconds (GetCase)
- Response within 5 seconds (GetDocument)

**Example endpoints:**
```
GetCase:     https://cms.yourcourt.gov/services/GetCase
GetDocument: https://cms.yourcourt.gov/services/GetDocument
```

### Load Expectations

**Typical court (500 e-filings/month):**
- 1,000 NotifyCaseEvent/month
- 2,000 GetCase calls/month (2 per e-filing + other events)
- Peak: 10-20 events/minute

**Large court (5,000 e-filings/month):**
- 10,000 NotifyCaseEvent/month
- 20,000 GetCase calls/month
- Peak: 50-100 events/minute

**Plan infrastructure accordingly:**
- Load balancer for high availability
- Auto-scaling if possible
- Monitor response times and error rates
- Alert on degraded performance

---

## Security Requirements

### mTLS (Mutual TLS) Authentication

**What is mTLS?**
- Both client and server authenticate with certificates
- CMS authenticates to re:Search (when sending NotifyCaseEvent)
- re:Search authenticates to CMS (when calling GetCase)
- Prevents unauthorized access and man-in-the-middle attacks

**Certificate Requirements:**
- Issued by Tyler Technologies or approved CA
- Separate certificates for test and production
- Rotation every 12-18 months
- Secure storage (HSM recommended)
- Backup certificate for emergency rotation

### Certificate Request Process

1. **Contact Tyler TPM**
   - Request certificate for re:Search integration
   - Specify test or production environment

2. **Generate CSR (Certificate Signing Request)**
   ```bash
   openssl req -new -newkey rsa:2048 -nodes \
     -keyout yourcourt.key \
     -out yourcourt.csr
   ```

3. **Submit CSR to Tyler**
   - Email CSR to your TPM
   - Include: Court name, environment (test/prod)

4. **Receive Certificate**
   - Tyler issues certificate
   - Install in your environment
   - Configure CMS to use certificate

5. **Validate**
   - Test connectivity to re:Search endpoints
   - Verify certificate authentication works
   - Confirm GetCase accessible from re:Search

6. **Production Certificate**
   - Repeat process for production
   - Install 30 days before go-live
   - Test thoroughly before cutover

### Certificate Storage

**Best Practices:**
- Store in hardware security module (HSM) if available
- Use encrypted key store if HSM not available
- Restrict access (only integration service account)
- Never commit to source control
- Maintain secure backup
- Document recovery procedures

**Example (C# with certificate store):**
```csharp
// Load certificate from Windows certificate store
var store = new X509Store(StoreName.My, StoreLocation.LocalMachine);
store.Open(OpenFlags.ReadOnly);
var cert = store.Certificates.Find(
    X509FindType.FindByThumbprint, 
    "YOUR_CERT_THUMBPRINT", 
    false)[0];
store.Close();

// Configure SOAP client to use certificate
var client = new ResearchClient();
client.ClientCredentials.ClientCertificate.Certificate = cert;
```

### Endpoint Security

**For CMS endpoints (GetCase, GetDocument):**
- TLS 1.2 or higher (TLS 1.3 recommended)
- Strong cipher suites only
- Valid SSL certificate (not self-signed)
- Certificate pinning (recommended)
- Rate limiting (prevent DoS)
- IP whitelisting (Tyler IP ranges)
- Web Application Firewall (recommended)

### Security Monitoring

**Monitor:**
- Certificate expiration (alert at 60, 30, 7 days)
- Certificate validation failures
- Unauthorized access attempts
- Unusual traffic patterns
- API error rates

---

## What CMS Must Provide

### 1. Event Publishing Logic

**Implementation checklist:**
- [ ] Identify all case change scenarios
- [ ] Send NotifyCaseEvent after database commit
- [ ] Use correct event type for each scenario
- [ ] Send security events immediately (never batch)
- [ ] Implement retry logic with exponential backoff
- [ ] Use async messaging (message queue recommended)
- [ ] Log all events sent (for troubleshooting)
- [ ] Monitor event delivery success rate

**Code locations to instrument:**
- Party add/edit/delete operations
- Attorney add/edit/delete operations
- Case security updates
- Document security updates
- Disposition entry
- Judge/division reassignment
- Document additions (outside e-filing)
- Any other case data changes

### 2. GetCase Response Generation

**Implementation checklist:**
- [ ] Query all case data efficiently (avoid N+1)
- [ ] Include all required fields per ECF schema
- [ ] Return current data (not cached)
- [ ] Handle missing/sealed documents appropriately
- [ ] Optimize database queries and indexes
- [ ] Return within performance targets (< 3s)
- [ ] Handle errors gracefully
- [ ] Log response times and errors

**Performance optimization:**
- Use database query profiling tools
- Add indexes on case_id, case_tracking_id
- Use eager loading for related data
- Consider read replicas for high load
- Avoid application-level loops
- Test with complex cases (many parties/docs)

### 3. Security Enforcement

**Implementation checklist:**
- [ ] Return correct security values in GetCase
- [ ] Security values match CMS business rules
- [ ] Check security before returning documents (GetDocument)
- [ ] Send CaseSecurity events immediately when security changes
- [ ] Test sealed case visibility
- [ ] Test sealed document access
- [ ] Verify security filters work correctly

**Security values:**
- `Public` - Visible to everyone
- `Confidential` - Restricted access (court staff only)
- `Sealed` - Sealed by court order (very restricted)

**Case-sensitive:** Use exact values as shown above.

---

## Next Steps

- **Plan your implementation** → Read [Implementation Guide](./implementation-guide.md)
- **Learn about operations** → Read [Operations](./operations.md)
- **Understand the workflows** → Read [How It Works](./how-it-works.md)
- **Return to overview** → [ECF Mode Overview](./README.md)

---

**Last Updated:** December 2025