# Delays and Timing

> **Breadcrumb:** [Home](../README.md) > [Implementation Playbook](./README.md) > Delays and Timing

Publication delays, timing rules, and when cases/documents become visible to different user roles.

---

## Table of Contents

### Understanding Publication Delays
- [What Are Publication Delays?](#what-are-publication-delays)
- [Why Delays Exist](#why-delays-exist)
- [Who Applies the Delay?](#who-applies-the-delay)

### Delay Types
- [Market-Based Delays](#market-based-delays)
  - Texas Criminal Case Delay (31 Days)
  - Sealed Case Period (180 Days)
  - Custom Market Rules

- [Case-Level Delays](#case-level-delays)
  - Filing Date-Based
  - Case Type-Based
  - Security-Based

### Texas (JCIT) Timing Rules
- [Texas Criminal Case Publication Delay](#texas-criminal-case-publication-delay)
  - 31-Day Delay Rule
  - Who Can See During Delay
  - What Happens After 31 Days
  - Court Staff Access During Delay

- [Texas Sealed Case Period](#texas-sealed-case-period)
  - 180-Day Seal Period
  - Automatic Unsealing
  - Manual Extension

### Role-Based Access During Delays
- [Who Can See What, When](#who-can-see-what-when)
  - Public Users
  - Registered Users
  - Court Staff
  - Parties/Attorneys

### How Delays Work Technically
- [re:Search Delay Calculation](#research-delay-calculation)
- [CMS Responsibilities](#cms-responsibilities)
- [What CMS Sends vs. What re:Search Enforces](#what-cms-sends-vs-what-research-enforces)

### Common Scenarios
- [Scenario 1: Texas Criminal Case Filed](#scenario-1-texas-criminal-case-filed)
- [Scenario 2: Case Sealed by Court Order](#scenario-2-case-sealed-by-court-order)
- [Scenario 3: Sealed Case Approaching 180 Days](#scenario-3-sealed-case-approaching-180-days)
- [Scenario 4: Document Filed During Publication Delay](#scenario-4-document-filed-during-publication-delay)

### Testing Delay Rules
- [How to Test Publication Delays](#how-to-test-publication-delays)
- [Verifying Role-Based Access](#verifying-role-based-access)

### Troubleshooting
- [Common Timing Issues](#common-timing-issues)
- [When Cases Don't Appear on Time](#when-cases-dont-appear-on-time)

---

## What Are Publication Delays?

Publication delays are time periods between when a case is filed and when it becomes visible to the public in re:Search.

**Key Concept:** The case exists in the CMS and can be processed immediately by court staff, but public access is restricted for a period of time based on market-specific rules.

**Example:**
```
Texas criminal case filed: October 1, 2024
Case visible to court staff: Immediately (October 1)
Case visible to public: November 1, 2024 (after 31-day delay)
```

---

## Why Delays Exist

Publication delays serve important legal and operational purposes:

**1. Privacy Protection**
- Criminal defendants have privacy during initial proceedings
- Allows time for notification before public exposure
- Protects sensitive case information during critical early period

**2. Legal Process**
- Time for service of process
- Time for parties to respond
- Time for preliminary motions

**3. Jurisdictional Requirements**
- State laws mandate specific delay periods
- Market standards enforce consistent rules
- Court rules may require delays for certain case types

---

## Who Applies the Delay?

**re:Search applies publication delays automatically.**

The CMS does NOT need to calculate or enforce publication delays. The CMS simply:
1. Sends the case type correctly
2. Sends the filing date accurately
3. Sets the case security appropriately

re:Search then applies market-specific delay rules based on:
- Case type (e.g., CR for criminal)
- Filing date (delay calculated from this date)
- Market standards (e.g., JCIT rules for Texas)
- Case security level

---

## Market-Based Delays

Different markets (states/jurisdictions) have different publication delay rules. re:Search enforces these rules automatically when properly configured for each market.

### Texas (JCIT) Publication Rules

Texas courts following JCIT standards have two primary delay rules:

**1. Criminal Case Publication Delay: 31 Days**
- Applies to: Criminal cases (case type starting with "CR")
- Duration: 31 days from filing date
- Purpose: Privacy protection during initial proceedings

**2. Sealed Case Period: 180 Days**
- Applies to: Cases marked as "Sealed"
- Duration: Up to 180 days
- Purpose: Extended privacy period for sensitive cases
- Can be unsealed earlier by court order
- Can be extended beyond 180 days by court order

### Other Markets

Other markets may have:
- Different delay periods
- Different case types subject to delays
- Different role-based access rules
- Custom publication rules

[See Market Standards Documentation →](../market-standards/README.md)

---

## Texas Criminal Case Publication Delay

### 31-Day Delay Rule

**What Happens:**
```
Day 0 (Filing Date):
├─ Case filed in CMS
├─ Case sent to re:Search
├─ Visible to: Court staff, parties, attorneys
└─ NOT visible to: Public, registered users

Days 1-30:
├─ Delay period continues
├─ Case data can be updated
├─ Documents can be filed
├─ Still NOT visible to public
└─ Court staff continue to see case

Day 31:
├─ Delay period expires
└─ Case becomes visible to public (if not sealed)
```

### Who Can See During Delay

**During 31-day delay period:**

| Role | Can See Case? | Can See Documents? | Notes |
|------|---------------|-------------------|-------|
| **Public** | ❌ No | ❌ No | Cannot find in search, cannot access |
| **Registered User** | ❌ No | ❌ No | Same restrictions as public |
| **Court Staff** | ✅ Yes | ✅ Yes | Full access immediately |
| **Case Parties** | ✅ Yes | ✅ Yes | Can see their own case |
| **Case Attorneys** | ✅ Yes | ✅ Yes | Can see their client's case |

### What Happens After 31 Days

**If case is Public security:**
- Case becomes visible to all users
- Appears in public search results
- Documents accessible (unless individually sealed)

**If case is Sealed security:**
- Case remains hidden from public
- 180-day sealed period begins (or continues)
- Only authorized users can access

### Court Staff Access During Delay

Court staff always have immediate access:
- Can search for case immediately after filing
- Can view all case details
- Can access all documents
- Can process case normally

**This allows courts to:**
- Review filings immediately
- Process documents
- Schedule hearings
- Conduct business without waiting for public visibility

---

## Texas Sealed Case Period

### 180-Day Seal Period

**What Happens:**
```
Day 0 (Seal Order Issued):
├─ Case security set to "Sealed"
├─ CMS sends CaseSecurity event
├─ re:Search updates case security
├─ Case hidden from public
└─ 180-day countdown begins

Days 1-179:
├─ Case remains sealed
├─ Only authorized users can access
├─ Can be unsealed earlier by court order
└─ Can be extended by court order

Day 180:
├─ Seal period expires (if not extended)
├─ Case typically becomes public
└─ Unless court orders continued seal
```

### Automatic Unsealing

**At 180 days (if no action taken):**
- Case automatically becomes public
- Appears in public search results
- Documents become accessible (unless individually sealed)

**However:**
- Court can unseal earlier by ordering case security changed to Public
- Court can extend seal beyond 180 days by court order
- Extension requires court action before expiration

### Manual Extension

**To extend seal beyond 180 days:**
1. Court issues order extending seal
2. CMS keeps case security as "Sealed"
3. No CaseSecurity event sent (security unchanged)
4. Case remains sealed indefinitely (or until court orders unsealing)

**To unseal before 180 days:**
1. Court issues order to unseal
2. CMS changes case security to "Public"
3. CMS sends CaseSecurity event immediately
4. re:Search updates case to public visibility
5. Case appears in public search

---

## Role-Based Access During Delays

Different user roles have different access during publication delay periods.

### Public Users

**Access Rights:**
- Can only see cases after publication delay expires
- Can only see cases with Public security
- Cannot access sealed or delayed cases
- Cannot access sealed documents

**During Delays:**
- Cases don't appear in search results
- Direct case number access returns "not found"
- No indication case exists

### Registered Users

**Access Rights:**
- Similar to public users in most markets
- May have enhanced access in some jurisdictions
- Can access own cases regardless of delay
- Can access cases where they are party/attorney

**Texas JCIT Rules:**
- Registered users treated same as public during 31-day delay
- Registered users CAN see sealed cases (role-based access)
- Registered users get full access after delay expires

### Court Staff

**Access Rights:**
- Immediate access to all cases
- Can see cases during publication delay
- Can see sealed cases
- Can see all documents
- Full search access

**Why:**
- Must process filings immediately
- Must review documents
- Must schedule hearings
- Must conduct court business

### Parties/Attorneys

**Access Rights:**
- Can see own cases immediately (no delay)
- Can see all documents in own cases
- Access not affected by publication delays
- Access not affected by sealed status (for their own cases)

**Why:**
- Need immediate access to their case information
- Need to review filings
- Need to prepare responses
- Constitutional right to access own case

---

## re:Search Delay Calculation

re:Search calculates publication delays automatically based on case metadata.

### How re:Search Determines Visibility

**Step 1: Receive Case Data**
```
re:Search receives NotifyCaseEvent
Calls GetCase to retrieve case details
Extracts:
├─ Case type (e.g., "CR" for criminal)
├─ Filing date (e.g., "2024-10-01")
└─ Case security (e.g., "Public")
```

**Step 2: Apply Market Rules**
```
Determine market: Texas (JCIT standards)
Check case type: CR (criminal)
Lookup rule: Criminal cases → 31-day delay
```

**Step 3: Calculate Publication Date**
```
Filing date: October 1, 2024
Add delay: 31 days
Publication date: November 1, 2024
```

**Step 4: Enforce Access Rules**
```
If current date < publication date:
├─ Hide from public search
├─ Deny public access
└─ Allow court staff/party access

If current date >= publication date:
└─ Apply normal security rules (Public/Sealed)
```

### Date Calculation Details

**Filing Date Source:**
- Comes from GetCase response
- Typically: `<nc:CaseFiledDate>` or similar field
- Must be accurate filing date from CMS

**Delay Calculation:**
- Delay added to filing date
- Publication date = filing date + delay period
- Time of day typically midnight (start of day)

**Current Date Comparison:**
- re:Search compares current date/time to publication date
- If before publication date → delay applies
- If on/after publication date → delay expired

---

## CMS Responsibilities

The CMS does NOT calculate or enforce publication delays, but CMS must provide accurate data for re:Search to calculate them correctly.

### What CMS Must Send

**1. Correct Case Type**
```xml
<tyler:CaseTypeText>CR</tyler:CaseTypeText>
```
- Must match market standard codes
- Texas: Use JCIT-approved case type codes
- Criminal cases must start with "CR"

**2. Accurate Filing Date**
```xml
<nc:CaseFiledDate>
  <nc:Date>2024-10-01</nc:Date>
</nc:CaseFiledDate>
```
- Must be actual filing date from CMS
- Format: YYYY-MM-DD
- Must not be altered or adjusted

**3. Appropriate Security Level**
```xml
<tyler:CaseSecurity>Public</tyler:CaseSecurity>
```
- Set security based on case status
- "Public" for standard cases
- "Sealed" for sealed cases
- "Confidential" for confidential cases

### What CMS Should NOT Do

**❌ Don't calculate publication delays:**
- re:Search handles delay calculation
- CMS just provides case type and filing date
- Delay rules may change; re:Search configuration updated centrally

**❌ Don't delay sending events:**
- Send NotifyCaseEvent immediately after filing
- Send CaseSecurity events immediately
- Don't wait for publication delay to expire

**❌ Don't alter filing dates:**
- Use actual filing date
- Don't add delay period to date
- Don't use future date

---

## What CMS Sends vs. What re:Search Enforces

This is an important distinction:

### CMS Sends (Always):
```
Case Type: CR (Criminal)
Filing Date: 2024-10-01
Case Security: Public
Event: CaseFiling
```

### re:Search Enforces (Automatically):
```
Receives case data
Applies 31-day delay rule (because case type = CR)
Calculates publication date: 2024-11-01
Hides from public until: 2024-11-01
Allows court staff access: Immediately
```

**Key Point:** CMS marks case as "Public" security, but re:Search still hides it from public for 31 days based on case type and market rules.

---

## Common Scenarios

### Scenario 1: Texas Criminal Case Filed

**Timeline:**

**October 1, 2024 - Filing Day**
```
CMS:
├─ Case filed by attorney
├─ Case type: CR (Criminal)
├─ Security: Public
├─ Sends NotifyCaseEvent
└─ Case processing complete

re:Search:
├─ Receives event
├─ Calls GetCase
├─ Identifies case type: CR
├─ Applies 31-day delay rule
├─ Publication date: November 1, 2024
└─ Indexes case (hidden from public)

Court Staff:
└─ Can see case immediately in re:Search

Public:
└─ Cannot see case (delay applies)
```

**October 15, 2024 - Mid-Delay Period**
```
Document filed to case:
├─ CMS sends DocumentFiling event
├─ re:Search updates case
└─ Still hidden from public (delay continues)

Court Staff:
└─ Can see new document immediately

Public:
└─ Still cannot see case
```

**November 1, 2024 - Delay Expires**
```
re:Search:
├─ Publication date reached
├─ Case becomes visible to public
└─ Appears in public search results

Public:
├─ Can now search for case
├─ Can view case details
└─ Can access documents (unless sealed)
```

---

### Scenario 2: Case Sealed by Court Order

**Timeline:**

**October 1, 2024 - Case Originally Public**
```
Case filed:
├─ Case type: CV (Civil)
├─ Security: Public
├─ No publication delay (civil case)
└─ Immediately visible to public
```

**October 15, 2024 - Seal Order Issued**
```
Court orders case sealed:

CMS:
├─ Updates case security to "Sealed"
├─ Sends CaseSecurity event IMMEDIATELY
└─ (Critical: security events never delayed)

re:Search:
├─ Receives security event
├─ Calls GetCase
├─ Updates case security to "Sealed"
├─ Starts 180-day seal countdown
└─ Case immediately hidden from public

Public:
├─ Case no longer appears in search
└─ Direct access denied ("case not found")

Court Staff:
└─ Continue to see case (authorized access)

Registered Users (Texas):
└─ Can still see case (role-based access)
```

**April 13, 2025 - 180 Days Later**
```
Seal period expires (if not extended):

re:Search:
├─ 180 days elapsed since seal
├─ Case automatically becomes public (if no extension)
└─ Appears in public search again

OR if court extends seal:
├─ CMS keeps security as "Sealed"
├─ No event sent (unchanged)
└─ Case remains sealed indefinitely
```

---

### Scenario 3: Sealed Case Approaching 180 Days

**Timeline:**

**March 1, 2025 - Case Sealed**
```
Case sealed on March 1, 2025
180-day expiration: August 28, 2025
```

**August 15, 2025 - 13 Days Before Expiration**
```
Court decides to extend seal:

CMS:
├─ No action needed
├─ Security remains "Sealed"
└─ No event sent

re:Search:
├─ Case continues as sealed
└─ No automatic unsealing occurs

Result:
└─ Case remains sealed past 180 days
```

**August 20, 2025 - 8 Days Before Expiration**
```
Court decides to unseal case:

CMS:
├─ Updates security to "Public"
├─ Sends CaseSecurity event immediately
└─ Case unsealed before 180-day expiration

re:Search:
├─ Receives security event
├─ Updates case to public
└─ Case appears in public search

Result:
└─ Case unsealed before 180-day period expires
```

---

### Scenario 4: Document Filed During Publication Delay

**Timeline:**

**October 1, 2024 - Criminal Case Filed**
```
Case filed:
├─ Case type: CR (Criminal)
├─ 31-day publication delay applies
├─ Publication date: November 1, 2024
└─ Hidden from public
```

**October 10, 2024 - Document Filed (Day 9 of delay)**
```
Attorney files motion:

CMS:
├─ Document filed to case
├─ Sends DocumentFiling event
└─ Document added to case

re:Search:
├─ Receives event
├─ Calls GetCase
├─ Updates case with new document
├─ Case still in delay period (22 days remaining)
└─ Document hidden from public (case not visible)

Court Staff:
└─ Can see new document immediately

Public:
└─ Cannot see case or document (delay continues)
```

**November 1, 2024 - Delay Expires**
```
Publication date reached:

re:Search:
├─ Case becomes visible
└─ All documents become visible (unless individually sealed)

Public:
├─ Can see case
└─ Can see all documents (including motion filed October 10)
```

**Key Point:** Documents filed during delay period become visible when delay expires, not when they were filed.

---

## How to Test Publication Delays

Testing publication delays requires special consideration because delays can be 31-180 days.

### Testing Approaches

**Approach 1: Wait for Real Delay (Not Practical)**
```
File case and wait 31 days
❌ Too slow for testing
❌ Blocks certification
❌ Not repeatable
```

**Approach 2: Use Past Filing Dates**
```
File case with filing date in the past:
└─ Filing date: 35 days ago

Result:
└─ Delay already expired when case filed
└─ Case immediately visible to public

✅ Practical for testing
✅ Immediate verification
✅ Can test multiple scenarios quickly
```

**Approach 3: Test in Non-Production (Recommended)**
```
Use test environment with:
├─ Shorter delay periods (e.g., 1 day instead of 31)
├─ Or use past filing dates
└─ Test all scenarios before production
```

### Test Scenarios to Verify

**1. Criminal Case Publication Delay**
```
Test: File criminal case (type CR)
Expected: Hidden from public for 31 days
Verify:
├─ Court staff can see immediately
├─ Public cannot see case
├─ Public cannot search for case
└─ After 31 days, case visible to public
```

**2. Civil Case (No Delay)**
```
Test: File civil case (type CV)
Expected: Immediately visible to public
Verify:
├─ Case appears in public search immediately
└─ Public can access case details immediately
```

**3. Sealed Case Period**
```
Test: Seal case with seal order
Expected: Hidden from public, visible to registered users
Verify:
├─ Public cannot see case
├─ Registered users can see case (Texas)
├─ Court staff can see case
└─ After 180 days, case becomes public (if not extended)
```

**4. Document During Delay**
```
Test: File document to case during delay period
Expected: Document hidden until delay expires
Verify:
├─ Court staff can see document immediately
├─ Public cannot see document during delay
└─ Public can see document after delay expires
```

---

## Verifying Role-Based Access

Test access with different user roles to verify delay enforcement.

### Test Matrix

| Scenario | Public User | Registered User | Court Staff | Party/Attorney |
|----------|-------------|----------------|-------------|----------------|
| **Criminal case filed (Day 1)** | ❌ No access | ❌ No access | ✅ Full access | ✅ Full access |
| **Criminal case (Day 31+)** | ✅ Full access | ✅ Full access | ✅ Full access | ✅ Full access |
| **Civil case filed** | ✅ Full access | ✅ Full access | ✅ Full access | ✅ Full access |
| **Sealed case** | ❌ No access | ✅ Full access (TX) | ✅ Full access | ✅ Full access |
| **Sealed document in public case** | ❌ No doc access | ❌ No doc access | ✅ Full access | ✅ Full access |

### Test Accounts Needed

**Public User Account:**
- No login required
- Browse anonymously
- Test public visibility

**Registered User Account:**
- Login with basic registration
- Test registered user access
- Verify role-based rules (Texas)

**Court Staff Account:**
- Login with staff credentials
- Test immediate access
- Verify no restrictions during delays

**Party/Attorney Account:**
- Login as case party or attorney
- Test access to own cases
- Verify delay doesn't affect own case access

---

## Common Timing Issues

### Issue 1: Criminal Case Visible Immediately

**Symptom:**
- Criminal case filed
- Should have 31-day delay
- Appears in public search immediately

**Possible Causes:**

**Cause 1: Incorrect Case Type**
```
Problem: Case type not recognized as criminal
Example: Case type = "CRIM" instead of "CR"
Solution: Use correct JCIT case type code (starts with "CR")
```

**Cause 2: Missing Filing Date**
```
Problem: CMS doesn't provide filing date
Result: re:Search can't calculate delay
Solution: Include CaseFiledDate in GetCase response
```

**Cause 3: Market Configuration**
```
Problem: Market not configured for delays
Result: No delays applied
Solution: Verify court configured for Texas JCIT rules
```

---

### Issue 2: Case Hidden Longer Than Expected

**Symptom:**
- Criminal case filed 31+ days ago
- Should be public now
- Still hidden from public search

**Possible Causes:**

**Cause 1: Case Security Set to Sealed**
```
Problem: CMS marked case as "Sealed" not "Public"
Result: Case remains hidden (seal rules apply)
Solution: Verify case security in GetCase response
```

**Cause 2: Incorrect Filing Date**
```
Problem: Filing date in future or incorrect
Example: Filed Oct 1, but date shows Nov 1
Result: Delay calculated from wrong date
Solution: Use actual filing date in GetCase
```

**Cause 3: No Reindexing After Date Change**
```
Problem: Filing date corrected but case not reindexed
Result: Old delay calculation still active
Solution: Send event to trigger reindexing
```

---

### Issue 3: Court Staff Cannot See Case

**Symptom:**
- Case filed
- Court staff should see immediately
- Cannot find case in re:Search

**This should never happen.** Court staff should always see cases immediately.

**Possible Causes:**

**Cause 1: Case Not Indexed Yet**
```
Problem: Event not sent or GetCase failed
Result: Case not in re:Search at all
Solution: Check event processing, verify GetCase response
```

**Cause 2: Permissions Issue**
```
Problem: User doesn't have court staff role
Result: Treated as public user
Solution: Verify user has correct role assignment
```

**Cause 3: Case Delete/Expunge**
```
Problem: Case deleted or expunged
Result: No longer in system
Solution: Verify case status in CMS
```

---

### Issue 4: Sealed Case Visible to Public

**Symptom:**
- Case sealed by court order
- Should be hidden from public
- Appears in public search

**This is a critical security issue.**

**Possible Causes:**

**Cause 1: Security Event Not Sent**
```
Problem: CMS sealed case but didn't send CaseSecurity event
Result: re:Search doesn't know case is sealed
Solution: Send CaseSecurity event immediately when sealing
```

**Cause 2: Wrong Security Value**
```
Problem: Case security set to "Public" not "Sealed"
Result: Case remains public
Solution: Use exact value "Sealed" in GetCase response
```

**Cause 3: Case-Sensitive Security Value**
```
Problem: CMS sends "SEALED" or "sealed"
Result: Value not recognized (must be "Sealed")
Solution: Use proper case: "Sealed"
```

---

## When Cases Don't Appear on Time

If cases aren't appearing when expected, follow this troubleshooting process:

### Step 1: Verify Event Sent

**Check:**
- Was NotifyCaseEvent sent?
- Was event received by re:Search?
- Did event processing succeed?

**How to Check:**
- Review CMS integration logs
- Check event queue for failures
- Use Integration Troubleshooting Tool

---

### Step 2: Verify GetCase Response

**Check:**
- Does GetCase return complete data?
- Is case type correct?
- Is filing date accurate?
- Is security level correct?

**How to Check:**
- Call GetCase manually with case ID
- Review response XML
- Verify all required fields present

---

### Step 3: Verify Case Type Mapping

**Check:**
- Is case type a valid JCIT code?
- Does case type match market standards?
- Is case type recognized by re:Search?

**How to Check:**
- Compare case type against JCIT standards
- Verify case type in re:Search configuration
- Test with known-working case type

---

### Step 4: Verify Delay Calculation

**Check:**
- What is the filing date?
- What delay should apply?
- What is the calculated publication date?
- Has publication date been reached?

**How to Calculate:**
```
Filing date: October 1, 2024
Case type: CR (Criminal)
Delay: 31 days
Publication date: November 1, 2024

Current date: October 15, 2024
Time until publication: 17 days
Expected: Case hidden from public
```

---

### Step 5: Verify Role-Based Access

**Check:**
- What role is user testing with?
- Should that role see the case?
- Is role assignment correct?

**Role Expectations:**
- Court staff: Always see cases immediately
- Parties/attorneys: See own cases immediately
- Public: See cases after delay expires
- Registered (Texas): See sealed cases, respect criminal delays

---

## Related Documentation

### For Technical Deep Dive
→ [Architecture Overview](./architecture-overview.md)  
→ [Security Logic](./security-logic.md)  
→ [Event Type Logic](./event-type-logic.md)

### For Market-Specific Rules
→ [Texas (JCIT) Standards](../market-standards/texas/README.md)  
→ [Illinois (AOIC) Standards](../market-standards/illinois/README.md)

### For Implementation Guidance
→ [Best Practices](../integration-guide/best-practices.md)  
→ [Common Mistakes](./common-mistakes.md)

### For Troubleshooting
→ [Real-World Examples](./real-world-examples.md)  
→ [Troubleshooting Guide](../troubleshooting/README.md)

---

**Last Updated:** January 2025