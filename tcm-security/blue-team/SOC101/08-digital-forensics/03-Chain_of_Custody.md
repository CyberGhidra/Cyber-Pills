# Chain of Custody

---

## Why Chain of Custody Matters

In digital forensics, the technical quality of an investigation is only half the story. The other half is being able to prove, to a court, a regulator, or an auditor, that the evidence presented is the same evidence that was collected — unaltered, untampered, and handled by known, authorized individuals at every step.

A chain of custody failure does not mean the investigation was wrong. It means the investigation **cannot be proven to be right** in a formal proceeding. Defense attorneys routinely challenge evidence on chain of custody grounds — not because they believe the evidence is fabricated, but because a gap in documentation creates reasonable doubt. A single undocumented handoff, a mislabeled evidence bag, or an unrecorded access to a forensic image can be enough to render otherwise solid evidence inadmissible.

> **Core principle:** the chain of custody is the difference between evidence that tells a story and evidence that can be used. Both the technical work and the documentation must be rigorous — neither alone is sufficient.

---

## 1. Forensic Data Integrity

Before evidence can be collected, a fundamental problem must be solved: how do you examine a storage device without changing it? Simply connecting a drive to a computer and reading it can alter timestamps, update access logs, and modify metadata — potentially corrupting the very evidence being investigated.

### Write Blockers

A **write blocker** is a device or software layer that sits between the forensic investigator's workstation and the evidence storage device, intercepting any write commands and preventing them from reaching the evidence. The result is a one-way channel: the investigator can read everything on the device, but nothing can be written back to it.

**Hardware write blockers:**
- Physical devices that connect between the evidence drive and the forensic workstation (e.g., Tableau, WiebeTech)
- Preferred in legal contexts because they are a visible, tangible layer of protection that is easy to explain to a court
- Work at the hardware level — no software vulnerability or misconfiguration can bypass them
- Support a range of interfaces: SATA, NVMe, USB, SAS

**Software write blockers:**
- OS-level or driver-level tools that achieve the same result through software (e.g., Windows registry write-blocking, Linux `mount -o ro`)
- More convenient and lower cost, but less authoritative in legal proceedings — a defense argument that software can have bugs or be misconfigured is harder to dismiss than the same argument against a dedicated hardware device

> **Rule:** whenever possible, use a hardware write blocker. In criminal or highly sensitive corporate investigations, this is not optional.

### Preserving Device Integrity

Write blockers address the risk of accidental writes during examination. Broader device integrity measures address the risk of damage, degradation, or unintentional alteration before examination begins:

- **Faraday bags** for mobile devices — block all wireless signals (cellular, Wi-Fi, Bluetooth, GPS), preventing remote wipe commands, incoming messages, and location updates from altering the device's state before forensic acquisition
- **Anti-static packaging** for hard drives and circuit boards — prevents electrostatic discharge damage during transport
- **Temperature and humidity controls** for storage — storage media can be physically damaged by extreme conditions, destroying evidence irreversibly

### Analysis on Forensic Images, Not Originals

All examination and analysis work must be conducted on a **forensic image** — a bit-for-bit copy of the original storage device — never on the original evidence itself.

The forensic image:
- Is created using tools that verify integrity (e.g., `dd`, FTK Imager, `dcfldd`)
- Is verified by computing a hash (SHA-256) of both the original device and the resulting image — the hashes must match exactly, proving the image is a perfect copy
- Can be duplicated as many times as needed without degrading the original
- If accidentally modified during analysis, another copy can be made from the original
- The original device is sealed, stored, and never touched again unless specifically required

```
Original Device  ──(write blocker)──►  Forensic Image  ──►  Analysis Copy
     [sealed]         [verified]           [hash check]      [working copy]
```

This three-layer separation ensures the original evidence is never at risk from the examination process.

---

## 2. Evidence Bags

Once a device or physical artifact has been collected, it must be secured in a way that protects it physically and documents its status at the time of collection.

