# Incident Response — Identification

---

## Why Identification Is Critical

Preparation builds the infrastructure; Identification is where it gets used for the first time. The faster and more accurately a team identifies an incident, the smaller the window the attacker has to operate — and the less damage is done before containment begins.

The challenge of Identification is not just *finding* something suspicious — it is correctly distinguishing a real incident from the constant background noise of alerts, false positives, and routine anomalies that any security monitoring system generates. Getting this wrong in either direction is costly: miss a real incident and the attacker moves freely; over-respond to false positives and the team burns out chasing ghosts.

> **Key principle:** Identification answers three questions — *Is this real? How bad is it? Who needs to know right now?*

---

## 1. Identification Sources

An effective IR program draws signals from multiple detection layers simultaneously. Relying on a single source creates blind spots; attackers know this and deliberately operate in the gaps between monitoring tools.

### Network Perimeter
Firewalls, IDS/IPS (Intrusion Detection/Prevention Systems), web proxies, and DNS logs at the boundary of the network. These catch:
- Inbound exploitation attempts
- Unusual outbound connections (e.g., C2 callbacks like the reverse shell in the previous labs)
- DNS requests to known malicious domains
- Large or unexpected data transfers (potential exfiltration)

The network perimeter is often the first place a reverse shell or C2 channel becomes visible — as demonstrated in the Linux Network Analysis lab with `netstat` and `ss`.

### Email Perimeter
Email is the single most common initial access vector. Email security gateways (SEGs) analyze inbound messages for:
- Malicious attachments (macro-enabled Office files, PDFs with embedded exploits)
- Phishing links (URLs leading to credential harvesting pages)
- Spoofed sender domains (look-alike domains, missing SPF/DKIM)
- Business Email Compromise (BEC) patterns

Even with good filtering in place, some malicious emails get through — which is why end user reporting (covered in Preparation) is the complementary human layer.

### Host Perimeter
Detection at the individual endpoint level, primarily via EDR tools (like LimaCharlie, covered in the previous lab) and AV solutions. Host-level detection catches:
- Malicious process execution (e.g., the `HACKTOOL` alert Mimikatz triggered in LimaCharlie)
- Suspicious file writes, especially to temp directories or system folders
- Registry modifications for persistence (e.g., Run keys)
- Process injection and credential dumping attempts
- Lateral movement tools being launched

Host perimeter detection is often the most precise — it sees exactly what is running on a specific machine, not just network traffic between machines.

### Application-Level Detection
Monitoring within applications themselves — web application firewalls (WAF), database activity monitoring (DAM), application logs. These catch:
- SQL injection and web exploitation attempts
- Unusual query patterns suggesting data exfiltration
- Authentication anomalies within business applications
- API abuse

This layer catches threats that have already passed through the network and host perimeter and are interacting directly with business logic.

### Threat Intelligence
External feeds providing known indicators of compromise (IOCs) — malicious IPs, domains, file hashes, TTPs (Tactics, Techniques, and Procedures). Formats like **STIX/TAXII** and **OpenIOC** allow automated ingestion into SIEMs and EDR platforms.

Threat intelligence converts "this looks unusual" into "this matches a known threat actor's infrastructure" — dramatically accelerating triage. The SHA-256 hash lookup on VirusTotal used in the previous labs is a manual form of threat intelligence correlation.

### User and Entity Behavior Analytics (UEBA)
UEBA tools establish a **behavioral baseline** for each user and system over time, then alert when behavior deviates significantly from that baseline. Examples:
- A user who normally works 9–5 from one location suddenly logging in at 3 AM from a foreign IP
- A service account that has never accessed file shares suddenly reading thousands of files per minute
- An admin account performing actions that no admin has ever performed before

UEBA is particularly valuable for detecting **insider threats** and **compromised credentials** — scenarios where the attacker is using legitimate credentials and therefore generating no signature-based alerts at all.

