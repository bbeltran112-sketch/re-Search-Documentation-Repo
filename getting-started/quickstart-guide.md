# Quick Start Guide

**Goal:** Understand re:Search integration in 10 minutes, then navigate to detailed guides.

## What is re:Search?

re:Search is Tyler's centralized case data repository. Courts and CMS systems send case data to re:Search, which makes it searchable for authorized users.

## Three Questions to Answer

### 1. How does my CMS connect to re:Search?

**Four integration modes:**
- **ECF Mode** - CMS integrated with EFM (most common)
- **CIP Mode** - Middleware handles the integration
- **Batch Mode** - Scheduled file uploads
- **Non-Integrated** - Manual portal entry

[Compare integration modes →](../integration-guide/integration-modes/README.md)

### 2. What APIs will I work with?

**Core APIs (ECF modes):**
- **NotifyCaseEvent** - Tell re:Search something changed
- **GetCase** - Return case data when requested
- **GetDocument** - Return PDFs when requested

**Batch Mode:**
- No APIs - just XML file exports

[See API Reference →](../api-reference/README.md)

### 3. What's market-specific vs universal?

**Universal (all courts):**
- How APIs work
- XML structure
- Security concepts

**Market-specific (varies by state):**
- Case types allowed
- Event types required
- Public access rules

[See Market Standards →](../market-standards/README.md)

## Next Steps

Choose your path:

**Just starting?**  
→ [Integration Guide](../integration-guide/README.md) - Start onboarding process

**Ready to code?**  
→ [API Reference](../api-reference/README.md) - Technical documentation

**Need market rules?**  
→ [Market Standards](../market-standards/README.md) - Texas, Illinois, etc.

**Have issues?**  
→ [Troubleshooting](../troubleshooting/README.md) - Common problems