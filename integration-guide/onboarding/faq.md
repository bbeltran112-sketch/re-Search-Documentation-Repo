# Frequently Asked Questions

> **⚠️ UNDER CONSTRUCTION**  
> This documentation is currently being updated. Content may be incomplete or subject to change.  
> For immediate assistance, contact your Technical Project Manager or the BIS team at [bryce.beltran@tylertech.com](mailto:bryce.beltran@tylertech.com)

**Navigation:**  
[Home](../../README.md) › [Client Documentation](../README.md) › [Client Onboarding](./README.md) › FAQ

---

## General Questions

### How long does onboarding typically take?

**Answer:** 8-16 weeks depending on integration mode:
- **Batch Mode:** 8-12 weeks
- **ECF Mode:** 12-16 weeks  
- **CIP Mode:** 10-14 weeks
- **Non-Integrated:** 2-4 weeks

Your Technical Project Manager will provide a customized timeline during kickoff based on your CMS complexity and court requirements.

---

### Can we start development before completing Phase 1?

**Answer:** We **strongly recommend** completing all preparation phases before beginning development to avoid rework. However, if timeline is critical, discuss parallel activities with your TPM.

**Common parallel activities:**
- CMS preparation while waiting for credentials
- Internal architecture review during Phase 1
- Developer onboarding during environment access setup

**Avoid:**
- Writing integration code before obtaining test credentials
- Making API assumptions without documentation review
- Committing to architecture before mode selection

---

### What if we need to change integration modes mid-project?

**Answer:** Contact your TPM immediately. Mode changes are possible but will impact:
- **Timeline:** +2-4 weeks typically
- **Effort:** Development rework required
- **Certification:** Must re-certify with new mode

**When mode changes make sense:**
- CMS capabilities discovered during development
- Court requirements change significantly
- Performance issues identified in testing

---

### Do we need separate onboarding for each court?

**Answer:** Generally **no**. Once your CMS is certified for one court, additional courts using the same CMS can be onboarded much faster:
- **First court:** Full 8-16 week process
- **Additional courts:** 1-2 weeks per court

**What's faster for subsequent courts:**
- No development work required
- Abbreviated testing (court-specific configuration only)
- Faster certification (CMS already validated)

**What still takes time:**
- Court-specific configuration
- Court data validation
- Historical backfill (if required)

---

### What happens if we fail certification?

**Answer:** Your TPM will provide detailed feedback on all issues found. You'll:
1. **Remediate issues** based on feedback
2. **Re-test** in your environment
3. **Submit for re-certification**

**Statistics:**
- 70% of vendors pass on first attempt
- 25% pass on second attempt
- 5% require three or more attempts

**Most common reasons for failure:**
- Sealed case handling errors
- Incomplete data mapping
- Missing error handling
- Performance issues

---

## Technical Questions

### Can we access production before certification?

**Answer:** **No.** Production credentials are only provided after:
- ✅ All test scenarios passed
- ✅ Data quality validated
- ✅ Security handling verified
- ✅ Performance benchmarks met
- ✅ BIS team certification approval

This policy protects production data quality and court operations.

---

### What if our CMS doesn't support the recommended integration mode?

**Answer:** Discuss with your TPM immediately. Options include:
- **Alternative mode** that matches your capabilities
- **CMS enhancement** to support recommended mode
- **Custom solution** (evaluated case-by-case)
- **Non-Integrated mode** as last resort

Your TPM will help evaluate the best path forward based on timeline, budget, and technical constraints.

---

### How do we handle historical data backfill?

**Answer:** Historical backfill approach depends on integration mode:

**Batch Mode:**
- Submit full historical dataset as initial batch
- Typical timeline: 1-2 weeks for large courts

**ECF Mode:**
- Use Batch Mode for historical data
- Switch to ECF Mode for ongoing updates
- "Hybrid approach" - discuss with TPM

**Best practices:**
- Plan backfill during Phase 3 (Development)
- Test with sample historical data first
- Schedule during low-activity period
- Monitor data quality closely

