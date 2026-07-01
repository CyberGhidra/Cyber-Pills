# Security Goals of Cybersecurity — CompTIA Security+

## 1. The Objectives of Information Security

The foundation of information security is built around three core objectives, universally known as the **CIA Triad**. Every security control, policy, or technology ultimately exists to support one or more of these goals.

### 1.1 The CIA Triad

| Principle | Definition | Goal |
|---|---|---|
| **Confidentiality** | Ensuring that information is accessible only to those authorized to view it | Prevent unauthorized disclosure |
| **Integrity** | Ensuring that data is accurate, consistent, and has not been tampered with | Prevent unauthorized alteration |
| **Availability** | Ensuring that systems and data are accessible to authorized users when needed | Prevent denial of service/access |

- **Confidentiality** is enforced through mechanisms such as encryption, access control lists (ACLs), and authentication.
- **Integrity** is enforced through hashing algorithms (e.g., SHA-256), digital signatures, and checksums.
- **Availability** is enforced through redundancy, fault tolerance, backups, and load balancing.

> 💡 **Exam tip:** The CIA Triad is the most frequently referenced model in Security+. Every scenario question about "what security goal was violated?" maps back to one (or more) of these three pillars.

---

## 2. The Risks of Data Breaches

While the CIA Triad describes what security *aims to protect*, the **DAD Triad** describes what happens when security *fails*. It represents the inverse/opposite of the CIA Triad.

### 2.1 The DAD Triad

| Principle | Definition | Corresponding CIA Failure |
|---|---|---|
| **Disclosure** | Sensitive information is exposed to unauthorized parties | Loss of Confidentiality |
| **Alteration** | Data is modified or corrupted, intentionally or not | Loss of Integrity |
| **Denial** | Legitimate users are unable to access data or systems | Loss of Availability |

The DAD Triad is a useful mental model: whenever you analyze an incident, ask "was something **disclosed**, **altered**, or **denied**?" — this immediately tells you which CIA principle was violated.

### 2.2 Impact of a Data Breach

A breach rarely affects just one area of a business. CompTIA Security+ identifies five main categories of impact:

1. **Financial** — direct costs (fines, legal fees, incident response, remediation), lost revenue, ransom payments, increased insurance premiums.
2. **Reputational** — loss of customer trust, brand damage, negative media coverage, loss of business partnerships.
3. **Strategic** — loss of competitive advantage, theft of intellectual property, exposure of business plans or trade secrets.
4. **Operational** — disruption of business processes, downtime, loss of productivity, degraded service delivery.
5. **Compliance** (regulatory) — violations of laws and regulations (e.g., GDPR, HIPAA, PCI-DSS) resulting in fines, audits, or legal action.

> 🧩 Note: These impact categories often cascade — a breach that starts as an operational disruption can quickly become a financial and reputational crisis, and may also trigger compliance violations.

### 2.3 Gap Analysis

**Gap analysis** is the process of comparing an organization's current security posture ("as-is" state) against a desired target state (a framework, standard, or best-practice baseline, such as NIST CSF or ISO 27001).

- It identifies **gaps** — missing controls, outdated policies, or unaddressed risks.
- The output is typically a **remediation roadmap** prioritizing which gaps to close first, based on risk level, cost, and effort.
- Gap analysis is often a precursor to **risk assessments** and **audits**, and is commonly used when preparing for compliance certifications.

---

## 3. Security Control Categories

Security controls are grouped into **categories** based on *how* they are implemented, and **types** based on *what function* they perform. CompTIA Security+ tests both dimensions.

### 3.1 Categories (the "how")

| Category | Description | Examples |
|---|---|---|
| **Technical** | Implemented via technology/systems | Firewalls, encryption, IDS/IPS, antivirus, MFA |
| **Managerial** | Administrative/governance-driven, focused on policy and risk oversight | Security policies, risk assessments, security awareness programs |
| **Operational** | Implemented and executed by people in day-to-day processes | Security guards, incident response procedures, backup rotations, user training |
| **Physical** | Controls that protect the physical environment | Locks, fences, badges, security cameras, mantraps |

> 💡 **Exam tip:** "Managerial" and "Operational" are sometimes confused. Managerial = policy/oversight (the *why* and *what should happen*); Operational = the people-driven execution of that policy day to day (the *doing*).

---

## 4. Security Control Types

While categories describe *how* a control is implemented, **types** describe the *purpose/function* the control serves in the security lifecycle.

| Type | Function | Example |
|---|---|---|
| **Preventive** | Stops an incident before it happens | Firewall rules, MFA, security awareness training |
| **Deterrent** | Discourages an attacker from attempting an action | Warning signs, visible security cameras, login banners |
| **Detective** | Identifies that an incident has occurred or is occurring | IDS, SIEM alerts, log monitoring |
| **Corrective** | Reduces the impact/restores systems after an incident | Backups/restore, patching, quarantine of infected hosts |
| **Directive** | Directs or mandates a particular action/behavior via policy | Security policies, standard operating procedures (SOPs), compliance mandates |
| **Compensating** | An alternative control used when the primary control isn't feasible | Using additional monitoring when patching isn't immediately possible |