### Human-Level Detection
Trained security analysts reviewing logs, alerts, and anomalies that automated tools flagged but did not definitively classify. Human judgment remains essential because:
- Attackers actively study and evade automated detection rules
- Some attack patterns are subtle enough to look legitimate to a machine but obvious to an experienced analyst
- Correlating signals across multiple data sources still often requires human reasoning

### User Reports
End users noticing and reporting something odd — a machine running slowly, a strange popup, an email asking for unusual information, a colleague behaving strangely. As covered in Preparation, making reporting easy and rewarding is essential because users are distributed across the entire organization, covering ground no monitoring tool can fully reach.

> **Detection tip:** The most dangerous incidents are often identified last by automated tools and first by a user who noticed something felt wrong. Never dismiss user reports as noise without investigation.

---

## 2. Alert Tuning

Having detection sources in place is not enough — the quality of what those sources produce determines whether analysts can actually act on the signals.

### Alert Fatigue
Alert fatigue occurs when the volume of alerts generated is so high that analysts cannot realistically investigate all of them. When analysts are overwhelmed:
- Real incidents get buried in the noise and missed
- Analysts start dismissing alerts without proper investigation just to keep pace
- The security monitoring system becomes theater — it looks like detection is happening, but it isn't effectively

Alert fatigue is one of the most serious operational problems in security operations, and it is almost always a symptom of **poorly tuned detection rules** generating too many false positives.

### Signal-to-Noise Ratio
The goal of alert tuning is to maximize the **signal-to-noise ratio** — keeping the alerts that represent genuine threats (signal) and suppressing or removing the alerts that don't (noise).

Tuning strategies:
- **Suppressing known-good behavior** — if a legitimate script runs every night and triggers a detection rule, whitelist that specific behavior rather than disabling the rule entirely
- **Raising thresholds** — some rules are too sensitive by default; adjust them so they only fire on genuinely abnormal patterns
- **Correlating events** — instead of alerting on individual low-severity events (which may be noise), alert only when multiple related events occur together within a time window (correlation = higher confidence)
- **Regular rule review** — detection rules drift out of relevance as environments change; review them periodically and retire rules that no longer provide value

> **Balance:** over-tuning (suppressing too aggressively) creates blind spots. Under-tuning (allowing too many false positives) creates alert fatigue. The correct balance requires ongoing, iterative work — not a one-time configuration.

---

## 3. Incident Prioritization

Not all incidents are equal. When multiple alerts or incidents are active simultaneously, the team must triage them to allocate resources effectively.

### Incident Triage
Triage is the rapid initial assessment of an incident to determine whether it is real, what type it is, and how urgently it needs a response. Borrowed from medical terminology — the same word is used in emergency medicine for the same reason: limited resources must go to where they matter most.

### Scope
How far has the incident spread?
- Is it isolated to a single endpoint, or have multiple systems been affected?
- Has the attacker moved laterally across the network?
- Are external parties (customers, partners, cloud services) involved?

Scope determines how many resources need to be mobilized and whether the incident can be handled internally or requires external forensics support.

### Urgency
How fast is the situation evolving?
- Is the attacker actively operating right now, or did this happen in the past?
- Is data being actively exfiltrated at this moment?
- Is a critical system at risk of going offline?

Urgency drives the speed of response. An attacker actively dumping data demands an immediate response; a historical indicator discovered weeks after the fact allows for a more methodical investigation.

### Criticality
How important are the affected systems to the organization?
- Is it a developer's workstation or the domain controller?
- Is it a test environment or the production database with customer PII?

