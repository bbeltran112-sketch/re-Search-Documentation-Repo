# Operations and Troubleshooting

[← Back to ECF Mode Overview](./README.md)

## Quick Navigation
- [Monitoring and Metrics](#monitoring-and-metrics)
- [Common Issues](#common-issues)
- [Certificate Management](#certificate-management)
- [Troubleshooting Guide](#troubleshooting-guide)
- [Getting Help](#getting-help)

---

## Monitoring and Metrics

### Key Performance Indicators

Monitor these metrics to ensure healthy ECF Mode operations:

| Metric | Target | Alert Threshold | Action if Exceeded |
|--------|--------|-----------------|-------------------|
| **Event delivery success** | 99.9%+ | < 99% | Check network/connectivity, review error logs |
| **GetCase response time** | < 2s avg | > 5s avg or frequent timeouts | Optimize queries, check database load |
| **Event-to-index latency** | < 30s | > 2 minutes | Check both CMS and re:Search side |
| **Certificate days remaining** | > 30 days | < 30 days | Request renewal immediately |
| **API error rate** | < 0.5% | > 2% | Investigate errors, check connectivity |
| **GetCase timeout rate** | < 0.1% | > 1% | Performance optimization needed |

### Daily Monitoring Checklist

**Morning:**
- [ ] Check overnight event delivery (success rate)
- [ ] Review error logs for failed events or GetCase calls
- [ ] Verify certificate still valid
- [ ] Check GetCase response time trends
- [ ] Review re:Search processing status

**Throughout Day:**
- [ ] Monitor real-time event processing
- [ ] Alert on any failed event deliveries
- [ ] Check for performance degradation
- [ ] Address any timeouts immediately

**Weekly:**
- [ ] Analyze event delivery trends
- [ ] Review GetCase performance patterns
- [ ] Check certificate expiration date
- [ ] Review error patterns for systemic issues
- [ ] Coordinate with Tyler TPM if concerns

**Monthly:**
- [ ] Generate integration health reports
- [ ] Review optimization opportunities
- [ ] Check for ECF standard updates
- [ ] Update documentation based on lessons learned
- [ ] Certificate expiration check (renew if < 90 days)

### What to Log

**Log every:**
- NotifyCaseEvent sent (case ID, event type, timestamp, success/failure)
- GetCase request received (case ID, timestamp, caller IP)
- GetCase response sent (case ID, timestamp, response time, size, success/failure)
- GetDocument request received (document ID, timestamp)
- GetDocument response sent (document ID, timestamp, response time, success/failure)
- All errors with full context
- Certificate validation checks
- Security events (especially important)

**Log format example:**
```json
{
  "timestamp": "2025-01-15T14:23:45Z",
  "event_type": "NotifyCaseEvent",
  "case_id": "CV-2024-001234",
  "event_category": "CaseParty",
  "success": true,
  "response_time_ms": 245,
  "retry_count": 0
}
```

### Monitoring Dashboard

**Key widgets:**
- Event delivery success rate (last 24h, 7d, 30d)
- GetCase response time distribution
- Event-to-index latency trend
- Error rate by type
- Certificate expiration countdown
- API call volume over time

**Alerts to configure:**
- Event delivery success < 99%
- GetCase average response > 3 seconds
- GetCase timeout rate > 1%
- Certificate expires in < 30 days
- API error rate > 2%
- No events sent in last hour (during business hours)

---

## Common Issues

### Issue: Events Not Reaching re:Search

**Symptoms:**
- Cases not updating in re:Search
- Changes made in CMS don't appear
- No errors visible in CMS logs
- re:Search shows stale data

**Possible Causes:**
- Network/firewall blocking outbound HTTPS to Tyler endpoints
- Incorrect re:Search endpoint URL configured
- Certificate expired or not properly installed
- Event publishing code not executing
- Events queued but not being sent

**Troubleshooting Steps:**

1. **Verify network connectivity:**
   ```bash
   # Test connectivity to re:Search endpoint
   curl -v https://researchXX.tylerhost.net/NotifyCaseEvent
   # Should connect (may return error, but connection succeeds)
   ```

2. **Check firewall rules:**
   ```bash
   # Verify outbound HTTPS allowed to *.tylerhost.net
   # Check with network team if blocked
   ```

3. **Validate certificate:**
   ```bash
   # Check certificate expiration
   openssl s_client -connect researchXX.tylerhost.net:443 -showcerts
   # Look for "Verify return code: 0 (ok)"
   ```

4. **Review CMS logs:**
   - Search for "NotifyCaseEvent" in logs
   - Check if events being generated
   - Look for error messages

5. **Check event queue:**
   - If using message queue, check queue depth
   - Look for backed-up messages
   - Verify queue processor running

**Resolution:**
- Fix network/firewall issues with IT team
- Update endpoint URL if incorrect
- Renew certificate if expired
- Fix event publishing code if broken
- Clear backed-up queue and monitor

---

### Issue: GetCase Timeout

**Symptoms:**
- re:Search logs show "GetCase timeout"
- Cases not updating despite events sent successfully
- Multiple retry attempts visible in logs
- CMS shows slow query performance

**Possible Causes:**
- Database query taking too long (N+1 problem)
- Missing database indexes
- CMS server overloaded/under-resourced
- Network latency between re:Search and CMS
- GetCase trying to return too much data

**Troubleshooting Steps:**

1. **Check GetCase response times in CMS logs:**
   ```
   Look for GetCase calls taking > 3 seconds
   Identify which cases are slow (complex cases?)
   ```

2. **Profile database queries:**
   ```sql
   -- Enable query logging
   -- Look for queries taking > 1 second
   -- Check for N+1 patterns (many small queries instead of one)
   ```

3. **Review database indexes:**
   ```sql
   -- Verify indexes exist on:
   SELECT * FROM sys.indexes WHERE object_id = OBJECT_ID('Cases');
   -- Look for indexes on: case_tracking_id, case_id
   ```

4. **Check server resources:**
   - CPU usage during GetCase calls
   - Memory utilization
   - Database connection pool exhaustion
   - Disk I/O bottlenecks

5. **Test GetCase directly:**
   ```bash
   # Send GetCase request directly to CMS
   # Measure response time
   time curl -X POST https://yourcms.court.gov/GetCase \
     -H "Content-Type: text/xml" \
     -d @getcase_request.xml
   ```

**Resolution:**

**Optimize queries:**
```csharp
// ❌ BAD: N+1 problem
var caseData = GetCase(caseId);
foreach (var party in caseData.Parties) {
    party.Attorney = GetAttorney(party.AttorneyId); // Separate query!
}

// ✅ GOOD: Single query with joins
var caseData = GetCaseWithAllData(caseId); // One query
```

**Add indexes:**
```sql
CREATE INDEX idx_case_tracking_id ON cases(case_tracking_id);
CREATE INDEX idx_parties_case_id ON parties(case_id);
CREATE INDEX idx_attorneys_case_id ON attorneys(case_id);
CREATE INDEX idx_documents_case_id ON documents(case_id);
```

**Increase server resources** if queries optimized but still slow.

**Consider read replica** for GetCase queries if very high load.

---

### Issue: Incorrect Event Type Used

**Symptoms:**
- Events sent successfully but data not refreshing
- re:Search logs show "noop_" messages (event ignored)
- Specific data not updating (e.g., attorney added but not showing)
- GetCase called but wrong data retrieved

**Possible Causes:**
- Using wrong event type for the change
- Using generic event when specific type exists
- Misunderstanding which event type to use
- Event type not matching actual change

**Common Mistakes:**

| Wrong Event Type | Correct Event Type | When to Use |
|------------------|-------------------|-------------|
| `CaseParty` | `CasePartyAttorney` | When attorney changes |
| `CaseFiling` | `CaseDisposition` | When disposition entered |
| `CaseInitiated` | `CaseParty` | When party added to existing case |
| Generic event | Most specific event | Always use most specific available |

**Troubleshooting Steps:**

1. **Review event type logic:**
   - Consult [Event Type Logic Guide](../../implementation-playbook/event-type-logic.md)
   - Identify what actually changed in CMS
   - Match change to correct event type

2. **Check re:Search logs:**
   - Look for event received confirmation
   - Check if event triggered GetCase
   - Look for "noop" or "ignored" messages

3. **Test event type:**
   - Make change in test CMS
   - Send specific event type
   - Verify correct data updates in re:Search

**Resolution:**
- Update CMS code to use correct event type
- Document event type decision logic
- Add code comments explaining why each event type chosen
- Test each change scenario thoroughly

[Event Type Logic Reference →](../../implementation-playbook/event-type-logic.md)

---

### Issue: Stale Data in GetCase Response

**Symptoms:**
- Changes made minutes/hours ago not visible in re:Search
- Event sent successfully but data appears old
- User reports seeing outdated information
- Resubmitting event makes data current

**Possible Causes:**
- CMS returning cached data instead of current
- GetCase querying read replica with replication lag
- Application-level caching not invalidated
- Transaction isolation causing stale reads
- Time lag between database commit and query

**Troubleshooting Steps:**

1. **Check for caching:**
   ```csharp
   // Look for code like this in GetCase implementation
   if (_cache.ContainsKey(caseId)) {
       return _cache.Get(caseId); // ❌ This is the problem!
   }
   ```

2. **Verify database source:**
   ```sql
   -- Are you querying primary or replica?
   -- If replica, check replication lag
   SELECT * FROM sys.dm_hadr_database_replica_states;
   ```

3. **Test GetCase directly:**
   - Make change in CMS
   - Immediately call GetCase
   - Check if change reflected
   - If not, caching is likely issue

4. **Check transaction timing:**
   - Verify event sent AFTER database commit
   - Ensure GetCase queries committed data

**Resolution:**

**Disable caching for GetCase:**
```csharp
// ❌ Don't do this
var caseData = _cache.GetOrAdd(caseId, () => QueryDatabase(caseId));

// ✅ Do this
var caseData = QueryDatabase(caseId); // Always fresh
```

**Query primary database:**
```csharp
// If using read replicas, force primary for GetCase
using (var context = new CmsDbContext(forcePrimary: true)) {
    var caseData = context.Cases
        .Include(c => c.Parties)
        .Include(c => c.Attorneys)
        .FirstOrDefault(c => c.TrackingId == caseId);
}
```

**Invalidate cache on changes:**
```csharp
// If you must cache, invalidate when data changes
public void UpdateParty(Party party) {
    _db.Update(party);
    _db.SaveChanges();
    _cache.Remove(party.CaseId); // Clear cache
    SendNotifyCaseEvent(party.CaseId, "CaseParty");
}
```

**Best Practice:** Don't cache GetCase responses. re:Search calls it infrequently enough that caching provides minimal benefit and adds staleness risk.

---

### Issue: Security Not Enforced

**Symptoms:**
- Sealed case visible to public in re:Search
- Sealed document accessible when shouldn't be
- Security change delayed appearing
- User reports seeing sealed content

**Possible Causes:**
- CaseSecurity event not sent when case sealed
- Security value incorrect (case-sensitive: "Sealed" not "SEALED")
- Security event batched/delayed instead of immediate
- GetDocument not checking security before returning
- Security value mapping incorrect

**Troubleshooting Steps:**

1. **Verify event sent:**
   - Check CMS logs for CaseSecurity event
   - Confirm event sent immediately when sealed
   - Look for any delays or queuing

2. **Check security value:**
   ```xml
   <!-- ❌ Wrong - case matters! -->
   <Security>SEALED</Security>
   <Security>sealed</Security>
   
   <!-- ✅ Correct -->
   <Security>Sealed</Security>
   ```

3. **Test security flow:**
   - Seal case in CMS
   - Verify CaseSecurity event sent immediately
   - Check re:Search shows case as sealed
   - Verify public user can't see case
   - Test document access if applicable

4. **Review GetDocument security:**
   ```csharp
   public GetDocumentResponse GetDocument(string docId) {
       var doc = _db.Documents.Find(docId);
       // ❌ Missing security check!
       return new GetDocumentResponse { Binary = doc.Content };
   }
   ```

**Resolution:**

**Send security events immediately:**
```csharp
public void SealCase(string caseId) {
    _db.Cases.Find(caseId).Security = "Sealed";
    _db.SaveChanges();
    
    // ❌ Don't queue or batch security events
    // _eventQueue.Enqueue(caseId, "CaseSecurity");
    
    // ✅ Send immediately, synchronously
    SendNotifyCaseEvent(caseId, "CaseSecurity").Wait();
}
```

**Use exact security values:**
```csharp
// Define constants to avoid typos
public static class SecurityLevel {
    public const string Public = "Public";
    public const string Confidential = "Confidential";
    public const string Sealed = "Sealed";
}

// Use in code
case.Security = SecurityLevel.Sealed;
```

**Check security in GetDocument:**
```csharp
public GetDocumentResponse GetDocument(string docId, User user) {
    var doc = _db.Documents.Find(docId);
    
    // ✅ Check security before returning
    if (doc.Security == "Sealed" && !user.CanAccessSealed) {
        throw new UnauthorizedAccessException();
    }
    
    return new GetDocumentResponse { Binary = doc.Content };
}
```

[Security Logic Reference →](../../implementation-playbook/security-logic.md)

---

## Certificate Management

### Certificate Lifecycle

**Typical certificate lifespan: 12-18 months**

### Timeline

**90 days before expiration:**
1. Receive automated expiration warning
2. Contact Tyler TPM to request renewal
3. Generate new CSR (if required)
4. Submit renewal request

**60 days before expiration:**
1. Receive new certificate from Tyler
2. Install in test environment
3. Validate certificate works correctly
4. Schedule production installation

**30 days before expiration:**
1. Install new certificate in production
2. Validate connectivity and authentication
3. Monitor for any issues
4. Keep old certificate as backup temporarily

**On expiration date:**
1. Remove expired certificate
2. Verify new certificate working correctly
3. Update documentation with new expiration date
4. Set reminder for next renewal

### Certificate Storage

**Best Practices:**
- Store in hardware security module (HSM) if available
- Use encrypted key store if HSM not available
- Restrict access (only integration service account)
- Never commit certificates to source control
- Maintain secure backup of certificate
- Document recovery procedures

**Windows Certificate Store:**
```powershell
# Install certificate
Import-PfxCertificate -FilePath cert.pfx `
  -CertStoreLocation Cert:\LocalMachine\My `
  -Password (ConvertTo-SecureString -String "password" -AsPlainText -Force)

# Grant access to service account
$cert = Get-ChildItem Cert:\LocalMachine\My\{thumbprint}
$rsaCert = [System.Security.Cryptography.X509Certificates.RSACertificateExtensions]::GetRSAPrivateKey($cert)
$fileName = $rsaCert.Key.UniqueName
$path = "$env:ALLUSERSPROFILE\Microsoft\Crypto\RSA\MachineKeys\$fileName"
$acl = Get-Acl -Path $path
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("DOMAIN\ServiceAccount","Read","Allow")
$acl.AddAccessRule($accessRule)
Set-Acl -Path $path -AclObject $acl
```

**Linux/Java KeyStore:**
```bash
# Import certificate into Java keystore
keytool -importcert -file certificate.crt \
  -keystore /path/to/keystore.jks \
  -alias research_cert \
  -storepass changeit

# Verify certificate
keytool -list -v -keystore /path/to/keystore.jks \
  -alias research_cert \
  -storepass changeit
```

### Monitoring Certificate Health

**Automated checks:**
```csharp
public class CertificateMonitor {
    public void CheckCertificate() {
        var store = new X509Store(StoreName.My, StoreLocation.LocalMachine);
        store.Open(OpenFlags.ReadOnly);
        var cert = store.Certificates.Find(
            X509FindType.FindByThumbprint, 
            _certThumbprint, 
            false)[0];
        
        var daysUntilExpiration = (cert.NotAfter - DateTime.Now).Days;
        
        if (daysUntilExpiration < 30) {
            _logger.LogCritical($"Certificate expires in {daysUntilExpiration} days!");
            // Send alert
        } else if (daysUntilExpiration < 60) {
            _logger.LogWarning($"Certificate expires in {daysUntilExpiration} days");
        }
        
        // Test certificate validation
        var chain = new X509Chain();
        bool isValid = chain.Build(cert);
        if (!isValid) {
            _logger.LogError("Certificate validation failed!");
            // Send alert
        }
    }
}
```

**Alert schedule:**
- Daily check: Certificate still valid
- 90 days: Warning (start renewal process)
- 60 days: Important (certificate should be ready)
- 30 days: Urgent (install new certificate now!)
- 7 days: Critical (emergency!)

### Emergency Certificate Rotation

**If certificate compromised or emergency renewal needed:**

1. **Contact Tyler TPM immediately**
   - Explain situation
   - Request emergency certificate

2. **Receive emergency certificate**
   - Usually within 24-48 hours
   - Expedited process for emergencies

3. **Install and test**
   - Install in test environment first
   - Validate works correctly
   - Deploy to production quickly

4. **Monitor closely**
   - Watch for authentication failures
   - Verify events processing
   - Check GetCase accessibility

---

## Troubleshooting Guide

### Debug Checklist

When troubleshooting issues, follow this systematic approach:

1. **Check event logs**
   - Review CMS logs for NotifyCaseEvent attempts
   - Check success/failure rates
   - Look for error messages

2. **Verify connectivity**
   - Test network connection to Tyler endpoints
   - Check firewall rules
   - Verify DNS resolution

3. **Validate certificate**
   - Check expiration date
   - Verify installed correctly
   - Test certificate authentication

4. **Review GetCase performance**
   - Check response times in logs
   - Look for timeout patterns
   - Identify slow cases

5. **Test event flow**
   - Make test change in CMS
   - Verify event sent
   - Check GetCase called
   - Confirm data updated

6. **Check re:Search side**
   - Contact Tyler TPM for re:Search logs
   - Verify events received
   - Check GetCase calls made
   - Review indexing status

7. **Compare timestamps**
   - When did change occur in CMS?
   - When was event sent?
   - When was GetCase called?
   - When did data appear in re:Search?
   - Identify delays

### Performance Analysis

**If GetCase is slow:**

1. **Enable query profiling:**
   ```sql
   SET STATISTICS TIME ON;
   SET STATISTICS IO ON;
   -- Run GetCase query
   -- Review execution plan
   ```

2. **Check for N+1:**
   - Count number of queries for one GetCase
   - Should be 1-3 queries total
   - If 10+ queries, N+1 problem exists

3. **Review indexes:**
   ```sql
   -- Check index usage
   SELECT * FROM sys.dm_db_index_usage_stats
   WHERE database_id = DB_ID()
   AND object_id = OBJECT_ID('Cases');
   ```

4. **Monitor server resources:**
   - CPU usage during GetCase
   - Memory utilization
   - Disk I/O
   - Database connection pool

---

## Getting Help

### Support Resources

**For Integration Issues:**
1. Review this troubleshooting guide
2. Check logs for error details
3. Test connectivity and certificates
4. Contact Tyler Technical Project Manager

**For Certificate Issues:**
- Email Tyler TPM for renewal/replacement
- Include: Court name, environment, expiration date
- Request expedited if urgent

**For Performance Issues:**
- Review [Performance Optimization Guide](../best-practices.md#optimize-getcase-response-times)
- Profile database queries
- Check server resources
- Contact Tyler TPM if re:Search-side issue suspected

### Support Ticket Information

When contacting support, provide:

**For Event Delivery Issues:**
- Case IDs that didn't update
- Event type sent
- Timestamp of change
- CMS logs showing event attempts
- Network/connectivity test results

**For GetCase Issues:**
- Case IDs with slow responses
- Response time measurements
- Database query execution plans
- Server resource utilization
- CMS logs showing GetCase calls

**For Certificate Issues:**
- Current certificate expiration date
- Error messages from certificate validation
- Environment (test/production)
- When issue started

### Contact Information

- **Tyler Technical Project Manager:** Assigned during onboarding
- **Email:** Provided by Tyler TPM
- **Phone:** For critical production issues
- **Support Portal:** Tyler support system (if available)

### Escalation Procedures

**Normal Priority (Response within 1 business day):**
- Non-critical performance issues
- Questions about event types
- Documentation clarifications

**High Priority (Response within 4 hours):**
- Production integration issues affecting operations
- Certificate expiration < 7 days
- High error rates impacting service

**Critical (Immediate response):**
- Production outage (no events processing)
- Security incident (sealed cases exposed)
- Certificate expired

---

## Best Practices

### Proactive Monitoring

- Set up automated alerts for key metrics
- Review logs daily for patterns
- Monitor certificate expiration continuously
- Track performance trends over time
- Schedule regular health checks

### Operational Excellence

- Document all procedures and runbooks
- Keep contact lists updated
- Train multiple team members
- Practice certificate rotation in test
- Maintain rollback procedures

### Performance Optimization

- Continuously monitor GetCase response times
- Optimize database queries proactively
- Add indexes based on query patterns
- Consider read replicas for high load
- Cache reference data (not case data)

### Security

- Never delay security events
- Test security enforcement regularly
- Monitor security event latency
- Validate certificate health daily
- Restrict certificate access strictly

---

## Next Steps

- **Plan your implementation** → Read [Implementation Guide](./implementation-guide.md)
- **Review technical details** → Read [Technical Setup](./technical-setup.md)
- **Understand the workflows** → Read [How It Works](./how-it-works.md)
- **Return to overview** → [ECF Mode Overview](./README.md)

---

**Last Updated:** December 2025