# JCIT Public Access & Registered User Matrix Requirements  
Version 10.0 – July 2025

Public access to case information in re:SearchTX is governed by the Registered User Matrix.  
Visibility depends on:

- Case Category  
- Case Type  
- Protected Style requirements  
- Document Security  
- Case Security  
- Criminal document type rules  
- County population (for family delay rules)  

## CMS Impact on Public Access

### 1. Case Type determines public accessibility  
If the CMS sends an invalid case type, the case is hidden from public users.

### 2. Protected Style Rules (Family Cases)
Certain Family case types must automatically convert the style to:

```
Protected vs. Protected
```

This is triggered by JCIT case type, not by CMS logic.

### 3. Criminal Documents  
Only these document types can be visible to public users:

- Information  
- Indictment  
- Sentence  
- Judgment  
- Order of Dismissal  

A CMS MUST classify documents correctly.

### 4. Delay Rules (31-Day Delay for Family Cases)
If the county exceeds a population threshold, some Family cases have delayed visibility.

CMS must still send the filed date accurately; re:Search enforces the delay.

### 5. Incorrect Case Types = Restricted Access  
Misclassified case types cause:

- Cases not appearing to the public  
- Missing documents  
- Missing styles  
- Incorrect grouping  

## CMS Must Ensure All Values Conform Before Transmission  
The public matrix cannot interpret county-specific or custom CMS values.

If the CMS uses incorrect values, the public access decision tree cannot execute correctly, and re:Search will default to restricted behavior.