Criticality informs how aggressively you respond and how much disruption you can accept from the response itself (e.g., taking a system offline hurts more if it's a critical production system).

### Impact
What is the actual or potential damage?
- **Confidentiality:** has sensitive data been accessed or exfiltrated?
- **Integrity:** have systems or data been modified or corrupted?
- **Availability:** are systems down or at risk of going down?
- **Regulatory:** does this trigger mandatory breach notification requirements?
- **Reputational:** is there risk of public disclosure that damages trust?

Combining scope, urgency, criticality, and impact produces an **overall severity rating** (commonly Low / Medium / High / Critical) that drives resource allocation and escalation decisions.

---

## 4. Documentation

The same documentation discipline introduced in Preparation applies with full force during Identification — and this is the first phase where it must be actively practiced under pressure.

- **Tickets** — every identified incident should have a corresponding ticket opened immediately upon confirmation, even before the full picture is clear. The ticket captures the initial indicator, who identified it, when, and what the first assessment is. It grows as the investigation progresses.
- **Reports** — structured summaries produced at defined intervals (hourly, daily, or at key milestones) for distribution to stakeholders. Reports translate technical findings into language that non-technical recipients can act on.
- **Record of events** — a chronological log of every action taken and every finding made, with timestamps. This is the raw material from which the incident timeline is built — essential for both the investigation itself and the post-incident review.

> **Reminder from Preparation:** slow down and document as you go. The temptation during Identification is to jump straight into investigation without recording what you're doing. Resist this — undocumented findings are far harder to defend, share, or reproduce.

---

## 5. Communication and Notification

Once an incident is confirmed, the right people need to know — quickly, clearly, and through the right channels.

### Notify Relevant Parties
Using the contact lists and escalation protocols built during Preparation:
- IR team lead and team members
- System and data owners of affected assets
- IT operations (if systems need to be isolated or taken offline)
- Legal, if data or regulatory obligations may be involved
- Management, according to the pre-defined escalation thresholds

### Situation Report (SITREP)
A **SITREP** is a concise, structured update sent to key stakeholders at regular intervals throughout an incident. It is not a technical deep-dive — it is a summary designed for people who need to make decisions but don't have time to read raw logs.

A good SITREP includes:

| Section | Content |
|---------|---------|
| **Clear summary** | What happened, in 2–3 sentences, in plain language |
| **Severity** | Current classification (Low / Medium / High / Critical) |
| **Status** | Where the response currently stands (investigating / contained / remediating) |
| **Actions taken** | What has been done so far |
| **Next steps** | What the team is doing next and when the next update will be |

### Communication Planning
As covered in Preparation, all incident communications should use **pre-approved channels** — including out-of-band channels if the affected network may be compromised. Key principles:

- **Need to know** — only share incident details with people who need them. Wider distribution increases the risk of information leaking to the wrong person (including a potential insider threat).
- **Consistent messaging** — everyone should be hearing the same facts. Inconsistent information creates confusion, undermines trust, and can cause well-meaning people to take unauthorized actions.
- **Cadence** — set expectations upfront about how often updates will come. Stakeholders who don't hear anything will start calling for updates, distracting the IR team from the actual work.

---

## Summary

```
DETECTION SOURCES          ALERT QUALITY           PRIORITIZATION
──────────────────         ─────────────           ──────────────
Network perimeter    ──►   Alert tuning     ──►    Triage
Email perimeter            Signal/noise            Scope
Host (EDR)                 ratio balance           Urgency
Application layer                                  Criticality
Threat intelligence                                Impact
UEBA                                                   │
Human analysts                                         ▼
User reports                                    SEVERITY RATING
                                                       │
                                            ┌──────────▼──────────┐
                                            │ Document + Notify   │
                                            │ Tickets / SITREPs   │
                                            │ Relevant parties    │
                                            └─────────────────────┘
```

| Area | Key Activity |
|------|-------------|
| Detection Sources | Monitor all layers: network, email, host, application, threat intel, UEBA, humans, users |
| Alert Tuning | Reduce noise, improve signal-to-noise ratio, prevent alert fatigue |
| Prioritization | Triage by scope, urgency, criticality, and impact → assign severity |
| Documentation | Open a ticket immediately, log all findings with timestamps |
| Communication | Notify relevant parties via pre-approved channels, issue regular SITREPs |
