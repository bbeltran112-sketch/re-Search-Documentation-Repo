# 2. JCIT Roles

The JCIT Technology Standards v10.0 define five user roles. Visibility rules in re:SearchTX are governed by these roles.

Understanding the role of the user experiencing the issue is the first step in troubleshooting.

## 2.1 Role Summary

Role 1A–1C: Judges  
- Full access to all records  
- Can view sealed and confidential items  
- Case-level and document-level security enforced only for auditing

Role 2: Attorneys of Record  
- Access to cases in which they are an attorney of record  
- Cannot see sealed documents unless explicitly permitted

Role 3A–3B: Clerks  
- Full access to their own county  
- Some restrictions on sealed cases depending on policy

Role 4: Non-Public Agency Users  
- Includes law enforcement and other government agencies  
- Access defined by statute, not public rules

Role 5: Registered Public Users  
- The most restrictive  
- Can only see:
  - Public civil and probate cases
  - Limited family case index
  - Limited criminal documents
  - No sealed or confidential materials

## 2.2 Why Role 5 Creates Most Issues

Role 5 cannot see:
- Confidential cases
- Sealed cases
- Any case containing a confidential/sealed document
- Many family case types
- Most criminal documents
- Cases under 31-day/180-day delays

Most visibility-related tickets originate here.

Reference: JCIT Technology Standards v10.0, Section 5.1.