### Purpose
Evidence bags **contain and protect** collected items — both from physical damage during transport and storage, and from contamination or tampering that could compromise their admissibility.

### Tamper-Resistant and Tamper-Evident Design
Evidence bags used in forensic investigations are specifically designed to reveal any unauthorized access:

- **Tamper-evident seals** — adhesive closures that leave a permanent, visible mark ("VOID" or a similar pattern) if the bag is opened after sealing. Once sealed, any opening of the bag is permanently recorded on the bag itself.
- **Sequential numbering** — each bag has a unique identifier that is logged in the evidence documentation, creating a link between the physical bag and the paper/digital record
- **Material integrity** — the bag itself is designed to resist tearing, puncturing, and moisture to protect the contents during storage and transport

### Labeling and Documentation
Every evidence bag must be labeled at the time of sealing with the following information:

- **Case number** — the unique identifier for the investigation
- **Item number** — the unique identifier for this specific piece of evidence within the case
- **Description** — what the item is (make, model, serial number where applicable)
- **Collection location** — exactly where the item was found or seized
- **Date and time of collection** — precise timestamp
- **Collector's name and signature** — the person who collected and sealed the item
- **Hash value** — for digital storage media, the hash of the device or image recorded at time of collection

The label is the bridge between the physical evidence and the written chain of custody record — they must be consistent and cross-referenced.

---

## 3. Legal Holds

A **legal hold** (also called a **litigation hold**) is a formal instruction to preserve all data that may be relevant to anticipated or ongoing legal proceedings, regulatory investigations, or audits.

### What It Is
When litigation is reasonably anticipated — whether a lawsuit, a criminal investigation, or a regulatory inquiry — organizations have a legal obligation to preserve potentially relevant data. A legal hold suspends the normal data lifecycle (routine deletion, log rotation, backup overwriting) for all data that falls within its scope.

### Why It Matters
Failing to preserve data subject to a legal hold — even accidentally, through routine automated deletion — constitutes **spoliation**: the destruction of evidence. Courts can impose severe sanctions for spoliation, including:
- Adverse inference instructions (the jury is told to assume the destroyed evidence would have been damaging to the party that destroyed it)
- Monetary sanctions
- In extreme cases, dismissal of claims or default judgment

### Scope and Duration
A legal hold:
- Is issued in writing by legal counsel and acknowledged in writing by the custodians responsible for the relevant data
- Specifies what data must be preserved (by data type, system, user, or time range)
- Remains in effect **until the resolution of the legal matter** — not until the IR team considers the incident closed. Legal proceedings can outlast the technical investigation by months or years.
- Must be actively managed — new custodians added, system changes tracked, scope adjusted if the legal matter evolves

> **Practical note:** the IR team's instinct to clean up and return systems to normal may conflict directly with a legal hold. Legal must always be consulted before any evidence is deleted, systems are wiped, or backups are overwritten during or after an incident.

---

## 4. Documentation

Documentation is the written record of every action taken during the forensic investigation. It is what allows another qualified investigator to reproduce the work, and what allows the investigation to be defended in court or in a regulatory proceeding.

### Collection Procedure
A documented account of exactly how evidence was collected:
- Which tool was used for acquisition (name, version, and configuration)
- The exact steps followed during collection
- Any deviations from standard procedure and the reason for them
- The state of the device at the time of collection (powered on/off, display contents, connected peripherals)

Procedure documentation ensures reproducibility — if the same steps are followed on the same evidence, the same results should be obtained.

### Tools
All forensic tools used throughout the investigation must be documented:
- Tool name and version number
- Verification that the tool is validated and functioning correctly (many labs maintain validated tool sets with known-good outputs for comparison)
- Any relevant configuration settings

Tool documentation matters because defense challenges often target the reliability of forensic software. A documented, validated, industry-standard tool is far easier to defend than an ad hoc script.

