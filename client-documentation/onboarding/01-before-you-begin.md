# Environment Access – Client Onboarding

**Navigation:**  
[Home](../../../README.md) › [Client Documentation](../README.md) › [Client Onboarding](./README.md) › Environment Access

To begin integration and testing, CMS vendors and courts must obtain access to the appropriate **Tyler-managed environments**, including credentials, certificates, and secure endpoints required by their chosen integration mode.

This page outlines what access is required, how to request it, and which environments are used for each phase of onboarding.

---

# 🌐 Environment Types

re:Search integrations use three primary environments:

| Environment | Purpose | Typical Use Cases |
|-------------|---------|-------------------|
| **Dev / Vendor Sandbox** | Lightweight, isolated environment | Early development, schema validation, unit tests |
| **Stage** | Full E2E testing environment | End-to-end workflows, certification, regression testing |
| **Production** | Live statewide system | Real users, live court data, post-go-live operations |

---

# 🔐 Access Requirements by Integration Mode

Different integration modes require different credentials.

| Mode | Required Access | Notes |
|------|------------------|-------|
| **Batch Mode** | S3 or SFTP credentials | S3 preferred; per-court folder isolation |
| **ECF Mode** | mTLS certs + SOAP/WSDL endpoints | Requires CMS → EFM → re:Search connectivity |
| **Non-Integrated Mode** | BIS tooling access (internal) | Limited use; manual or assisted publishing |

If using **ECF Mode**, you must also reference EFM documentation located in a separate folder.

---

# 📝 How to Request Access

To request environment credentials, provide:

- Vendor name and primary technical contact  
- Integration mode (Batch, ECF, Non-Integrated)  
- Markets / courts participating in the onboarding  
- IP addresses for firewall allowlists  
- Desired timeline for access  

Submit requests through your Tyler onboarding contact. This may require an email to EFMinfo@tylertech.com

---

# 🧾 Access Components

## 1. S3 or SFTP Credentials (Batch Mode)
These are used only for JSONL snapshot delivery.

**Includes:**
- Access key or private key  
- Bucket or directory path  
- Folder naming conventions  
- Ingestion schedule and expectations  

---

## 2. X.509 & Tyler Root Certificates (ECF Mode)
Required for secure SOAP communication. Tyler issues two certificates:

- Tyler Root Certificate – This is used by your CMS to validate messages that we sign when the EFM sends responses back to you.
- Tyler-issued X.509 Client Certificate – This is the certificate you use to sign messages sent to the EFM. This is the same X.509 we provided for stage, and the same one that must be used in production.


**Includes:**
- Client certificate  
- Private key  
- Certificate installation instructions  
- SOAP endpoint URLs

---

## 3. Application-Level Accounts (Optional)
Some scenarios require user accounts for:

- Portal UI verification  
- Testing public vs. restricted access  
- Viewing case security outcomes  

These are provided only if needed for onboarding and testing.

---

## 🌐 Environment URLs

The following URLs are used for Stage and Production environments across all supported re:Search markets.

### Current Markets

| Market | Stage URL | Production URL |
|--------|-----------|----------------|
| **CA** | https://researchca-stage.tylerhost.net/ | https://researchca.tylerhost.net/ |
| **WA** | https://researchwa-stage.tylerhost.net/ | https://researchwa.tylerhost.net/ |
| **IL** | https://researchil-stage.tylerhost.net/ | https://researchil.tylerhost.net/ |
| **TX** | https://researchtx-stage.tylerhost.net/ | https://researchtx.tylerhost.net/ |
| **GA** | https://researchga-stage.tylerhost.net/ | https://researchga.tylerhost.net/ |
| **OK** | https://researchok-stage.tylerhost.net/ | https://researchok.tylerhost.net/ |
| **LA** | https://researchla-stage.tylerhost.net/ | https://researchla.tylerhost.net/ |
| **NM** | https://researchnm-stage.tylerhost.net/ | https://researchnm.tylerhost.net/ |
| **ME** | https://researchme-stage.tylerhost.net/ | https://researchme.tylerhost.net/ |

### Upcoming Markets

| Market | Stage URL | Production URL |
|--------|-----------|----------------|
| **RI** | https://researchri-stage.tylerhost.net/ | https://researchri.tylerhost.net/ |
| **PA** | https://researchpa-stage.tylerhost.net/ | https://researchpa.tylerhost.net/ |
| **VT** | https://researchvt-stage.tylerhost.net/ | https://researchvt.tylerhost.net/ |
| **TN** | https://researchtn-stage.tylerhost.net/ | https://researchtn.tylerhost.net/ |
| **ND** | https://researchnd-stage.tylerhost.net/ | https://researchnd.tylerhost.net/ |

---

# ➡ Next Steps

- **[CMS Preparation Checklist →](./03-cms-preparation-checklist.md)**  
- **[Client Onboarding Home →](./README.md)**  
- **[Client Documentation Home →](../README.md)**  
