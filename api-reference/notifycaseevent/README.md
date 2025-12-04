# NotifyCaseEvent API

**Navigation:** [Home](../../../README.md) › [Technical Documentation](../../README.md) › [API Reference](../README.md) › NotifyCaseEvent

## Quick Navigation
- [Request & Response Specification](./request-response.md) - Technical details and examples
- [Event Types Guide](./event-types.md) - All supported event types
- [Workflows](./workflows.md) - When and how to send events
- [XML Examples](./examples/) - Sample request/response files

---

## What is NotifyCaseEvent?

NotifyCaseEvent is the **event notification API** that your CMS calls to tell re:Search when something has changed in a case. It's a lightweight message that triggers re:Search to call GetCase and retrieve the updated data.

**Think of it as:** Your CMS saying "Hey re:Search, case CV-123 just changed - go look at it!"

---

## At a Glance

| Property | Value |
|---------|--------|
| **Direction** | CMS → re:Search |
| **Protocol** | SOAP 1.2 over HTTPS |
| **Security** | Mutual TLS (mTLS) |
| **Standard** | OASIS ECF 4.x |
| **Message Type** | NotifyCaseEventRequest / NotifyCaseEventResponse |
| **Required?** | Yes (all ECF/CIP integrations) |
| **Triggers** | GetCase call from re:Search |

---

## When to Send NotifyCaseEvent

Send NotifyCaseEvent **immediately after** any of these changes:

### Critical (Send Immediately - Never Delay)
- ⚠️ **Case sealed/unsealed** (`CaseSecurity`)
- ⚠️ **Document sealed/restricted** (`DocumentSecurity`)

### Common (Send After Commit)
- **New filing added** (`CaseFiling`)
- **Party added/changed** (`CaseParty`)
- **Attorney added/changed** (`CasePartyAttorney`)
- **Disposition entered** (`CaseDisposition`)
- **Judge/division changed** (`CaseReassigned`)
- **Case status changed** (`CaseStatus`)
- **New case created** (`CaseInitiated`)

[Complete event type list →](./event-types.md)

---

## How It Works

```
Step 1: Change happens in CMS
        ↓
Step 2: CMS sends NotifyCaseEvent (small message with case ID + event type)
        ↓
Step 3: re:Search receives event
        ↓
Step 4: re:Search calls GetCase to retrieve current data
        ↓
Step 5: Case updated in re:Search
```

**Key concept:** NotifyCaseEvent doesn't contain the actual case data - just enough to tell re:Search what changed and where to look.

---

## Quick Example

### Request (What You Send)

```xml
<NotifyCaseEvent>
  <NotifyCaseEventMessage>
    <!-- Which court? -->
    <CaseCourt>
      <OrganizationIdentification>
        <IdentificationID>Hopkins:cc</IdentificationID>
      </OrganizationIdentification>
    </CaseCourt>
    
    <!-- Which case? -->
    <CaseTrackingID>CV-2024-00123</CaseTrackingID>
    
    <!-- Why are you notifying? -->
    <NotificationReason>Party added to case</NotificationReason>
    
    <!-- What changed? -->
    <Event>
      <ID>EVT-2024-0001</ID>
      <EventType>CaseParty</EventType>
    </Event>
  </NotifyCaseEventMessage>
</NotifyCaseEvent>
```

### Response (What You Get Back)

```xml
<NotifyCaseEventResponse>
  <Success>true</Success>
</NotifyCaseEventResponse>
```

**That's it!** Simple acknowledgment. re:Search will call GetCase separately.

