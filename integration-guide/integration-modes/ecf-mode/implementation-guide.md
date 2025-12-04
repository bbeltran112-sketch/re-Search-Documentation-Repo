# Implementation Guide

[← Back to ECF Mode Overview](./README.md)

## Quick Navigation
- [Development Timeline](#development-timeline)
- [Resources Required](#resources-required)
- [Development Checklist](#development-checklist)
- [Testing Requirements](#testing-requirements)
- [Certification Process](#certification-process)
- [Deployment](#deployment)

---

## Development Timeline

### Typical Implementation: 14-20 Weeks

```
Phase 1: Planning & Setup (2-3 weeks)
├─ Week 1: Kickoff and requirements review
├─ Week 2: Environment setup and certificate request
└─ Week 3: Development environment configuration
    Deliverable: Dev environment ready, certificates installed

Phase 2: Development (8-12 weeks)
├─ Week 1-2: SOAP client infrastructure
├─ Week 3-4: NotifyCaseEvent publishing logic
├─ Week 5-7: GetCase response generation
├─ Week 8-9: Event type implementation (all scenarios)
├─ Week 10-11: Error handling and retry logic
└─ Week 12: Logging, monitoring, and documentation
    Deliverable: All APIs implemented and unit tested

Phase 3: Testing (4-6 weeks)
├─ Week 1-2: Integration testing with re:Search test environment
├─ Week 3: Security testing (sealed cases, role-based access)
├─ Week 4: Performance testing (response times, load)
├─ Week 5: End-to-end testing (all event types)
└─ Week 6: Certification with Tyler team
    Deliverable: Certification approved

Phase 4: Deployment (2 weeks)
├─ Week 1: Production certificate installation and configuration
├─ Week 2: Production deployment and smoke testing
└─ Deliverable: Production integration live and monitored
```

### Timeline Factors

**Faster Implementation (12-14 weeks):**
- CMS already has SOAP infrastructure
- Team experienced with ECF standards
- Existing EFM integration to leverage
- Simple case data model

**Slower Implementation (20-24 weeks):**
- Building SOAP infrastructure from scratch
- Team learning ECF standards
- Complex case data model (many custom fields)
- Multiple court jurisdictions in scope
- Legacy CMS requiring significant refactoring

---

## Resources Required

### Development Team

| Role | Time Commitment | Responsibilities |
|------|----------------|------------------|
| **Technical Lead** | 50-60% | Architecture, design, code review, coordination |
| **SOAP Developer** | 80-100% | SOAP client/server implementation, APIs |
| **Integration Developer** | 60-80% | Event publishing, data mapping, GetCase logic |
| **Database Developer** | 40-50% | Query optimization, indexes, schema updates |
| **QA/Test Analyst** | 40-50% | Test planning, execution, certification support |
| **DevOps Engineer** | 20-30% | Environment setup, deployment, monitoring |

### Required Skills

**Essential:**
- SOAP/XML web services
- C# or Java (typical CMS languages)
- SQL and database optimization
- Certificate and security management
- Error handling and retry patterns

**Helpful:**
- ECF 4.x or 5.x standards knowledge
- Previous EFM integration experience
- Message queue systems
- Load testing and performance tuning

### External Dependencies

**Tyler Technologies:**
- Technical Project Manager (TPM) assignment
- Test environment credentials
- Certificate issuance
- Certification testing support
- Production deployment coordination

**Your Organization:**
- Network/firewall team (endpoint configuration)
- Security team (certificate management)
- Operations team (monitoring setup)
- Management approval for go-live

---

## Development Checklist

### Phase 1: Planning & Setup

**Project Kickoff:**
- [ ] Schedule kickoff meeting with Tyler TPM
- [ ] Review project scope and timeline
- [ ] Identify key stakeholders and roles
- [ ] Establish communication channels
- [ ] Set project milestones

**Environment Setup:**
- [ ] Request test environment access from Tyler
- [ ] Generate CSR and request test certificate
- [ ] Install certificate in development environment
- [ ] Configure firewall rules (outbound HTTPS to Tyler)
- [ ] Set up development/test CMS environment
- [ ] Verify connectivity to Tyler test endpoints

**Documentation Review:**
- [ ] Review ECF 4.x/5.x standards documentation
- [ ] Study NotifyCaseEvent API specification
- [ ] Study GetCase API specification
- [ ] Review event type logic guide
- [ ] Understand security requirements
- [ ] Document CMS data model mapping

### Phase 2: Development

**SOAP Infrastructure:**
- [ ] Set up SOAP client library (e.g., WCF, JAX-WS)
- [ ] Configure client with mTLS certificate
- [ ] Test connectivity to re:Search test endpoint
- [ ] Implement SOAP server for GetCase endpoint
- [ ] Configure server with SSL certificate
- [ ] Test GetCase endpoint accessibility

**NotifyCaseEvent Implementation:**
- [ ] Identify all case change scenarios requiring events
- [ ] Implement event publishing framework
- [ ] Add event hooks to CMS change operations:
  - [ ] Party add/edit/delete
  - [ ] Attorney add/edit/delete
  - [ ] Case security changes (CRITICAL)
  - [ ] Document security changes (CRITICAL)
  - [ ] Disposition entry
  - [ ] Judge/division reassignment
  - [ ] Document additions (non-e-filing)
  - [ ] Case status changes
- [ ] Implement async event sending (message queue recommended)
- [ ] Implement retry logic with exponential backoff
- [ ] Add comprehensive event logging
- [ ] Test event delivery to re:Search test environment

**GetCase Implementation:**
- [ ] Design database query strategy (avoid N+1)
- [ ] Implement efficient data retrieval:
  - [ ] Case metadata
  - [ ] All parties with roles
  - [ ] All attorneys with associations
  - [ ] All documents with metadata
  - [ ] Docket entries
  - [ ] Case events/hearings
- [ ] Map CMS data to ECF schema format
- [ ] Implement security value mapping
- [ ] Add comprehensive error handling
- [ ] Add response time logging
- [ ] Optimize database queries and indexes
- [ ] Test with simple, average, and complex cases

**Event Type Logic:**
- [ ] Implement correct event type for each scenario
- [ ] Test each event type triggers appropriate GetCase
- [ ] Verify data refreshed correctly for each event type
- [ ] Document event type decision logic

**Error Handling:**
- [ ] Implement retry logic for failed NotifyCaseEvent
- [ ] Handle GetCase timeouts gracefully
- [ ] Return proper HTTP error codes
- [ ] Log all errors with context
- [ ] Implement alerting for error rate thresholds

**Monitoring & Logging:**
- [ ] Log all NotifyCaseEvent sent (success/failure)
- [ ] Log all GetCase requests received
- [ ] Log all GetCase responses sent (with timing)
- [ ] Log certificate validation status
- [ ] Set up monitoring dashboards
- [ ] Configure alerts for key metrics

**Documentation:**
- [ ] Create operational runbook
- [ ] Document troubleshooting procedures
- [ ] List key contacts and escalation paths
- [ ] Provide example event scenarios
- [ ] Document GetCase query optimization

### Phase 3: Testing

**Unit Testing:**
- [ ] Test NotifyCaseEvent with all event types
- [ ] Test GetCase with various case complexities
- [ ] Test error handling and retry logic
- [ ] Test certificate authentication
- [ ] Test security value mapping

**Integration Testing:**
- [ ] Send test events to re:Search test environment
- [ ] Verify GetCase called by re:Search
- [ ] Verify case data indexed correctly in re:Search
- [ ] Test all event types end-to-end
- [ ] Test with real case data (anonymized if needed)

**Security Testing:**
- [ ] Test sealed case not visible to public
- [ ] Test sealed document not accessible
- [ ] Test security changes take effect immediately
- [ ] Test role-based access controls
- [ ] Verify CaseSecurity events sent immediately

**Performance Testing:**
- [ ] Test GetCase response times:
  - [ ] Simple cases (< 1 second)
  - [ ] Average cases (< 2 seconds)
  - [ ] Complex cases (< 3 seconds)
- [ ] Load testing (simulate peak event volume)
- [ ] Stress testing (find breaking point)
- [ ] Identify and fix performance bottlenecks

**End-to-End Testing:**
- [ ] Test complete e-filing workflow (if applicable)
- [ ] Test direct case update workflow
- [ ] Test with multiple simultaneous events
- [ ] Test recovery after network failure
- [ ] Test certificate rotation scenario

### Phase 4: Deployment

**Production Preparation:**
- [ ] Request production certificate from Tyler
- [ ] Install production certificate
- [ ] Configure production endpoints
- [ ] Update firewall rules for production
- [ ] Set up production monitoring and alerts
- [ ] Document production support procedures

**Deployment:**
- [ ] Deploy to production environment
- [ ] Smoke test all APIs in production
- [ ] Verify connectivity to production re:Search
- [ ] Send test event and verify GetCase called
- [ ] Monitor for 24-48 hours post-deployment
- [ ] Address any issues immediately

---

## Testing Requirements

### What You'll Test

**Functional Testing:**
- Event delivery for all event types
- GetCase returns complete, correct data
- Security enforcement (sealed cases/documents)
- Error handling and retry logic
- Certificate authentication

**Performance Testing:**
- GetCase response times meet targets
- System handles expected load
- No timeouts under normal operation
- Recovery from high load periods

**Integration Testing:**
- Events trigger GetCase calls
- Data flows correctly end-to-end
- re:Search indexes data accurately
- Search results reflect CMS changes

**Security Testing:**
- mTLS authentication works correctly
- Sealed cases not visible to public
- Sealed documents not accessible
- Security changes immediate
- Role-based access enforced

### Test Scenarios

**Event Processing:**
1. Add party to case → Verify CaseParty event sent → Verify GetCase called → Verify party visible in re:Search
2. Seal case → Verify CaseSecurity event sent immediately → Verify GetCase called → Verify case not visible to public
3. Enter disposition → Verify CaseDisposition event sent → Verify GetCase called → Verify disposition visible
4. Add attorney → Verify CasePartyAttorney event sent → Verify GetCase called → Verify attorney visible
5. Change judge → Verify CaseReassigned event sent → Verify GetCase called → Verify judge updated

**E-Filing Workflow (if applicable):**
1. Submit e-filing → RecordFiling received → NotifyDocketingComplete sent → Verify GetCase called twice → Verify filing visible in re:Search

**Performance:**
1. Simple case (5 parties, 10 docs) → GetCase < 1 second
2. Average case (10 parties, 30 docs) → GetCase < 2 seconds
3. Complex case (20+ parties, 100+ docs) → GetCase < 3 seconds
4. Load test: 100 events in 10 minutes → All process successfully

**Error Handling:**
1. Network failure during event send → Retry succeeds → Event processed
2. GetCase timeout → re:Search retries → Eventually succeeds
3. Invalid case ID in event → Error logged → Operations alerted

---

## Certification Process

### Overview

Tyler requires certification testing before production deployment. This ensures integration quality and prevents production issues.

### Certification Steps

**1. Complete Development**
- All APIs implemented
- Internal testing complete
- Documentation prepared
- Ready for external validation

**2. Submit for Certification**
- Contact Tyler TPM
- Provide test environment access
- Share test data (if needed)
- Schedule certification window

**3. Tyler Team Testing**
- Tyler tests all event types
- Tyler validates GetCase responses
- Tyler checks security enforcement
- Tyler tests performance
- Tyler identifies any issues

**4. Issue Resolution**
- Review Tyler feedback
- Fix identified issues
- Retest internally
- Request re-certification

**5. Re-Testing**
- Tyler re-tests after fixes
- Verify all issues resolved
- Check for regressions

**6. Certification Approval**
- Tyler approves integration
- Provides production certificate
- Coordinates go-live

### Typical Certification Timeline

**First Attempt:**
- Testing: 1-2 weeks
- Issues found: Common (don't worry!)
- Fix time: 1-2 weeks

**Second Attempt:**
- Testing: 1 week
- Issues found: Fewer
- Fix time: < 1 week

**Third Attempt (if needed):**
- Testing: < 1 week
- Issues found: Rare
- Approval likely

**Total certification time: 3-5 weeks typically**

### Common Certification Issues

**Event Problems:**
- Wrong event type used for scenario
- Security events delayed (must be immediate)
- Events not sent for all change scenarios
- Event format doesn't match specification

**GetCase Problems:**
- Missing required fields
- Response times too slow (> 5 seconds)
- Returning cached/stale data
- Incorrect security values
- N+1 query performance issues

**Security Problems:**
- Sealed cases visible when shouldn't be
- Security values case-sensitive (must be exact)
- Document security not enforced
- Security events not immediate

**Certificate Problems:**
- Certificate not installed correctly
- Certificate expired
- TLS configuration issues
- Handshake failures

---

## Deployment

### Pre-Deployment Checklist

**Infrastructure:**
- [ ] Production certificate installed and validated
- [ ] Production endpoints configured
- [ ] Firewall rules updated for production
- [ ] Load balancer configured (if applicable)
- [ ] Monitoring and alerts configured
- [ ] Log aggregation configured

**Testing:**
- [ ] Certification approved by Tyler
- [ ] Smoke tests passed in production
- [ ] Rollback plan documented and tested
- [ ] Support team trained

**Communication:**
- [ ] Stakeholders notified of go-live date
- [ ] Court staff informed of changes
- [ ] Support contacts confirmed
- [ ] On-call schedule established

### Deployment Day

**Morning:**
1. Final environment check (certificates, connectivity)
2. Deploy code to production
3. Smoke test NotifyCaseEvent → Success
4. Smoke test GetCase → Success
5. Send test event, verify GetCase called
6. Monitor for first hour closely

**First 24 Hours:**
- Monitor all key metrics continuously
- Review logs for errors hourly
- Check GetCase response times
- Verify events processing correctly
- Address any issues immediately

**First Week:**
- Daily review of metrics and logs
- Weekly meeting with Tyler TPM
- Document any issues and resolutions
- Tune performance if needed

### Post-Deployment

**Week 2-4:**
- Continue monitoring (can reduce frequency)
- Analyze trends and patterns
- Optimize based on production data
- Update documentation with lessons learned

**Ongoing:**
- Monitor key metrics daily
- Review certificate expiration monthly
- Update documentation as needed
- Coordinate with Tyler on improvements

---

## Common Implementation Challenges

### Challenge: GetCase Performance

**Problem:** Response times > 5 seconds, causing timeouts

**Solutions:**
- Profile database queries, identify N+1 problems
- Add indexes on case_id, case_tracking_id
- Use eager loading for related entities
- Consider database read replicas
- Cache reference data (court codes, document types)

### Challenge: Event Delivery Reliability

**Problem:** Events occasionally not reaching re:Search

**Solutions:**
- Implement message queue for async sending
- Add retry logic with exponential backoff
- Monitor event delivery success rate
- Set up alerts for delivery failures
- Log all events for troubleshooting

### Challenge: Security Events Delayed

**Problem:** Sealed case visible briefly before disappearing

**Solutions:**
- Send CaseSecurity events synchronously (not queued)
- Send immediately after commit, don't batch
- Monitor security event latency specifically
- Test security event flow thoroughly

### Challenge: Wrong Event Types

**Problem:** Events sent but data not refreshing correctly

**Solutions:**
- Review event type logic guide carefully
- Use most specific event type available
- Test each event type scenario
- Document decision logic clearly

### Challenge: Certificate Management

**Problem:** Certificate expires, causing production outage

**Solutions:**
- Set up expiration monitoring (alert at 60/30/7 days)
- Request renewal 90 days before expiration
- Test new certificate in test environment first
- Keep backup certificate for emergency

---

## Next Steps

- **Review technical details** → Read [Technical Setup](./technical-setup.md)
- **Learn about operations** → Read [Operations](./operations.md)
- **Understand the workflows** → Read [How It Works](./how-it-works.md)
- **Return to overview** → [ECF Mode Overview](./README.md)

---

**Need Help?**

Contact your Tyler Technical Project Manager for assistance with implementation planning and certification.

**Last Updated:** December 2025