---

### What network/firewall requirements are needed?

**Answer:** Requirements vary by integration mode:

**Batch Mode:**
- **Outbound HTTPS (443)** to S3 or SFTP endpoints
- **AWS S3:** `s3.amazonaws.com` domain
- **SFTP:** `sftp.tylerhost.net` on port 22

**ECF Mode:**
- **Outbound HTTPS (443)** for NotifyCaseEvent
- **Inbound HTTPS (443)** for GetCase/GetDocument (if CMS hosts these)
- **mTLS certificates** installed and trusted

**All modes:**
- TLS 1.2 or higher required
- Modern cipher suites supported
- Certificate chain validation enabled

**Detailed requirements:** Provided by TPM during Phase 2

---

### How do we handle certificate rotation?

**Answer:**

**S3/SFTP credentials:**
- Rotated every 90-180 days
- TPM provides new credentials 30 days before expiration
- Update in your secrets management system

**mTLS certificates (ECF Mode):**
- Rotated every 12-18 months
- TPM provides new certificate 60 days before expiration
- Test new certificate in non-production first
- Coordinate cutover with TPM

**Best practice:** Set up automated alerts for credential expiration.

---

## Testing & Certification Questions

### What test data is provided?

**Answer:** During Phase 2, you'll receive:
- **Sample court configuration**
- **Test case dataset** (public, confidential, sealed)
- **Test filing scenarios**
- **Expected validation results**

**You must provide:**
- Your own test cases from your CMS
- Edge case scenarios specific to your implementation
- Performance test data

---

### How long does certification review take?

**Answer:**
- **Submission to initial feedback:** 3-5 business days
- **Re-certification (after fixes):** 2-3 business days

**To expedite:**
- Submit complete test results (don't skip scenarios)
- Include detailed logs for any errors
- Document any known limitations upfront
- Respond quickly to feedback

---

### Can we skip testing scenarios we don't support?

**Answer:** **No.** All required test scenarios must pass. If your CMS doesn't support a scenario:
1. **Discuss with TPM** during Phase 2
2. **Document limitation** formally
3. **Implement workaround** (if possible)
4. **Accept reduced functionality** (if unavoidable)

Skipped scenarios will result in automatic certification failure.

---

## Operations Questions

### How much does onboarding support cost?

**Answer:** Basic onboarding support is **included** in your re:Search contract:
- TPM assignment
- Test environment access
- Standard certification process
- Documentation and training materials

**Additional services** (if needed) are discussed with your TPM:
- Accelerated timelines
- Custom development support
- Extended post-launch support

---

### What support is available post-go-live?

**Answer:**

**First 30 days:**
- Daily monitoring by BIS team
- Rapid response to issues
- Weekly check-ins with TPM

**Ongoing:**
- Standard support through Tyler Partner Portal
- Escalation path for critical issues
- Quarterly business reviews (large deployments)

See [Post-Go-Live Support](./08-post-go-live-support.md) for details.

---

### Can we make changes after go-live?

**Answer:** **Yes**, but changes require coordination:

**Non-breaking changes** (approved quickly):
- Bug fixes
- Performance optimizations
- Additional court onboarding

**Breaking changes** (require re-certification):
- Integration mode change
- Major CMS version upgrade
- Significant data model changes

Always coordinate changes with your TPM before implementation.

---

## Still Have Questions?

**For onboarding questions:**  
Contact your assigned Technical Project Manager

**For technical implementation questions:**  
Review the [Technical Onboarding Guide](./05-technical-onboarding.md) or [API Reference](../../technical-documentation/api-reference/README.md)

**For operational questions:**  
See the [Support Playbook](../../technical-documentation/support-playbook/README.md)

**For general inquiries:**  
BIS Team – Bryce Beltran  
📧 [bryce.beltran@tylertech.com](mailto:bryce.beltran@tylertech.com)

---

[← Back to Client Onboarding](./README.md)