[See complete examples →](./request-response.md#complete-examples)

---

## Key Requirements

### Must Include (Required)
- ✅ Court identification (`OrganizationIdentification/IdentificationID`)
- ✅ Case tracking ID (`CaseTrackingID`)
- ✅ Notification reason (`NotificationReason`)
- ✅ Event ID (`Event/ID`)
- ✅ Event type (`Event/EventType`)

### Timing Rules (Critical)
- ✅ Send **after database commit** (not before)
- ✅ Security events sent **immediately** (never batch/queue)
- ✅ Other events sent **promptly** (within seconds)
- ✅ Use async messaging for reliability (recommended)

### What NOT to Include
- ❌ Case data (use GetCase response for that)
- ❌ Document binaries
- ❌ Complete party lists
- ❌ Large payloads

**Keep it small and fast!**

---

## EventType Determines GetCase Processing

**This is critical:** The EventType you send determines which blocks re:Search processes in the GetCase response.

| EventType You Send | What GetCase Processes |
|-------------------|------------------------|
| `CaseFiling` | Filing events, documents |
| `CaseParty` | Parties and attorneys |
| `CaseSecurity` | Security settings only |
| `CaseDisposition` | Disposition data |
| `CaseInitiated` | **Everything** (complete case) |

**Example:**
```
You send: EventType="CaseParty"
       ↓
re:Search processes: Only party/attorney blocks in GetCase response
re:Search ignores: Filings, documents, security, etc.
```

[Complete EventType mapping →](./event-types.md)

---

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **Event not received** | Network/firewall issue | Check connectivity, firewall rules |
| **GetCase not called** | Invalid EventType or CaseTrackingID | Verify EventType is supported, case exists |
| **Changes not appearing** | Wrong EventType used | Use correct EventType or CaseInitiated |
| **Security delay** | Event queued instead of immediate | Send CaseSecurity synchronously |
| **Duplicate events** | No idempotency check | Add event deduplication logic |

[Complete troubleshooting →](./workflows.md#troubleshooting)

---

## Event Sending Patterns

### Pattern 1: Single Change (Most Common)

```
Change: Party added
Action: Send one NotifyCaseEvent with EventType=CaseParty
Result: GetCase called once, party updated
```

### Pattern 2: Multiple Related Changes

```
Changes: Party added + Attorney assigned
Options:
  A) Send 2 events: CaseParty + CasePartyAttorney
  B) Send 1 event: CaseInitiated (processes everything)

Recommendation: Use CaseInitiated for bulk changes
```

### Pattern 3: Security Change (Time-Critical)

```
Change: Case sealed
Action: Send CaseSecurity event IMMEDIATELY (synchronous)
Result: GetCase called with high priority, case hidden within seconds
```

[More patterns →](./workflows.md#common-patterns)

---

## Performance Guidelines

### Message Size
- **Target:** < 5 KB per message
- **Maximum:** < 50 KB

### Frequency
- **Typical:** 10-50 events per minute (average court)
- **Peak:** 100-200 events per minute (large court)
- **Burst:** System handles spikes well

### Timing
- **Event delivery:** < 1 second
- **GetCase triggered:** < 5 seconds after event
- **Total latency:** < 30 seconds (change to visible in UI)

---

## Quick Start Checklist

### For Developers Implementing NotifyCaseEvent

- [ ] Review [EventTypes Guide](./event-types.md) for all supported types
- [ ] Understand [EventType mapping](./event-types.md#eventtype-mapping)
- [ ] Review [request/response specification](./request-response.md)
- [ ] Implement event generation for all change scenarios
- [ ] Add hooks to CMS change operations:
  - [ ] Party add/edit/delete
  - [ ] Filing creation
  - [ ] Security changes (critical!)
  - [ ] Disposition entry
  - [ ] Judge/division reassignment
- [ ] Implement async messaging (recommended)
- [ ] Add retry logic with exponential backoff
- [ ] Add comprehensive logging
- [ ] Test all EventTypes
- [ ] Verify security events sent immediately

### For Operations Teams

- [ ] Monitor event delivery success rate
- [ ] Alert on delivery failures
- [ ] Monitor event-to-GetCase latency
- [ ] Review error logs regularly
- [ ] Ensure mTLS certificates valid

---

## Related APIs

NotifyCaseEvent works in coordination with:

**Triggers GetCase:**
- CMS sends NotifyCaseEvent → re:Search calls GetCase

**Workflow:**
```
NotifyCaseEvent (CMS → re:Search) → GetCase (re:Search → CMS)
```

**Related:**
- [GetCase API](../getcase/) - Data retrieval triggered by events
- [NotifyDocketingComplete](../notifydocketingcomplete/) - Docketing confirmation
- [RecordFiling](../recordfiling/) - Filing submission (via EFM)

---

## Authentication

NotifyCaseEvent requires mutual TLS (mTLS) authentication.

**Certificate requirements:**
- Valid mTLS certificate issued by Tyler
- Certificate must not be expired
- Proper certificate chain validation
- Secure certificate storage

[Authentication details →](../common-headers-and-auth.md)

---

## Next Steps

### New to NotifyCaseEvent?
1. Read [Request & Response Specification](./request-response.md) to understand the message format
2. Review [Event Types Guide](./event-types.md) to learn all supported event types
3. Study [Workflows](./workflows.md) to see NotifyCaseEvent in context

### Implementing NotifyCaseEvent?
1. Start with [Request & Response Specification](./request-response.md)
2. Review all [Event Types](./event-types.md) and when to use each
3. Download [example XML files](./examples/)
4. Test with simple events first
5. Use [Workflows guide](./workflows.md) for integration patterns

### Troubleshooting Issues?
1. Check [common issues](#common-issues) above
2. Review [Workflows troubleshooting](./workflows.md#troubleshooting)
3. Verify [EventType mapping](./event-types.md#eventtype-mapping)
4. Contact your Tyler Technical Project Manager

---

## Additional Resources

- [ECF Mode Implementation](../../integration-modes/ecf-mode/) - Overall ECF integration
- [GetCase API](../getcase/) - What happens after NotifyCaseEvent
- [Event Type Logic](../../support-playbook/event-type-logic.md) - Decision guide
- [Troubleshooting Guide](../../support-playbook/troubleshooting.md) - Common problems

---

**Last Updated:** December 2025