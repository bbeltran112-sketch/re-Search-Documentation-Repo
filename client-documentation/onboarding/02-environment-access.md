# Environment Access – Client Onboarding

**Navigation:**  
[Home](../../../README.md) › [Client Documentation](../README.md) › [Client Onboarding](./README.md) › Environment Access

This page provides the required URLs, credentials, and environment details needed to begin integration with re:Search.

---

# 🌐 Overview

All CMS vendors must ensure:

- Access to **Stage** and **Production** environments  
- Credentials or certificates depending on the integration mode  
- Ability to reach all required endpoints from court or vendor networks  
- Optional UI access for confirming visibility and search behavior  

This section serves as the central reference for environment URLs across all re:Search markets.

---

# 📡 Environment URL Patterns

## Stage Environment  
Format:  
`https://researchXX-stage.tylerhost.net/`  
Replace `XX` with the market abbreviation.

## Production Environment  
Format:  
`https://researchXX.tylerhost.net/`  

---

# 🗺 Market URL Reference

## Current Markets

| Market | Stage URL | Production URL |
|--------|-----------|----------------|
| **TX** | https://researchtx-stage.tylerhost.net/ | https://researchtx.tylerhost.net/ |
| **CA** | https://researchca-stage.tylerhost.net/ | https://researchca.tylerhost.net/ |
| **WA** | https://researchwa-stage.tylerhost.net/ | https://researchwa.tylerhost.net/ |
| **IL** | https://researchil-stage.tylerhost.net/ | https://researchil.tylerhost.net/ |
| **GA** | https://researchga-stage.tylerhost.net/ | https://researchga.tylerhost.net/ |
| **OK** | https://researchok-stage.tylerhost.net/ | https://researchok.tylerhost.net/ |
| **LA** | https://researchla-stage.tylerhost.net/ | https://researchla.tylerhost.net/ |
| **NM** | https://researchnm-stage.tylerhost.net/ | https://researchnm.tylerhost.net/ |
| **ME** | https://researchme-stage.tylerhost.net/ | https://researchme.tylerhost.net/ |

---

## Upcoming Markets

| Market | Stage URL | Production URL |
|--------|-----------|----------------|
| **RI** | https://researchri-stage.tylerhost.net/ | https://researchri.tylerhost.net/ |
| **PA** | https://researchpa-stage.tylerhost.net/ | https://researchpa.tylerhost.net/ |
| **VT** | https://researchvt-stage.tylerhost.net/ | https://researchvt.tylerhost.net/ |
| **TN** | https://researchtn-stage.tylerhost.net/ | https://researchtn.tylerhost.net/ |
| **ND** | https://researchnd-stage.tylerhost.net/ | https://researchnd.tylerhost.net/ |

---

# 🔐 Authentication Requirements

## Batch Mode (S3 or SFTP)
- S3 Access Key + Secret Key **or** SFTP SSH Key  
- Vendor receives isolated folder paths per court/market  
- All uploads must follow the Batch Mode manifest structure  

## ECF Mode (SOAP + mTLS)
- Client certificate (issued by Tyler)  
- Private key (vendor managed)  
- Trusted CA bundle  
- All SOAP calls require mutual TLS authentication  

---

# 👤 UI Access for Testing (Optional)

Vendors may receive user accounts for:

- Basic Role 5 visibility testing  
- Verifying confidential vs public case behavior  
- Checking document padlock logic  

Access may vary by market rules.

---

# 🧪 Environment Readiness Checklist

Before onboarding begins, confirm:

- Stage URL accessible from vendor network  
- Credentials or certificates validated  
- S3/SFTP upload connectivity confirmed (Batch Mode)  
- GetCase/GetDocument working (ECF Mode)  
- UI access (if needed) created and verified  

---

# 🔗 Related Documentation

- [Client Onboarding Home](./README.md)  
- [Before You Begin](./01-before-you-begin.md)  
- [Technical Onboarding](./05-technical-onboarding.md)  
- [Integration Modes Overview](../integration-modes/README.md)  
- [Support Playbook](../../technical-documentation/support-playbook/README.md)

---

# ➡️ Next Steps

Proceed to:  
**[Technical Onboarding →](./05-technical-onboarding.md)**

---

# ⬅️ Back to

**[Client Onboarding Home](./README.md)**  
