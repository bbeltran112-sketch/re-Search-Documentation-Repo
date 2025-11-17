# Support Playbook Overview

**Navigation:**  
[Home](../../README.md) › [Technical Documentation](../README.md) › [Support Playbook](./index.md) › Overview

The Support Playbook defines how re:SearchTX interprets case data, enforces JCIT standards, processes EventTypes, and updates indexed content. It serves as the foundation for support, onboarding, and vendor troubleshooting.

---

## Purpose

This overview provides:

- A high-level understanding of re:Search’s processing model  
- Core concepts used throughout the playbook  
- Definitions of visibility, indexing, and CMS interactions  

---

## Core Concepts

### Case as the System of Record
The CMS is authoritative for:

- Case metadata  
- Filings  
- Parties & attorneys  
- Security  
- Docket history  

### EventType-Driven Processing
EventTypes determine which sections of the GetCaseResponse are evaluated.

### JCIT Visibility Rules
Registered users see different content based on:

- Role  
- Case type  
- CaseSecurity  
- DocumentSecurity  

### Indexing Triggers
Reindex operations occur when:

- NotifyCaseEvent is received  
- NotifyDocketingComplete is processed  
- Security changes  
- Correction of stale or invalid vendor XML  

---

## Related Topics

- [JCIT Roles](./jcit-roles.md)
- [EventType Logic](./eventtype-logic.md)
- [Security Logic](./security-logic.md)
- [Troubleshooting](./troubleshooting.md)
