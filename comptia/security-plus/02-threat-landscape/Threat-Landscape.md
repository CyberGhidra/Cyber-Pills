# Threat Landscape — CompTIA Security+

## 1. Classification of Security Threats

Before analyzing specific threat actors, it's important to understand the **dimensions** used to classify and compare them. CompTIA Security+ evaluates threat actors along four main axes:

| Dimension | Description |
|---|---|
| **Location (Internal vs. External)** | Whether the threat originates from inside the organization (employees, contractors, insiders) or from outside it (external attackers, criminal groups) |
| **Level of Sophistication/Capability** | The technical skill level of the actor — ranges from unskilled "script kiddies" using pre-built tools, to highly sophisticated nation-state operators developing custom malware and zero-day exploits |
| **Resources and Funding** | The amount of financial and material backing available — a lone individual has minimal resources, while nation-states and organized crime groups can fund large, sustained operations |
| **Intent and Motivation** | The underlying goal driving the attack (financial gain, espionage, disruption, ideology, revenge, etc. — covered in detail in Section 3) |

> 🧩 **Internal vs. External example:** A disgruntled employee who steals data before resigning is an *internal* threat — the danger doesn't always come from outside the perimeter. This is why insider threat programs and least-privilege access are just as critical as perimeter defenses like firewalls.

---

## 2. Hacker Ethics Classification: White, Black, and Grey Hat

| Type | Description | Authorization |
|---|---|---|
| **White Hat** | Ethical hackers who test systems with explicit permission to find and report vulnerabilities | Authorized |
| **Black Hat** | Malicious attackers who exploit systems for personal gain, damage, or disruption, without permission | Unauthorized |
| **Grey Hat** | Operates in a legal/ethical grey area — may discover and disclose vulnerabilities without prior authorization, but typically without malicious intent | Unauthorized, but not malicious |

> 💡 **Exam tip:** The key differentiator is **authorization**, not skill level. A grey hat might have good intentions (e.g., responsibly disclosing a bug) but is still acting outside the boundaries of a legal agreement — which is what separates them from a white hat.

---

## 3. Threat Actors

### 3.1 Types of Threat Actors

| Actor | Description | Typical Sophistication |
|---|---|---|
| **Unskilled Attackers** (Script Kiddies) | Individuals with limited technical knowledge who use pre-made tools/scripts created by others | Low |
| **Hacktivists** | Attackers motivated by political, social, or ideological causes; aim to raise awareness or protest through disruption/defacement | Low–Medium |
| **Organized Crime** | Professional criminal groups conducting attacks for financial gain (ransomware, fraud, data theft for resale) | Medium–High, well-funded |
| **Nation-State Actors** (APT — Advanced Persistent Threat) | Government-sponsored groups with extensive resources and long-term strategic objectives (espionage, sabotage, geopolitical advantage) | Very High |

**Nation-state actors** are notably associated with **zero-day attacks** — exploits that target a vulnerability *before* the vendor is aware of it or has released a patch. Because these vulnerabilities are unknown to defenders, zero-days are extremely valuable and are often reserved for high-priority targets due to the resources required to discover or purchase them.

### 3.2 Other Notable Threat Sources

| Source | Description |
|---|---|
| **Insider Threats** | Current or former employees, contractors, or business partners who misuse legitimate access — can be malicious (intentional harm) or negligent (unintentional mistakes) |
| **Competitors** | Rival organizations that may engage in corporate espionage to steal intellectual property, trade secrets, or gain competitive advantage |

> 🧩 Insider threats are especially dangerous because the actor already has **legitimate access**, bypassing many perimeter-based defenses — this is why monitoring, auditing, and the principle of least privilege are essential mitigations (see Data Protection notes from Part 1).

---

## 4. Motivations of Threat Actors

Understanding *why* an attacker acts helps predict *how* they will act and what they will target.

