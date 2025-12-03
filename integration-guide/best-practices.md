# Best Practices for Successful Integration

> **Breadcrumb:** [Home](../README.md) > [Integration Guide](./README.md) > Best Practices

Lessons learned from successful re:Search integrations. Follow these guidelines to avoid common pitfalls and ensure a smooth implementation.

---

## Planning & Preparation

### ✅ Choose the Right Integration Mode Early

**Why it matters:** Your integration mode determines timeline, resource requirements, and technical complexity.

**How to choose:**
- **Have EFM integration already?** → ECF Mode is natural fit
- **Using Enterprise Justice?** → CIP Mode designed for you
- **Limited API capabilities?** → Batch Mode may be best
- **Very small court?** → Non-Integrated Mode works

**Don't:**
- Rush the decision to "just get started"
- Choose based solely on timeline (wrong mode = longer overall time)
- Ignore your technical capabilities

**Do:**
- Assess your CMS's actual capabilities honestly
- Consult with your Tyler TPM before deciding
- Review the [Integration Modes comparison](./integration-modes/README.md)
- Consider long-term maintenance, not just initial build

---

### ✅ Understand Market Requirements Before Development

**Why it matters:** Market-specific requirements significantly affect implementation. Discovering them late causes rework.

**What to review:**
- **Case types allowed** in your jurisdiction
- **Event types required** for your case types
- **Security rules** and publication delays
- **Data mapping requirements** for your market

**Texas courts must review:**
- [JCIT case types](../market-standards/texas/case-types/README.md)
- [JCIT security requirements](../market-standards/texas/security/README.md) (31-day, 180-day delays)
- [JCIT event requirements](../market-standards/texas/event-types/README.md)

**Illinois courts must review:**
- [AOIC requirements](../market-standards/illinois/README.md)

**Timeline impact:**
- Reviewing requirements upfront: 1 week
- Discovering requirements during development: 2-4 weeks of rework

---

### ✅ Allocate Sufficient Time for Testing

**Why it matters:** Testing and certification take longer than most teams expect.

**Realistic timeline expectations:**

| Phase | Typical Duration | What Causes Delays |
|-------|------------------|-------------------|
| Development | 4-8 weeks | Depends on mode choice |
| **Testing** | **3-4 weeks** | Finding and fixing issues |
| **Certification** | **2-3 weeks** | Multiple test cycles common |
| Production prep | 1-2 weeks | Credential setup, cutover planning |

**Most common mistake:** Assuming testing can be done in 1 week

**Reality:**
- First test cycle reveals issues (always)
- Fixes take time to implement
- Retesting required after fixes
- Most integrations need 2-3 test cycles before certification

**Plan for:**
- Multiple test cycles
- Time to fix discovered issues
- Coordination with Tyler TPM for testing
- Buffer time for unexpected issues

---

### ✅ Engage Your Tyler TPM Early

**Why it matters:** Your Technical Project Manager is your primary resource for integration success.

**Engage TPM for:**
- Integration mode selection
- Environment access requests
- Technical questions during development
- Testing coordination
- Certification scheduling
- Go-live planning
- Post-launch support

**When to contact TPM:**
- **Planning phase** - Before making major decisions
- **Development phase** - When hitting technical roadblocks
- **Testing phase** - To coordinate testing schedule
- **Issues** - Any integration problems or questions

**Don't:**
- Wait until you're stuck to contact TPM
- Try to solve everything internally without asking
- Skip TPM involvement in certification
- Assume something without confirming with TPM

---

## Data Quality

### ✅ Map Case Types Accurately

**Why it matters:** Incorrect case type mapping causes indexing failures and compliance issues.

**The challenge:**
Your CMS has its own case type names. re:Search uses market-standard codes (JCIT for Texas, AOIC for Illinois, etc.). You must map between them.

**Example mapping (Texas):**
- Your CMS: "Criminal Felony" → JCIT Code: "CR"
- Your CMS: "Civil Lawsuit" → JCIT Code: "CV"
- Your CMS: "Divorce Case" → JCIT Code: "FM"

