# Incident Response Frameworks

**Reference documents:**
- **NIST SP 800-61r2** — [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)
- **SANS Institute** — [https://www.sans.org/white-papers/33901/](https://www.sans.org/white-papers/33901/)
- **ISO/IEC 27035** — [https://www.iso.org/standard/78973.html](https://www.iso.org/standard/78973.html)

---

## Why Frameworks Matter

No two incidents are identical, but the *process* of responding to them can and should be standardized. Incident Response (IR) frameworks provide:

- A **repeatable, structured methodology** so responders don't improvise under pressure
- A **common language** across teams (legal, IT, management, external parties)
- **Documentation and accountability** — essential for post-incident review, compliance, and legal proceedings
- **Coverage gaps reduction** — a checklist-style approach ensures no critical step is skipped during a stressful, fast-moving event

The three frameworks below are the most widely referenced in the industry. They differ in origin, depth, and naming conventions — but they all address the same fundamental lifecycle.

---

## 1. NIST SP 800-61r2 — Computer Security Incident Handling Guide

**Published by:** National Institute of Standards and Technology (U.S. government)
**Full title:** *Computer Security Incident Handling Guide, Special Publication 800-61 Revision 2*

NIST is arguably the most widely adopted IR framework, especially in U.S. federal agencies, regulated industries, and organizations that follow NIST's broader Cybersecurity Framework (CSF). It is freely available as a PDF and written to be practical and accessible.

### Structure — The 4 Phases

```
┌─────────────────┐
│   Preparation   │
└────────┬────────┘
         │
┌────────▼────────┐     ┌──────────────────────┐
│  Detection &    │────►│  Containment,        │
│  Analysis       │     │  Eradication &       │
└─────────────────┘     │  Recovery            │
                        └──────────┬───────────┘
                                   │
                        ┌──────────▼───────────┐
                        │  Post-Incident       │
                        │  Activity            │
                        └──────────────────────┘
```

| Phase | Key Activities |
|-------|---------------|
| **Preparation** | Build the IR team, define policies/procedures, set up tools, train staff, establish communication plans |
| **Detection & Analysis** | Monitor for indicators, classify the incident, determine scope and impact, prioritize response |
| **Containment, Eradication & Recovery** | Isolate affected systems, remove the threat, restore operations, validate systems are clean |
| **Post-Incident Activity** | Lessons learned meeting, update playbooks, report to stakeholders, legal/compliance follow-up |

### Key Characteristics
- Highly **detailed and prescriptive** — the PDF runs to ~79 pages with concrete guidance for each phase
- Covers **incident categorization** (DoS, malware, unauthorized access, etc.) with specific handling advice per type
- Includes guidance on **evidence handling**, chain of custody, and coordination with law enforcement
- Designed to be adapted to organizations of any size

---

## 2. SANS Institute — PICERL Framework

**Published by:** SANS Institute (leading cybersecurity training organization)
**Common name:** The **PICERL** model (from the initials of its six phases)

The SANS framework is widely taught in certifications (including GIAC) and is popular in the private sector and among incident responders who come from a hands-on/technical background. It breaks the response lifecycle into **six distinct phases**, giving slightly more granularity than NIST — particularly by separating Containment, Eradication, and Recovery into their own steps.

### Structure — The 6 Phases (PICERL)

```
Preparation → Identification → Containment → Eradication → Recovery → Lessons Learned
     P              I               C               E              R            L
```

| Phase | Key Activities |
|-------|---------------|
| **Preparation** | IR policies, team roles, toolkits, communication plans, tabletop exercises |
| **Identification** | Detect the incident, determine if it's a true positive, document initial findings, notify stakeholders |
| **Containment** | Short-term (isolate the system NOW) and long-term (rebuild/patch in a controlled way) containment strategies |
| **Eradication** | Remove malware, close the attack vector, patch vulnerabilities that were exploited |
| **Recovery** | Restore systems to production, monitor closely for re-infection, confirm normal operations |
| **Lessons Learned** | Post-mortem meeting (ideally within 2 weeks), update playbooks, report metrics, improve defenses |

### Key Characteristics
- The **most memorizable** structure — PICERL is easy to recall under pressure, which is why it's favored in operational IR teams
- Emphasizes the distinction between **short-term and long-term containment** — a nuance NIST groups together
- Practical, **practitioner-focused** writing style — oriented toward people actually doing the work, not compliance officers
- Heavily referenced in **SOC analyst certifications** and blue team training

---

## 3. ISO/IEC 27035 — Information Security Incident Management

**Published by:** International Organization for Standardization (ISO) / International Electrotechnical Commission (IEC)
**Full title:** *ISO/IEC 27035 — Information Technology: Information Security Incident Management*

ISO/IEC 27035 is part of the broader **ISO/IEC 27000 family** of information security management standards (which also includes ISO/IEC 27001 for ISMS). It is the most **internationally recognized** of the three frameworks and is often required for organizations pursuing ISO 27001 certification or operating in multinational/regulated environments.

### Structure — The 5 Phases

| Phase | Key Activities |
|-------|---------------|
| **Plan and Prepare** | Establish IR policy, define roles (ISIRT — Information Security Incident Response Team), set up reporting channels |
| **Detection and Reporting** | Identify events, assess whether they qualify as incidents, report through defined channels |
| **Assessment and Decision** | Triage the incident, determine severity, decide on response approach |
| **Responses** | Containment, forensic investigation, eradication, recovery — coordinated by the ISIRT |
| **Post-Incident Activity** | Lessons learned, evidence preservation, update policies, report to management and regulators |

### Key Characteristics
- **International standard** — recognized and required in many regulatory/compliance contexts (especially in Europe and Asia-Pacific)
- More **process and governance oriented** than NIST or SANS — focuses on organizational structures, roles, and accountability rather than technical steps
- Published in multiple parts: Part 1 (principles), Part 2 (guidelines for planning and preparation), Part 3 (guidelines for ICS/SCADA environments)
- Integrates naturally with **ISO 27001** — organizations already certified under 27001 often adopt 27035 as the natural IR complement

---

## 4. Side-by-Side Comparison

| Attribute | NIST SP 800-61r2 | SANS (PICERL) | ISO/IEC 27035 |
|-----------|-----------------|---------------|---------------|
| **Origin** | U.S. Government | Private (training org) | International standards body |
| **Phases** | 4 | 6 | 5 |
| **Focus** | Technical + procedural | Operational/practitioner | Governance + compliance |
| **Best suited for** | U.S. federal/regulated orgs | SOC teams, blue teamers | Multinational/ISO-certified orgs |
| **Availability** | Free PDF | Free white paper | Paid standard document |
| **Memorability** | Medium | High (PICERL acronym) | Low (more reference than operational) |
| **Technical depth** | High | High | Medium |
| **Compliance use** | FISMA, NIST CSF | GIAC certifications | ISO 27001 ecosystems |

---

## 5. Which One Should You Know?

In practice, most organizations **blend elements from all three** rather than following any single framework rigidly. However:

- For **certifications** (Security+, CEH, GIAC): PICERL (SANS) is the most commonly tested — memorize the six phases in order
- For **U.S. government / federal work**: NIST SP 800-61r2 is the mandatory reference
- For **enterprise compliance or ISO 27001 contexts**: ISO/IEC 27035 is the relevant standard