> 🧩 A single technology can serve multiple purposes. For example, a **camera** is both **deterrent** (visible presence discourages attackers) and **detective** (records footage for later review) — but it is *not* preventive, since it doesn't physically stop an intrusion.

---

## 5. Data Protection

Protecting data requires understanding *where* the data currently resides in its lifecycle, since different states require different protective mechanisms.

### 5.1 Data States

| State | Description | Typical Protection |
|---|---|---|
| **Data at Rest** | Data stored on disk, in databases, backups, etc. | Full-disk encryption, database encryption, access controls |
| **Data in Transit** | Data actively moving across a network | TLS/SSL, VPNs, IPsec |
| **Data in Use** | Data actively being processed in memory/by an application | Secure enclaves, memory encryption, homomorphic encryption |

### 5.2 Encryption

Encryption is the primary mechanism protecting confidentiality across all data states. It converts plaintext into ciphertext using an algorithm and a key, so that only authorized parties possessing the correct key can decrypt and read it. Encryption can be applied at multiple levels: full-disk, file-level, database-level (transparent data encryption), or communications-level (TLS).

### 5.3 Data Loss Prevention (DLP)

**DLP** systems monitor, detect, and block unauthorized transmission or exposure of sensitive data (e.g., PII, credit card numbers, intellectual property).

**Deployment models:**

| Model | Description |
|---|---|
| **Agent-based DLP** | Software agent installed directly on endpoints (laptops, servers), providing visibility and control even when the device is off the corporate network |
| **Agentless DLP** | Operates at the network or cloud level (e.g., inline network appliances, cloud API integrations) without requiring installation on each endpoint |

**Detection techniques:**

| Technique | Description |
|---|---|
| **Pattern matching** | Uses regular expressions/signatures to detect structured sensitive data (e.g., credit card number formats, SSNs) |
| **Watermarking** (also called *tagging*/*fingerprinting*) | Embeds a hidden marker or fingerprint into a document, allowing the DLP system to recognize and track that specific file even after modification |

### 5.4 Data Minimization

Data minimization reduces the exposure risk of sensitive data by limiting how much identifiable/sensitive data is stored, processed, or exposed.

| Technique | Description | Reversible? |
|---|---|---|
| **Hashing** | Converts data into a fixed-length value using a one-way mathematical function | No — cannot be reversed to obtain the original value |
| **Tokenization** | Replaces sensitive data with a non-sensitive placeholder ("token") that maps back to the original value in a secure lookup table (vault) | Yes — via the token vault |
| **Masking** | Obscures part of the data while preserving its format (e.g., showing only the last 4 digits of a credit card number) | Partial — original may still be stored elsewhere |

> 💡 **Exam tip:** Hashing is used for *integrity verification* (e.g., password storage, file checksums) and is one-way. Tokenization is used to protect data *while retaining usability* (e.g., payment processing) and is reversible via the vault.

### 5.5 Access Restrictions

Restricting *who* and *from where* data can be accessed is another key layer of data protection.

| Restriction Type | Description |
|---|---|
| **Geographic restrictions** | Limits data access based on the geographic location of the user/request (e.g., geofencing, geoblocking specific countries), often used for regulatory compliance (data residency/sovereignty) |
| **Permission-based restrictions** | Access granted based on user roles, groups, or explicit authorization — following principles like **least privilege** and **role-based access control (RBAC)** |

### 5.6 Segmentation and Isolation

- **Segmentation** divides a network or system into smaller, controlled zones (e.g., VLANs, subnets) to limit the blast radius of a breach and enforce granular access policies between zones.
- **Isolation** goes a step further by completely separating a system or data set from the rest of the environment (e.g., air-gapped systems, sandboxing, isolated VLANs for critical infrastructure/OT systems), preventing any lateral movement.

Both are foundational to a **Zero Trust** architecture and to limiting the impact of a compromised system.

---

## Summary

| Concept Group | Key Idea |
|---|---|
| CIA Triad | The three goals security aims to protect: Confidentiality, Integrity, Availability |
| DAD Triad | The three ways security fails: Disclosure, Alteration, Denial |
| Breach Impact | Financial, Reputational, Strategic, Operational, Compliance |
| Gap Analysis | Comparing current vs. desired security posture to find and prioritize gaps |
| Control Categories | Technical, Managerial, Operational, Physical (the *how*) |
| Control Types | Preventive, Deterrent, Detective, Corrective, Directive, Compensating (the *why/function*) |
| Data States | At rest, in transit, in use |
| Data Protection Tools | Encryption, DLP (agent-based/agentless, pattern matching/watermarking) |
| Data Minimization | Hashing (one-way), Tokenization (reversible), Masking (partial) |
| Access Restrictions | Geographic, permission-based (RBAC, least privilege) |
| Network Protection | Segmentation, isolation |

---
