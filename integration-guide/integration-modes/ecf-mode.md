# ECF Mode (Electronic Court Filing)

> **Breadcrumb:** [Home](../../README.md) > [Integration Guide](../README.md) > [Integration Modes](./README.md) > ECF Mode

Real-time, event-driven integration for courts already connected to Tyler's Electronic Filing Manager (EFM).

---

## Table of Contents

### Overview
- [What is ECF Mode?](#what-is-ecf-mode)
- [When to Use ECF Mode](#when-to-use-ecf-mode)
- [When NOT to Use ECF Mode](#when-not-to-use-ecf-mode)

### How It Works
- [The Two Workflows](#the-two-workflows)
  - E-Filing Workflow (via EFM)
  - Direct Case Updates (CMS to re:Search)
- [Key Integration Points](#key-integration-points)

### Technical Requirements
- [APIs You'll Implement](#apis-youll-implement)
- [Infrastructure Requirements](#infrastructure-requirements)
- [Security Requirements](#security-requirements)

### Implementation
- [What CMS Must Provide](#what-cms-must-provide)
- [Development Timeline](#development-timeline)
- [Testing Requirements](#testing-requirements)

### Operations
- [Monitoring and Health](#monitoring-and-health)
- [Common Issues](#common-issues)
- [Certificate Management](#certificate-management)

### Moving Forward
- [Next Steps](#next-steps)
- [Migration Options](#migration-options)

---

## What is ECF Mode?

ECF Mode is a real-time, event-driven integration between your CMS and re:Search, designed for courts that already use Tyler's Electronic Filing Manager (EFM) for electronic court filing.

**How it works:**
```
E-Filing happens:
├─ Attorney files through EFSP
├─ EFM routes to CMS (RecordFiling)
├─ CMS dockets the filing
├─ CMS notifies re:Search (via EFM)
└─ re:Search calls GetCase to retrieve current data

Case updated in CMS:
├─ Party added, case sealed, disposition entered, etc.
├─ CMS sends NotifyCaseEvent to re:Search
├─ re:Search calls GetCase to retrieve current data
└─ re:Search indexes updated case
```

**Key characteristics:**
- Event-driven: CMS pushes notifications when cases change
- Real-time: Changes appear in re:Search within seconds
- Pull-based data: re:Search calls GetCase to retrieve actual case data
- SOAP/XML: Uses ECF 4.x web service standards
- Requires: mTLS certificates, always-on connectivity

---

## When to Use ECF Mode

ECF Mode is appropriate when:

**✅ Your court already has EFM integration**
- CMS is already integrated with Tyler's Electronic Filing Manager
- Court processes electronic filings through EFM
- Integration infrastructure already exists

**✅ You need real-time synchronization**
- Cases must appear in re:Search immediately after filing
- Security changes must take effect instantly
- Court requires up-to-the-minute data accuracy

**✅ Your CMS supports ECF standards**
- CMS has native SOAP/XML web service capabilities
- CMS vendor supports ECF 4.x (or ECF 5.x)
- Development team has SOAP integration experience

**✅ You process significant e-filing volume**
- Court handles 500+ e-filings per month
- High automation requirements
- Multiple filing events per day requiring immediate sync

---

## When NOT to Use ECF Mode

Consider alternatives if:

**❌ You don't have EFM integration**
- No existing electronic filing infrastructure
- Court not connected to statewide filing system
- **Better option:** [Batch Mode](./batch-mode.md) - simpler, file-based integration

**❌ You want simpler implementation**
- SOAP/XML complexity is a concern
- Limited development resources
- Prefer scheduled processing over real-time
- **Better option:** [Batch Mode](./batch-mode.md) - easier to implement and maintain

**❌ Your CMS is legacy/custom-built**
- CMS doesn't support web services
- No SOAP capability
- **Better option:** [CIP Mode](./cip-mode.md) - Court Integration Platform handles the web services

**❌ You need temporary solution**
- Short-term integration need
- Pilot or evaluation period
- **Better option:** [Non-Integrated Mode](./non-integrated-mode.md) - manual data entry

---

## The Two Workflows

ECF Mode operates through two distinct workflows:

### 1. E-Filing Workflow (via EFM)

When an attorney files a document electronically, the filing flows through EFM to your CMS, then synchronizes to re:Search.

**Step-by-step:**
```
1. Attorney submits filing through EFSP
   ↓
2. EFM validates and routes to CMS (RecordFiling)
   ↓
3. CMS accepts filing, returns receipt to EFM
   ↓
4. EFM notifies re:Search: "Filing received"
   ↓
5. re:Search calls CMS GetCase (pre-docket refresh)
   ↓
6. CMS dockets the filing
   ↓
7. CMS sends NotifyDocketingComplete to EFM
   ↓
8. EFM notifies re:Search: "Filing docketed"
   ↓
9. re:Search calls CMS GetCase (final refresh)
   ↓
10. Case updated in re:Search with docketed filing
```

**Why two GetCase calls?**
- First GetCase: Captures preliminary filing data
- Second GetCase: Retrieves final docketed record with complete metadata

**Your CMS responsibilities:**
- Accept RecordFiling from EFM
- Process and docket the filing
- Send NotifyDocketingComplete to EFM
- Respond to GetCase with current case data

### 2. Direct Case Updates (CMS to re:Search)

For changes that occur outside the e-filing workflow (party changes, security updates, dispositions, etc.), your CMS notifies re:Search directly.

**Step-by-step:**
```
1. Change occurs in CMS
   Examples: Party added, case sealed, disposition entered
   ↓
2. CMS sends NotifyCaseEvent to re:Search
   Includes: Case ID, Event Type (what changed)
   ↓
3. re:Search receives event notification
   ↓
4. re:Search calls CMS GetCase
   ↓
5. CMS returns complete current case data
   ↓
6. re:Search indexes updated case
   ↓
7. Changes appear in re:Search search/display
```

**Common event types you'll send:**
- **CaseParty** - Party or attorney added/changed
- **CaseSecurity** - Case sealed/unsealed
- **CaseDisposition** - Disposition entered
- **CaseReassigned** - Judge or division changed
- **DocumentFiling** - Document added outside e-filing
- **DocumentSecurity** - Document sealed/unsealed

[See all supported event types →](../../implementation-playbook/event-type-logic.md)

---

## Key Integration Points

### What Gets Sent to re:Search

**Notifications only, not data:**
- CMS sends small XML messages indicating WHAT changed
- Messages contain case ID and event type
- Actual case data comes from GetCase response

**Example NotifyCaseEvent:**
```xml
<NotifyCaseEvent>
  <CaseTrackingID>CV-2024-001234</CaseTrackingID>
  <EventType>CaseParty</EventType>
</NotifyCaseEvent>
```

This tells re:Search "case CV-2024-001234 had a party change" - then re:Search calls GetCase to get the actual current party list.

### What re:Search Retrieves from CMS

**Complete case data via GetCase:**
- All case metadata (case number, type, status, filing date, etc.)
- All parties with roles and contact information
- All attorneys with party associations and bar numbers
- All documents with metadata (no binary content)
- Current security settings
- Docket entries
- Case events/hearings

**CMS must return:**
- Current, accurate data (not cached)
- Complete data (all parties, attorneys, documents)
- Proper security settings
- Fast response (< 3 seconds for typical cases)

[See GetCase requirements →](../../api-reference/getcase.md)

---

## APIs You'll Implement

### Required APIs

**1. NotifyCaseEvent (CMS → re:Search)**
- **What:** Notification that a case changed
- **When:** Every time case data changes in CMS
- **Contains:** Case ID, event type (what changed)
- **CMS must:** Send event immediately after database commit
- **Critical:** Security events (CaseSecurity, DocumentSecurity) must never be delayed

[NotifyCaseEvent specification →](../../api-reference/notifycaseevent.md)

**2. GetCase (re:Search → CMS)**
- **What:** Request for complete current case data
- **When:** Triggered by NotifyCaseEvent or NotifyDocketingComplete
- **Returns:** Complete case record with all current data
- **CMS must:** Return current data, not cached data
- **Performance:** Must respond in < 3 seconds for typical cases

[GetCase specification →](../../api-reference/getcase.md)

**3. RecordFiling (EFM → CMS)**
- **What:** Delivery of e-filing package from EFM
- **When:** Attorney submits filing through EFSP
- **Contains:** Filing metadata and document binaries
- **CMS must:** Accept, process, and docket the filing
- **Note:** This is part of EFM integration (separate from re:Search)

**4. NotifyDocketingComplete (CMS → EFM)**
- **What:** Confirmation that filing has been docketed
- **When:** After CMS completes docketing process
- **Contains:** Filing confirmation and case status
- **CMS must:** Send after filing successfully docketed
- **Triggers:** re:Search to call GetCase for final data

### Optional API

**5. GetDocument (re:Search → CMS)**
- **What:** Request for document binary content (PDF)
- **When:** User clicks to view/download document in re:Search
- **Returns:** Base64-encoded PDF or document binary
- **Alternative:** Provide document URL in GetCase response instead

[GetDocument specification →](../../api-reference/getdocument.md)

---

## Infrastructure Requirements

### Network Connectivity

**Required:**
- Outbound HTTPS from CMS to Tyler re:Search endpoints
- Always-on connectivity (99.9% uptime recommended)
- Firewall rules allowing HTTPS to Tyler hosts
- Sufficient bandwidth for event and document transmission

**re:Search endpoints:**
- Test: `https://researchXX-stage.tylerhost.net`
- Production: `https://researchXX.tylerhost.net`

(XX = your court's identifier, provided by Tyler TPM)

### CMS Endpoint Requirements

**Your CMS must provide:**
- Publicly accessible HTTPS endpoint for GetCase
- Optional: HTTPS endpoint for GetDocument (if implementing)
- Static IP or domain name
- SSL/TLS certificate (for HTTPS)
- Response to GetCase within 3 seconds (typical cases)

---

## Security Requirements

### mTLS (Mutual TLS) Authentication

ECF Mode requires mutual TLS for all SOAP communications between CMS and re:Search.

**What is mTLS?**
- Both client and server authenticate with certificates
- Ensures only authorized systems communicate
- Prevents man-in-the-middle attacks
- Required for production operation

**Certificate requirements:**
- **Issued by:** Tyler Technologies or approved Certificate Authority
- **Rotation:** Every 12-18 months
- **Storage:** Secure key store (hardware security module recommended)
- **Backup:** Maintain backup certificate for emergency rotation
- **Monitoring:** Daily validation checks, expiration alerts

**Certificate request process:**
1. Contact your Tyler Technical Project Manager
2. Provide Certificate Signing Request (CSR)
3. Tyler issues test environment certificate
4. Validate in test environment
5. Request production certificate
6. Deploy to production

### Endpoint Security

**Required for CMS endpoints:**
- TLS 1.2 or higher
- Strong cipher suites
- Valid SSL certificate
- Firewall protection
- DDoS mitigation (recommended)
- Rate limiting (recommended)

---

## What CMS Must Provide

### 1. Event Publishing Logic

**Requirement:** Send NotifyCaseEvent whenever case data changes

**Must send events for:**
- New case filings (if outside EFM workflow)
- Party/attorney changes
- Security changes (IMMEDIATE - never delay)
- Case status changes
- Dispositions
- Judge/division reassignments
- Document additions (outside e-filing)
- Any other material case changes

**Event generation rules:**
- Send event AFTER database commit (not before)
- Send event IMMEDIATELY (especially security events)
- Include correct EventType for the change
- Include case tracking ID
- Implement retry logic for failed sends
- Log all events sent (for troubleshooting)

[Event Type Logic Guide →](../../implementation-playbook/event-type-logic.md)

### 2. GetCase Response Generation

**Requirement:** Return complete, current case data when re:Search calls GetCase

**Must include:**
- All case metadata (number, type, status, dates, etc.)
- All parties with roles and contact information
- All attorneys with party associations and bar numbers
- All documents with metadata (filing dates, security, etc.)
- All docket entries
- Current case security settings
- Current document security settings (if different from case)

**Performance requirements:**
- Response time < 1 second for simple cases
- Response time < 3 seconds for average cases
- Response time < 5 seconds for complex cases
- No cached data (must be current)

**Common mistakes to avoid:**
- Returning stale/cached data
- Missing required fields
- N+1 query performance issues (query once, not per party/attorney/document)
- Incorrect security values

[GetCase Best Practices →](../best-practices.md#optimize-getcase-response-times)

### 3. Document Retrieval (Optional)

**If implementing GetDocument:**
- Return document binary as base64-encoded string
- Check security before returning document
- Return appropriate errors for missing/sealed documents
- Response time < 5 seconds for typical documents

**Alternative to GetDocument:**
- Provide document URL in GetCase response
- re:Search links directly to your document viewer
- CMS handles security enforcement at document URL

### 4. EFM Integration (If Supporting E-Filing)

**Required for e-filing workflow:**
- Accept RecordFiling from EFM
- Process and docket filings
- Send NotifyDocketingComplete to EFM
- Handle filing rejections appropriately

**Note:** EFM integration is separate from re:Search integration but they work together in ECF Mode.

---

## Development Timeline

### Typical Implementation Schedule

**Phase 1: Development (8-12 weeks)**
- Set up development environment
- Implement SOAP client infrastructure
- Develop NotifyCaseEvent publishing logic
- Build GetCase response generation
- Implement error handling and retry logic
- Create logging and monitoring

**Phase 2: Testing (4-6 weeks)**
- Unit testing of individual components
- Integration testing with re:Search test environment
- Test all event types
- Test with real case complexity
- Security testing (sealed cases, role-based access)
- Performance testing (response times, load)
- Certification with Tyler team

**Phase 3: Deployment (2 weeks)**
- Production certificate installation
- Production environment configuration
- Smoke testing in production
- Go-live preparation
- Monitoring setup

**Total timeline: 14-20 weeks**

### Resources Required

**Development team:**
- Technical lead (50-60% time)
- SOAP/Web services developer (80-100% time)
- Integration/mapping developer (60-80% time)
- QA/test analyst (40-50% time)
- DevOps engineer (20-30% time)

**Skills needed:**
- SOAP/XML web services
- ECF 4.x standards knowledge
- Certificate and security management
- Performance optimization
- Error handling and retry logic

---

## Testing Requirements

### What You'll Test

**Event processing:**
- All event types process correctly
- Events trigger GetCase calls
- Failed events retry appropriately
- Event-to-index latency acceptable (< 30 seconds)

**Data accuracy:**
- GetCase returns complete, current data
- All required fields present
- Security settings correct
- Case type mapping correct

**Security enforcement:**
- Sealed cases not visible to public
- Sealed documents not accessible
- Role-based access works correctly
- Security changes take effect immediately

**Performance:**
- GetCase response times acceptable
- System handles expected load
- No timeouts under normal operation

**Error handling:**
- Network failures handled gracefully
- Invalid data handled appropriately
- Retry logic works correctly
- Errors logged for troubleshooting

### Certification Process

**Working with Tyler team:**
1. Complete development in test environment
2. Submit for certification review
3. Tyler team tests integration thoroughly
4. Issues identified and resolved
5. Re-test after fixes
6. Certification approval
7. Production deployment

**Typical certification: 2-3 test cycles**

[Testing & Certification Guide →](../onboarding/README.md#testing--certification)

---

## Monitoring and Health

### Key Metrics to Monitor

**Event delivery:**
- Events sent successfully vs. failed
- Target: 99.9%+ success rate
- Alert if: < 99% success rate

**GetCase performance:**
- Average response time
- Target: < 2 seconds average
- Alert if: > 5 seconds average or frequent timeouts

**Event-to-index latency:**
- Time from event sent to case updated in re:Search
- Target: < 30 seconds
- Alert if: > 2 minutes

**Certificate expiration:**
- Days until certificate expires
- Target: > 30 days
- Alert if: < 30 days remaining

**API error rate:**
- Failed API calls / total calls
- Target: < 0.5%
- Alert if: > 2%

### What to Log

**Log every:**
- NotifyCaseEvent sent (case ID, event type, timestamp, success/failure)
- GetCase call received (case ID, timestamp)
- GetCase response sent (case ID, timestamp, response time, success/failure)
- GetDocument call received (document ID, timestamp)
- GetDocument response sent (document ID, timestamp, response time, success/failure)
- All errors with details
- Certificate validation checks

**Why logging matters:**
- Troubleshooting integration issues
- Performance analysis
- Security auditing
- Compliance requirements

---

## Common Issues

### Issue: Events Not Reaching re:Search

**Symptoms:**
- Cases not updating in re:Search
- Changes made in CMS don't appear
- No errors visible in CMS logs

**Possible causes:**
- Network/firewall blocking outbound HTTPS
- Incorrect re:Search endpoint URL
- Certificate issues (expired, not installed)
- Event not being generated by CMS

**Troubleshooting:**
1. Verify network connectivity to Tyler endpoints
2. Check firewall rules allow outbound HTTPS
3. Validate certificate installed and valid
4. Review CMS logs for event generation
5. Check event queue for backed-up events
6. Contact Tyler support if issue persists

---

### Issue: GetCase Timeout

**Symptoms:**
- re:Search reports GetCase timeout
- Cases not updating despite events sent
- Errors in re:Search logs about CMS not responding

**Possible causes:**
- CMS query taking too long (N+1 problem, missing indexes)
- CMS server overloaded
- Network latency issues
- GetCase trying to return too much data

**Troubleshooting:**
1. Review CMS database query performance
2. Add database indexes on case lookups
3. Optimize query to retrieve all data in 1-3 queries (not per party/attorney/document)
4. Check CMS server load and resources
5. Test GetCase response time for complex cases
6. Consider pagination or limiting data if necessary

[Performance Optimization →](../best-practices.md#optimize-getcase-response-times)

---

### Issue: Incorrect Event Type Used

**Symptoms:**
- Changes not appearing in re:Search as expected
- re:Search logs show "noop_" messages (event ignored)
- Case updated but wrong data refreshed

**Possible causes:**
- Using wrong event type for the change
- Using generic event when specific type exists
- Misunderstanding which event type to use

**Examples:**
- Using CaseParty when attorney changes → should be CasePartyAttorney
- Using CaseFiling when case is disposed → should be CaseDisposition
- Using generic event when specific type available

**Solution:**
- Review event type logic guide
- Use most specific event type available
- Test each event type to verify behavior
- Consult documentation when unsure

[Event Type Logic →](../../implementation-playbook/event-type-logic.md)

---

### Issue: Stale Data in GetCase Response

**Symptoms:**
- re:Search shows old information
- Changes made minutes/hours ago not visible
- Event sent successfully but data not current

**Possible causes:**
- CMS returning cached data
- GetCase retrieving from read replica with lag
- Application-level caching not invalidated
- Query not using current transaction data

**Solution:**
- Disable caching for GetCase response
- Query primary database, not read replica
- Invalidate cache when data changes
- Ensure GetCase returns committed data only
- Never cache GetCase responses for more than 5-10 minutes

**Best practice:** Don't cache GetCase responses at all - re:Search calls it infrequently enough that caching isn't beneficial

---

### Issue: Security Not Enforced

**Symptoms:**
- Sealed case visible to public
- Sealed document accessible
- Security change delayed

**Possible causes:**
- CaseSecurity event not sent when case sealed
- Security value incorrect (case-sensitive: must be "Sealed" not "SEALED")
- Security event sent but delayed (batched instead of immediate)
- GetDocument not checking security before returning document

**Solution:**
- Send CaseSecurity event IMMEDIATELY when security changes
- Use exact security values: "Public", "Sealed", "Confidential"
- Never batch or delay security events
- Check security in both GetCase AND GetDocument
- Test security enforcement thoroughly

[Security Logic →](../../implementation-playbook/security-logic.md)

---

## Certificate Management

### Certificate Lifecycle

**Typical certificate lifespan: 12-18 months**

**90 days before expiration:**
1. Request new certificate from Tyler
2. Receive new certificate
3. Install in test environment
4. Validate in test
5. Schedule production installation

**30 days before expiration:**
1. Install new certificate in production
2. Validate in production
3. Monitor for issues
4. Keep old certificate as backup until validation complete

**On expiration date:**
1. Remove expired certificate
2. Verify new certificate working correctly
3. Update documentation

### Certificate Storage

**Best practices:**
- Store in hardware security module (HSM) if available
- Use encrypted key store if HSM not available
- Restrict access to certificate private keys
- Maintain backup of certificates in secure location
- Never commit certificates to source control
- Use separate certificates for test and production

### Monitoring Certificate Health

**Daily checks:**
- Certificate still valid (not expired)
- Certificate correctly installed
- Certificate accessible to application

**Alerts:**
- 60 days before expiration: Warning
- 30 days before expiration: Urgent
- 7 days before expiration: Critical

---

## Next Steps

### Ready to Implement ECF Mode?

**Step 1: Confirm Prerequisites**
- [ ] Court already has EFM integration
- [ ] CMS supports SOAP/XML web services
- [ ] Development team has web services experience
- [ ] Network connectivity to Tyler endpoints available

**Step 2: Contact Your Tyler Technical Project Manager**
- Schedule kickoff meeting
- Discuss implementation timeline
- Review resource requirements
- Establish project milestones

**Step 3: Request Certificates**
- Provide Certificate Signing Request (CSR)
- Receive test environment certificate
- Install and validate in test
- Plan for production certificate

**Step 4: Review API Documentation**
- [NotifyCaseEvent API →](../../api-reference/notifycaseevent.md)
- [GetCase API →](../../api-reference/getcase.md)
- [Event Type Logic →](../../implementation-playbook/event-type-logic.md)
- [Security Logic →](../../implementation-playbook/security-logic.md)

**Step 5: Begin Development**
- Set up development environment
- Implement NotifyCaseEvent publishing
- Build GetCase response generation
- Create error handling and retry logic
- Implement logging and monitoring

**Step 6: Schedule Testing**
- Complete internal testing
- Submit for Tyler certification
- Resolve any issues identified
- Obtain certification approval
- Deploy to production

---

## Migration Options

### Moving from ECF to Batch Mode

If you're currently using ECF Mode but want to simplify operations, you can migrate to Batch Mode.

**Why migrate to Batch Mode?**
- **Simpler:** File-based integration, no SOAP/XML complexity
- **Easier maintenance:** No certificates to manage, no real-time connectivity required
- **More predictable:** Scheduled processing instead of real-time
- **Lower cost:** Less infrastructure, less monitoring overhead

**Migration process:**
1. Coordinate with re:Search team
2. Implement Batch Mode export in parallel with ECF
3. Run both modes during transition (validation period)
4. Verify data consistency
5. Cut over to Batch Mode
6. Decommission ECF endpoints

**Timeline: 6-8 weeks with re:Search support**

**Contact your court liaison to discuss migration**

### Upgrading Within ECF Mode

**ECF 5.x available:**
- Improved error handling
- Better performance
- Enhanced security
- More flexible event types

**Contact your CMS vendor for upgrade path**

---

## Related Documentation

### Integration Modes
→ [Integration Modes Overview](./README.md) - Compare all integration methods  
→ [Batch Mode](./batch-mode.md) - Simpler file-based integration  
→ [CIP Mode](./cip-mode.md) - Court Integration Platform  
→ [Non-Integrated Mode](./non-integrated-mode.md) - Manual processing

### Technical Guides
→ [Architecture Overview](../../implementation-playbook/architecture-overview.md) - How the system works  
→ [Event Type Logic](../../implementation-playbook/event-type-logic.md) - Understanding event types  
→ [Security Logic](../../implementation-playbook/security-logic.md) - How security works  
→ [Real-World Examples](../../implementation-playbook/real-world-examples.md) - Common scenarios

### API Reference
→ [NotifyCaseEvent](../../api-reference/notifycaseevent.md) - Event notification API  
→ [GetCase](../../api-reference/getcase.md) - Case retrieval API  
→ [GetDocument](../../api-reference/getdocument.md) - Document retrieval API  
→ [API Reference Index](../../api-reference/README.md) - All APIs

### Best Practices
→ [Integration Best Practices](../best-practices.md) - Implementation guidance  
→ [Common Mistakes](../../implementation-playbook/common-mistakes.md) - What to avoid  
→ [Troubleshooting](../../troubleshooting/README.md) - Common issues

---

**Last Updated:** January 2025