### Conditions
The environmental and situational conditions at the time of collection:
- Physical location (address, room, position within room)
- Environmental conditions (temperature, humidity — relevant for media integrity)
- Network status of the device at time of collection (online/offline, connected to which networks)
- Any unusual circumstances (signs of physical damage, evidence of prior tampering, unusual software behavior observed)

### Observations
Real-time notes recorded during the investigation — things the investigator noticed that may not be captured automatically by tools:
- What was displayed on a powered-on device's screen at the time of collection
- Unusual files, processes, or behaviors observed during initial examination
- Physical observations about the scene
- Statements made by individuals present during collection

Observations recorded contemporaneously (at the time, not reconstructed later) carry significantly more weight than recollections written up days after the fact.

---

## 5. The Chain of Custody

The chain of custody is the cumulative, unbroken record of everything described above — a living document that tracks the evidence from the moment it is identified through every transfer, examination, and storage event until the investigation is closed.

### Tracking Evidence Throughout the Entire Investigation
The chain of custody must account for the evidence at every moment — not just at collection and at presentation, but at every point in between:

- Who has physical or logical possession of the evidence right now?
- Where is it stored?
- Has it been accessed since the last entry — and if so, by whom, when, and for what purpose?

Any moment where the evidence's location or custodian is unaccounted for is a gap in the chain — and any gap is a potential challenge to admissibility.

### Documenting Evidence Possession
Every transfer of evidence from one person or location to another must be documented at the time of transfer:

| Field | What to Record |
|-------|---------------|
| **From** | Name, role, and signature of the person releasing the evidence |
| **To** | Name, role, and signature of the person receiving the evidence |
| **Date and time** | Exact timestamp of the transfer |
| **Reason** | Purpose of the transfer (e.g., transport to lab, examination, return to storage) |
| **Condition** | Condition of the evidence at transfer (seals intact, bag undamaged, hash value re-verified) |

Both parties sign at every transfer — the release signature and the receipt signature create a two-sided paper record that neither party can later deny.

### Storage Conditions
The conditions in which evidence is stored are part of the chain of custody record:

- **Physical security** — locked evidence room or safe with access log; only authorized personnel can enter
- **Access log** — every access to the evidence storage location is recorded, even if the evidence itself is not removed
- **Environmental controls** — appropriate temperature and humidity to prevent media degradation
- **Logical security** — for forensic images stored digitally, access-controlled storage with audit logging; hash verification at each access to confirm the image has not been altered since the previous access

---

## Summary

```
EVIDENCE COLLECTED
        │
        ▼
┌─────────────────────────────────────┐
│  FORENSIC DATA INTEGRITY            │
│  ─ Write blocker in place           │
│  ─ Forensic image created           │
│  ─ Hash verified (original = image) │
│  ─ Original sealed, never touched   │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│  EVIDENCE BAG                       │
│  ─ Tamper-evident seal applied      │
│  ─ Label completed (case#, hash,    │
│    collector, date/time, location)  │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│  LEGAL HOLD (if applicable)         │
│  ─ Issued by legal counsel          │
│  ─ Normal deletion/rotation paused  │
│  ─ Active until legal resolution    │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│  DOCUMENTATION                      │
│  ─ Collection procedure recorded    │
│  ─ Tools documented (name/version)  │
│  ─ Conditions and observations      │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│  CHAIN OF CUSTODY RECORD            │
│  ─ Every transfer signed (from/to)  │
│  ─ Storage conditions logged        │
│  ─ Hash re-verified at each access  │
│  ─ Unbroken from collection to      │
│    presentation                     │
└─────────────────────────────────────┘
```

| Area | Key Requirement |
|------|----------------|
| Write blockers | Hardware preferred; prevents any write to evidence device |
| Forensic images | All analysis on copies; original sealed and untouched |
| Evidence bags | Tamper-evident; fully labeled at time of sealing |
| Legal holds | Issued by legal; suspends deletion until legal resolution |
| Documentation | Procedure, tools, conditions, observations — all recorded at time of action |
| Chain of custody | Every possession, transfer, and storage event documented and signed |