| Motivation | Description |
|---|---|
| **Data exfiltration** | Stealing sensitive data (PII, financial data, credentials) for resale or misuse |
| **Espionage** | Covertly gathering confidential/strategic information, often on behalf of a nation-state or competitor |
| **Service disruption** | Disabling or degrading systems/services (e.g., DoS/DDoS attacks) |
| **Blackmail / targeted extortion** | Threatening to release stolen data or disrupt operations unless a ransom is paid (e.g., ransomware, double extortion) |
| **Financial gain** | Directly profiting from the attack (fraud, theft, cryptomining, selling stolen data) |
| **Philosophical/political beliefs** | Acting in support of an ideology or cause (see Hacktivism) |
| **Ethical attacks** | Conducted with good intent, typically to identify and report vulnerabilities (white/grey hat activity) |
| **Revenge** | Personal retaliation, often by disgruntled former employees or individuals with a grievance |
| **Disruption/Chaos** | Causing damage or disorder for its own sake, without a specific strategic goal |
| **War (Cyberwarfare)** | State-sponsored attacks conducted as part of military/geopolitical conflict, targeting critical infrastructure or military systems |

---

## 5. Threat Vectors and Attack Surfaces

A **threat vector** is the method or pathway an attacker uses to gain access to a target. The **attack surface** is the sum of all possible points where an unauthorized user could attempt to enter or extract data.

### 5.1 Common Threat Vectors

| Vector | Description |
|---|---|
| **Message-based** | Phishing emails, SMS (smishing), instant messaging — used to deliver malicious links/attachments or manipulate victims |
| **Wired networks** | Physical network connections exploited via direct access (e.g., plugging in a rogue device) |
| **Wireless networks** | Attacks against Wi-Fi (rogue access points, evil twin attacks, weak encryption exploitation) |
| **Systems** | Exploiting vulnerabilities in operating systems, applications, or unpatched software |
| **Image files** | Malicious code hidden within image files (steganography) or exploiting image-parsing vulnerabilities |
| **Removable devices** | USB drives and other portable media used to introduce malware (e.g., "USB drop attacks") |
| **Cloud** | Exploiting misconfigured cloud storage, weak API security, or compromised cloud credentials |
| **Supply chain** | Compromising a trusted third-party vendor, software update, or hardware component to indirectly reach the real target |

> 💡 **Exam tip:** Supply chain attacks are increasingly emphasized in Security+ because they bypass an organization's own defenses entirely — the attacker compromises a *trusted* upstream provider instead (e.g., a compromised software update).

---

## 6. Threat Data and Intelligence

**Threat intelligence** is evidence-based knowledge about existing or emerging threats, used to inform security decisions.

### 6.1 Key Foundations

| Term | Description |
|---|---|
| **OSINT** (Open-Source Intelligence) | Threat information gathered from publicly available sources (websites, social media, forums, public records) |
| **CVE** (Common Vulnerabilities and Exposures) | A standardized, publicly available database/naming system for known vulnerabilities, allowing organizations to track and reference specific flaws consistently across tools and vendors |

### 6.2 Open-Source Threat Intelligence Sources

| Source | Focus |
|---|---|
| **Senki.org** | Community-driven network security best practices and threat information |
| **AlienVault Open Threat Exchange (OTX)** | Community-powered platform for sharing threat indicators and research |
| **MISP** (Malware Information Sharing Platform) | Open-source platform for sharing structured threat intelligence between organizations |
| **ThreatFeeds.io** | Aggregator/directory of publicly available threat intelligence feeds |
| **SANS Internet Storm Center** | Analysis and alerts on emerging threats from the SANS community |
| **VirusShare** | Repository of malware samples for research purposes |
| **Spamhaus Project** | Tracks spam and malicious sources, providing blocklists for email/network security |

### 6.3 Proprietary / Closed-Source Intelligence

Some organizations subscribe to **paid, proprietary threat intelligence feeds** from vendors, which typically offer more curated, timely, and context-rich data than free/open sources.

**When a threat feed fails**, it typically means one of the following:
- The data is **outdated** (not delivered in time to act — see timeliness, below)
- The information is **inaccurate** (false positives, unreliable sourcing)
- The information is **not relevant** to the organization's actual environment or risk profile
- The feed lacks proper **context** (an indicator without explanation of the associated threat/TTPs has limited actionable value)