**Best practices:**
- Document all mappings clearly
- Validate mappings with your TPM before testing
- Plan for unmapped case types (what's the default?)
- Test with all case types your court uses
- Update mappings when new case types are added

**Common mistakes:**
- Guessing at mappings instead of confirming
- Not handling all case types in your CMS
- Forgetting to update mappings when case types change

---

### ✅ Ensure Complete Party and Attorney Information

**Why it matters:** Incomplete party/attorney data reduces search effectiveness and user satisfaction.

**Required party information:**
- Full names (First, Middle, Last)
- Party roles (Plaintiff, Defendant, etc.)
- All parties on the case (don't omit any)

**Required attorney information:**
- Full attorney names
- Bar numbers
- Which party each attorney represents
- Contact information (if available)

**Impact of incomplete data:**
- Cases harder to find in search
- Users can't identify correct case
- Missing attorney information frustrates legal community
- Poor user experience reflects badly on court

**Quality checks:**
- Review sample GetCase responses for completeness
- Test search with typical user queries
- Verify all parties appear in search results
- Confirm attorney associations are clear

---

### ✅ Maintain Consistent Case Numbering

**Why it matters:** Case numbers are the primary identifier for cases.

**Best practices:**
- Use the same case number format in re:Search as your CMS
- Don't reformat or modify case numbers
- Include court identifier if needed for uniqueness
- Be consistent across all cases

**Example good case numbers:**
- CV-2024-001234 (consistent format)
- 2024-CR-05678 (consistent format)
- FM24-000456 (consistent format)

**Example problematic patterns:**
- Mixing formats (some with dashes, some without)
- Changing format over time
- Different format for different case types
- Ambiguous numbers that could match multiple cases

**Why consistency matters:**
- Users type case numbers to search
- Minor format differences break searches
- Links from other systems use case numbers
- Court staff rely on consistent numbering

---

### ✅ Keep Security Settings Synchronized

**Why it matters:** Security mismatches expose confidential information or hide public records.

**The synchronization challenge:**
Your CMS is the authoritative source for case/document security. re:Search must always reflect current CMS security settings.

**Critical rules:**
- GetCase must return current security (not cached/stale)
- Send CaseSecurity event immediately when security changes
- Send DocumentSecurity event immediately for document changes
- Don't delay security events (even by minutes)

**Test scenarios:**
- Seal a public case → Verify it becomes non-public immediately
- Unseal a case → Verify it becomes public (per market rules)
- Seal individual documents → Verify documents become restricted
- Security changes in batch → All changes should process quickly

**Common mistakes:**
- Caching security settings for too long
- Not sending events when security changes
- Inconsistent security between GetCase and GetDocument
- Forgetting to test security changes

---

### ✅ Include All Relevant Case Information

**Why it matters:** "Optional" fields often improve search quality and user experience.

**Don't skip these fields:**
- Judge assignments (users search by judge)
- Docket entries (provides case context)
- Case status (Active, Disposed, etc.)
- Filing dates (critical for searches)
- Document descriptions (helps identify documents)

**Fields that improve searchability:**
- Party addresses (helps distinguish similar names)
- Attorney contact information (users need to reach attorneys)
- Hearing dates (users look for upcoming hearings)
- Case cross-references (related cases)

**Balance to strike:**
- Include everything that helps users find/understand cases
- Don't include confidential information
- Follow market rules for what should be public
- When in doubt, check with your TPM

---

## Event Management

### ✅ Send Events for All Material Changes

**Why it matters:** Missing events mean re:Search has stale data.

**What counts as a "material change":**
- New case filed
- Party added or modified
- Attorney added or modified
- Document filed
- Case security changed
- Document security changed
- Case disposition entered
- Judge reassigned
- Case deleted/expunged

**Don't skip events for:**
- "Small" changes (like name corrections)
- Changes made outside normal workflow
- Bulk updates or data corrections
- Changes made by administrators
- System-generated changes

**The golden rule:** When in doubt, send an event. re:Search handles duplicate/extra events well.

---

### ✅ Send Security Events Immediately

**Why it matters:** Security changes need immediate effect to protect confidential information.

**Security events that need immediate transmission:**
- CaseSecurity (case sealed/unsealed)
- DocumentSecurity (document sealed/unsealed)

**Don't:**
- Queue security events with other events
- Wait for batch processing window
- Delay security events for any reason

**Do:**
- Send security events as soon as security changes in CMS
- Use high-priority queue for security events (if you have priority system)
- Monitor security event processing closely
- Test security change timing during certification

**Why this matters:**
- Sealed case visible for hours = confidential information exposed
- Unsealed case still hidden = public denied access to public records
- Security is time-sensitive in ways other data isn't

---

### ✅ Use Appropriate Event Types

**Why it matters:** Correct event types help re:Search process changes efficiently.

**Match event type to change:**
- New case? → CaseFiling
- Party changed? → CaseParty
- Attorney changed? → CasePartyAttorney
- Document added? → DocumentFiling
- Case sealed? → CaseSecurity
- Case disposed? → CaseDisposition

**Don't:**
- Use generic event type for everything
- Use wrong event type because it "works"
- Skip event types required by your market

**When multiple things change:**
Send multiple events. re:Search consolidates them efficiently.

**Example:**
If you update a party name AND add an attorney in one transaction, send both CaseParty and CasePartyAttorney events.

[Complete event type guide →](../implementation-playbook/event-type-logic.md)

---

### ✅ Monitor Event Processing Health

**Why it matters:** Event queue problems cause synchronization delays.

**What to monitor:**
- **Queue depth** - How many events are pending?
- **Failed events** - Are events failing to send?
- **Processing time** - How long from event to indexed?
- **Event gaps** - Are there periods with no events?

**Warning signs:**
- Queue depth steadily increasing (not draining fast enough)
- High failure rate (>5% of events failing)
- Long processing times (>10 minutes from event to indexed)
- No events during business hours (generation may be broken)

**Set up monitoring:**
- Dashboard showing queue health
- Alerts for queue depth thresholds
- Alerts for high failure rates
- Daily summary of event activity

---

## Security & Access Control

### ✅ Understand Your Market's Security Model

**Why it matters:** Each market has different security rules and requirements.

**Texas (JCIT) specific:**
- Criminal cases: 31-day publication delay
- Sealed documents: 180-day seal period
- Role-based access (Public, Attorney, Registered User, Court Staff)
- County-based registered user visibility

**Illinois (AOIC) specific:**
- Review [Illinois security requirements](../market-standards/illinois/security/README.md)

**Other markets:**
- Universal security model (Public/Sealed/Confidential)
- Simpler role structure

**Don't assume:**
- Security works the same in all markets
- You can skip market-specific requirements
- Rules are the same as your current system

**Do:**
- Read your market's security documentation completely
- Test all security scenarios in your market
- Understand publication delays (if applicable)
- Clarify any questions with your TPM

---

### ✅ Enforce Security at Document Retrieval

**Why it matters:** Document security must be checked when document is requested, not just when indexed.

**The principle:**
GetDocument must enforce the same security rules as GetCase. Never assume a document is safe to return without checking security.

**Check at GetDocument time:**
- Current case security level
- Current document security level (if different from case)
- Requesting user's role
- Whether user has appropriate access

**Why check again at retrieval:**
- Security may have changed since indexing
- Prevents race conditions
- Adds defense-in-depth
- Required for certification

**Test thoroughly:**
- Public user requesting sealed document → Access denied
- Attorney of record requesting sealed document → May allow (check market rules)
- Court staff requesting any document → Allow

---

### ✅ Test Security Enforcement Thoroughly

**Why it matters:** Security bugs expose confidential information - the highest risk area.

**Test all security scenarios:**

**Sealed case visibility:**
- Create sealed case
- Verify public cannot see it
- Verify court staff can see it
- Verify attorneys of record can see it (if market allows)

**Mixed security in case:**
- Public case with sealed document
- Verify case is searchable
- Verify sealed document is not accessible

**Security changes:**
- Public case → Seal it → Verify becomes non-public immediately
- Sealed case → Unseal it → Verify becomes public (per market rules)

**Publication delays (if applicable):**
- Criminal case (Texas) → Verify 31-day delay
- Test different user roles during delay period

**Document retrieval security:**
- Request sealed document as public user → Access denied
- Request public document → Access granted
- Request document from sealed case → Access denied appropriately

---

### ✅ Never Make Documents More Visible Than Case

**Why it matters:** This is a fundamental security rule that must never be violated.

**The rule:**
If case is Sealed, all documents must be at least Sealed. If case is Confidential, all documents must be at least Confidential.

**Valid combinations:**
- Public case, Public document ✅
- Public case, Sealed document ✅
- Sealed case, Sealed document ✅
- Sealed case, Confidential document ✅

**Invalid combinations:**
- Sealed case, Public document ❌ (document must be at least Sealed)
- Confidential case, Public document ❌ (document must be Confidential)
- Confidential case, Sealed document ❌ (document must be Confidential)

**Implementation:**
Validate that document security is never less restrictive than case security. If your CMS allows this configuration, enforce the rule in GetCase response.

---

## Performance & Reliability

### ✅ Optimize GetCase Response Times

**Why it matters:** Slow GetCase responses cause indexing delays and timeout errors.

**Target performance:**
- Simple cases (few parties/documents): < 1 second
- Average cases: < 3 seconds
- Complex cases (many parties/documents): < 5 seconds

**If exceeding targets:**
- Review database query performance
- Add appropriate indexes
- Optimize how data is retrieved
- Consider caching (with short TTL)
- Consult with your database administrator

**Don't:**
- Ignore slow performance ("it works")
- Assume timeouts are re:Search's problem
- Return partial data to meet timing (return error instead)

**Test performance:**
- Test with real case complexity (not just simple test cases)
- Test with production-sized database
- Monitor performance during certification
- Set up ongoing performance monitoring

---

### ✅ Implement Retry Logic for Failed Events

**Why it matters:** Network issues happen - they shouldn't cause permanent data loss.

**The pattern:**
Rather than sending events directly to re:Search, use a queue with retry capability.

**Queue behavior:**
- Event written to queue after CMS change
- Background process sends events from queue
- Failed events automatically retried (3-5 attempts)
- Exponential backoff between retries
- Permanently failed events flagged for investigation

**Benefits:**
- Temporary network issues don't cause data loss
- Integration survives re:Search maintenance windows
- Clear visibility into failed events
- Automatic recovery from transient issues

**Without retry logic:**
- Network blip → Event lost → Case never updated in re:Search
- re:Search maintenance → Events lost during maintenance
- No visibility into why case wasn't updated

---

### ✅ Log All API Interactions

**Why it matters:** Logs are essential for troubleshooting integration issues.

**What to log:**
- Timestamp of API call
- API name (GetCase, GetDocument, NotifyCaseEvent)
- Case tracking ID or document ID
- Success or failure
- Response time
- Error message (if failed)

**Why logging matters:**
- Troubleshoot why case isn't updating
- Investigate performance issues
- Understand event patterns
- Support issue diagnosis
- Identify trends and problems

**Log retention:**
Keep logs for at least 90 days, preferably longer.

**Review logs:**
- During troubleshooting
- Weekly for trends
- Monthly for performance analysis
- When users report issues

---

### ✅ Set Up Monitoring and Alerting

**Why it matters:** Catch problems before users report them.

**Monitor:**
- Event queue health
- API error rates
- Response times
- Failed event count
- Integration lag (time from change to indexed)

**Alert on:**
- Queue depth exceeds threshold
- Error rate spikes
- No events sent during business hours
- Response times degrading
- Repeated failures for same case

**Response procedures:**
Document what to do when each alert fires. Who investigates? What are the troubleshooting steps? When to escalate?

---

## Testing & Certification

### ✅ Test All Event Types Before Certification

**Why it matters:** Tyler certification validates all event types work correctly.

**Event types to test:**
- CaseFiling (new case)
- CaseParty (party changes)
- CasePartyAttorney (attorney changes)
- DocumentFiling (new documents)
- CaseDisposition (case closure)
- CaseSecurity (case security changes)
- DocumentSecurity (document security changes)
- CaseReassigned (judge changes)
- CaseDelete or CaseExpunge (case removal)

**For each event type:**
1. Perform the action in your CMS
2. Verify event is generated
3. Verify event reaches re:Search
4. Verify GetCase returns updated data
5. Verify change appears in re:Search search/display

**Don't:**
- Test only the "common" event types
- Skip event types you "rarely use"
- Assume similar events work the same

---

### ✅ Test With Real Case Data

**Why it matters:** Test cases don't reveal real-world complexity.

**Use real data for testing:**
- Actual case types from your court
- Realistic party counts (some cases have 10+ parties)
- Real document volumes
- Actual judge names and assignments
- Production-like case numbers

**Why test data isn't sufficient:**
- Test cases are simpler than reality
- Doesn't stress performance realistically
- Misses edge cases in real data
- Doesn't test your actual case type mappings

**Data sanitization:**
If using production data for testing, sanitize:
- Party names
- Addresses
- Sensitive case details
- But keep data structure and complexity

---

### ✅ Validate Security Enforcement

**Why it matters:** Security is the highest-risk area - test it extensively.

**Required security tests:**
- Sealed case not visible to public
- Sealed document not accessible to public
- Public case with sealed document (mixed security)
- Security changes take effect immediately
- Publication delays (if your market has them)
- Role-based access (all roles in your market)
- GetDocument enforces security correctly

**Test negative cases:**
- Try to access sealed case as public user → Should fail
- Try to retrieve sealed document → Should fail
- Try to access case during publication delay → Should fail for public

**Test positive cases:**
- Court staff can access sealed cases
- Attorneys of record can access their cases
- Public can access public cases after delays

---

### ✅ Work Through Certification Methodically

**Why it matters:** Certification ensures your integration meets all requirements.

**Certification process:**
1. Complete development
2. Internal testing
3. Request certification from TPM
4. Tyler performs certification tests
5. Issues identified → Fix and retest
6. Certification approved
7. Production deployment

**Typical certification timeline:**
- First certification attempt: Issues found (normal)
- Fix issues: 1-2 weeks
- Second certification attempt: Usually successful
- Some integrations need 3rd attempt

**Expect to iterate:**
- First attempt rarely passes completely
- Issues found are learning opportunities
- Each cycle improves the integration
- Don't be discouraged by initial issues

**Common certification issues:**
- Missing required fields in GetCase
- Event types not working correctly
- Security not enforced properly
- Performance below targets
- Case type mapping errors

---

### ✅ Plan for Regression Testing

**Why it matters:** Future changes shouldn't break working functionality.

**When to regression test:**
- After fixing certification issues
- After CMS upgrades
- After adding new features
- Periodically (quarterly recommended)

**What to test:**
- All event types still working
- Security still enforced correctly
- Performance still acceptable
- New case types handled correctly

**Maintain test cases:**
- Document test scenarios
- Keep test data current
- Update tests when requirements change
- Automate where possible

---

## Going Live

### ✅ Start With Pilot or Phased Rollout

**Why it matters:** Reduces risk by limiting initial scope.

**Rollout strategies:**

**Pilot court approach:**
- Start with one small court
- Monitor closely for 2-4 weeks
- Validate data quality
- Fix any issues found
- Expand to additional courts

**Case type phasing:**
- Start with simplest case type (often civil)
- Add more complex case types incrementally
- Save highest-volume case types for last
- Validate each phase before expanding

**Benefits of phased approach:**
- Catch issues with limited impact
- Build confidence gradually
- Staff learns system incrementally
- Easier to troubleshoot with smaller scope

**When to use full deployment:**
- Small court (limited risk)
- Very high confidence in integration
- Pressure for immediate statewide coverage

---

### ✅ Monitor Closely in First Weeks

**Why it matters:** Issues surface quickly after go-live with real data and real users.

**Enhanced monitoring period:**

**Week 1-2 (Intensive):**
- Check integration health multiple times daily
- Review all error logs daily
- Monitor queue depth every few hours
- Validate case data quality daily
- Be available for rapid fixes

**Week 3-4 (Active):**
- Daily health checks
- Review error trends
- Monitor performance metrics
- Continue data quality validation

**Month 2+ (Standard):**
- Regular health checks per schedule
- Weekly performance review
- Monthly optimization review
- Ongoing quality validation

**What to watch for:**
- Event processing delays
- Error rate increases
- Security issues
- User complaints about missing cases
- Performance degradation

---

### ✅ Have a Rollback Plan

**Why it matters:** Serious issues may require temporary rollback.

**Rollback options:**

**Option 1: Disable event generation**
Stop sending events temporarily while fixing critical issue. re:Search keeps existing data, just stops receiving updates.

**Option 2: Route to legacy system**
If users need access during fix, route searches to legacy system temporarily.

**Option 3: Manual entry fallback**
Court staff manually enter critical cases while integration is repaired.

**Rollback criteria:**
Define in advance what issues justify rollback:
- Security breach or confidential data exposed
- Integration completely non-functional
- Widespread incorrect data
- Critical performance issues

**Don't rollback for:**
- Minor issues affecting few cases
- Issues with clear workaround
- Problems that can be fixed quickly

---

### ✅ Communicate With Stakeholders

**Why it matters:** Keep everyone informed about integration status.

**Who to communicate with:**
- Court staff (what to expect, how to report issues)
- Court leadership (status updates)
- Tyler TPM (all issues and changes)
- Users (if public announcement appropriate)

**Communication timing:**

**Before go-live:**
- What's changing
- When it's happening
- What users should know
- How to get help

**During go-live:**
- Deployment status
- Any issues encountered
- When fully operational

**After go-live:**
- Status updates (especially first week)
- Issue resolution status
- Performance metrics
- User feedback

---

### ✅ Document Lessons Learned

**Why it matters:** Helps with future maintenance and other courts' integrations.

**Capture:**
- What went well
- What was harder than expected
- Issues discovered and how they were resolved
- Time estimates vs. actual time
- Resource requirements
- Helpful resources and contacts

**Use lessons learned for:**
- Planning future phases/courts
- Onboarding new team members
- Improving integration over time
- Sharing with other courts (sanitized)

---

## Ongoing Maintenance

### ✅ Keep Integration Healthy

**Why it matters:** Integration requires ongoing attention, not just initial deployment.

**Regular maintenance tasks:**

**Daily:**
- Check integration health dashboard
- Review error logs for issues
- Monitor queue depth

**Weekly:**
- Review error trends
- Check performance metrics
- Validate recent cases indexed correctly

**Monthly:**
- Performance review and optimization
- Review and address failed events
- Update documentation as needed
- Team review of any recurring issues

**Quarterly:**
- Comprehensive integration review
- Regression testing
- Review with Tyler TPM
- Update case type mappings if needed

---

### ✅ Plan for CMS Upgrades

**Why it matters:** CMS upgrades can break re:Search integration.

**Before CMS upgrade:**
- Review upgrade release notes for impacts
- Test integration in non-production with upgraded CMS
- Validate all event types still work
- Check performance hasn't degraded
- Verify case type mappings still correct

**After CMS upgrade:**
- Monitor integration health closely
- Run regression tests
- Validate security still enforced
- Check for new errors in logs

**Coordinate with Tyler:**
- Notify TPM of planned CMS upgrades
- Discuss potential integration impacts
- Plan testing approach
- Schedule re-certification if needed

---

### ✅ Add New Case Types Carefully

**Why it matters:** New case types need proper mapping and testing.

**Process for new case type:**
1. Determine market-standard code for case type
2. Validate mapping with Tyler TPM
3. Update case type configuration
4. Test in non-production environment
5. Verify security rules apply correctly
6. Deploy to production
7. Monitor first cases of new type

**Don't:**
- Add case types without verifying market code
- Skip testing new case types
- Assume new type works like existing types

---

## Getting Help

### ✅ Know When to Ask for Help

**Why it matters:** Don't waste days on issues your TPM can resolve in minutes.

**Contact your TPM for:**
- Questions about requirements
- Clarification on event types
- Case type mapping questions
- Security rule interpretation
- Certification scheduling
- Any integration issues you can't resolve

**Don't:**
- Assume you understand everything
- Guess at requirements or mappings
- Struggle alone with technical issues
- Wait until the last minute for help

**How to get effective help:**
- Provide specific case tracking IDs
- Include relevant logs
- Describe what you've already tried
- Be clear about what's not working

---

## Related Documentation

### For Integration Process
→ [Onboarding Guide](./onboarding/README.md) - Step-by-step process  
→ [Integration Modes](./integration-modes/README.md) - Choose your approach

### For Technical Understanding
→ [Implementation Playbook](../implementation-playbook/README.md) - How re:Search works  
→ [API Reference](../api-reference/README.md) - Technical specifications

### For Market Requirements
→ [Market Standards](../market-standards/README.md) - Jurisdiction-specific rules

### For Problem Solving
→ [Troubleshooting Guide](../troubleshooting/README.md) - Common issues  
→ [Common Mistakes](../implementation-playbook/common-mistakes.md) - What to avoid

---

**Last Updated:** January 2025