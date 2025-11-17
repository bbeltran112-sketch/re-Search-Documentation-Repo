# Integration Mode Selection Decision Tree

Use this guide to determine which integration mode applies to your CMS or court.

---

## Decision Tree

**1. Can the CMS export JSONL and deliver via S3/SFTP?**  
→ **Yes** → Use **Batch Mode**  
→ **No** → Continue

**2. Is the CMS already ECF-enabled?**  
→ **Yes** → Use **ECF Mode**  
→ **No** → Continue

**3. Is the court running Enterprise Justice?**  
→ **Yes** → Use **CIP Mode (until migrated to Batch)**  
→ **No** → Continue

**4. Is the court unable to support integration?**  
→ **Yes** → Use **Non-Integrated Mode**  

---

## Related Documentation

- [Integration Modes Overview](./README.md)
- [Batch Mode Overview](./batch-mode-overview.md)
- [ECF Mode Overview](./ecf-mode-overview.md)

**Back to:** [Integration Modes](./README.md)