### 6.4 Evaluating Threat Intelligence

Not all threat intelligence is equally useful. CompTIA emphasizes three key evaluation criteria:

| Criterion | Key Question |
|---|---|
| **Timeliness** | Is the information delivered quickly enough to act on before the threat impacts the organization? |
| **Accuracy** | Is the information correct and verified, minimizing false positives/negatives? |
| **Relevance** | Does the information actually apply to this organization's systems, industry, or threat profile? |

> 🧩 Intelligence that is accurate but arrives too late — or is timely but irrelevant to your environment — has little practical value. All three criteria must be satisfied together.

### 6.5 Sharing Threat Indicators

To make threat intelligence actionable and shareable between organizations and tools, standardized formats are used:

| Standard | Description |
|---|---|
| **STIX** (Structured Threat Information eXpression) | A standardized, structured language (typically JSON-based) for describing threat intelligence — including indicators, attack patterns, and threat actor behavior — in a machine-readable format |
| **OpenIOC** (Open Indicators of Compromise) | An XML-based framework for describing and sharing indicators of compromise (IOCs), such as file hashes, IP addresses, or registry keys associated with an attack |

**Example — a simplified STIX indicator in JSON:**

```json
{
  "type": "indicator",
  "id": "indicator--8e2e2d2b-17d4-4cbf-938f-98ee46b3cd3f",
  "created": "2026-01-15T10:00:00.000Z",
  "name": "Malicious IP associated with C2 traffic",
  "pattern": "[ipv4-addr:value = '203.0.113.55']",
  "valid_from": "2026-01-15T10:00:00.000Z",
  "labels": ["malicious-activity"]
}
```

This structure allows security tools (SIEMs, firewalls, threat intel platforms) to automatically ingest and act on the indicator (e.g., automatically block the IP) without manual interpretation.

> 💡 **Exam tip:** STIX is often paired with **TAXII** (Trusted Automated eXchange of Indicator Information) — the transport protocol used to actually *distribute* STIX-formatted data between systems. STIX = the format, TAXII = the delivery mechanism.

### 6.6 Conducting Independent Research

Beyond automated feeds, security professionals should actively research emerging threats through:

- **Vendor websites** — security bulletins and advisories published directly by software/hardware vendors
- **Vulnerability feeds** — CVE databases, NVD (National Vulnerability Database), vendor-specific advisories
- **Academic journals** — peer-reviewed research on emerging attack techniques and defenses
- **Conferences** — industry events (e.g., DEF CON, Black Hat, RSA Conference) where new research and techniques are presented
- **Social media accounts** — security researchers, vendors, and organizations often share real-time threat updates and analysis

---

## Summary

| Concept Group | Key Idea |
|---|---|
| Threat Classification | Internal/external, sophistication, resources, motivation |
| Hat Classification | White (authorized), Black (malicious), Grey (unauthorized, non-malicious) |
| Threat Actors | Unskilled attackers, hacktivists, organized crime, nation-states (APT, zero-days) |
| Other Sources | Insider threats, competitors |
| Motivations | Data theft, espionage, disruption, extortion, financial gain, ideology, ethics, revenge, chaos, war |
| Threat Vectors | Message-based, wired/wireless networks, systems, image files, removable media, cloud, supply chain |
| Threat Intelligence Basics | OSINT, CVE database |
| Open-Source Feeds | Senki.org, OTX, MISP, ThreatFeeds.io, SANS ISC, VirusShare, Spamhaus |
| Closed-Source Intelligence | Paid/proprietary feeds; failure = outdated, inaccurate, or irrelevant data |
| Evaluating Intelligence | Timely, accurate, relevant |
| Sharing Standards | STIX (structured format, often JSON) + TAXII (transport); OpenIOC (XML-based) |
| Independent Research | Vendor sites, vulnerability feeds, academic journals, conferences, social media |

---
