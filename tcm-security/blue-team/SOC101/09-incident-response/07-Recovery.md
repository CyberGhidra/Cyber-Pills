# Incident Response — Recovery

---

## The Goal of Recovery

Recovery is the final operational phase before the post-incident review. By this point, the attacker's artifacts have been removed (Eradication) and the environment has been hardened. Recovery answers one question: **are the affected systems back in production, confirmed clean, and properly monitored?**

It is tempting to treat Recovery as simply "turning things back on" — but done carelessly, it reintroduces risk. Systems returned to production before functionality is verified may fail under load; systems returned without heightened monitoring may be re-compromised without detection. Recovery is a controlled, validated re-entry into normal operations, not just a power-on.

> **Key principle:** A system is not recovered until it has been tested, verified clean, and confirmed monitored — not just when it is running again.

---

## 1. Getting Affected Systems Back into Production

### Restoring Systems
Before any system is returned to production, it must be in a definitively clean state — either restored from a verified pre-incident backup or rebuilt from a clean image, as carried out during Eradication. The transition from Eradication to Recovery is the confirmation that this work is complete:

- All attacker artifacts have been removed (malware, backdoors, unauthorized accounts, malicious cron jobs, web shells)
- The system has been patched against the exploited vulnerability and hardened per CIS Benchmarks
- Credentials that may have been compromised have been reset
- The forensic image and chain of custody documentation are secured and filed

Only when all of these are confirmed should a system be considered a candidate for return to production.

### Removing Remaining Artifacts
Recovery is also the final checkpoint for artifact removal — a last sweep before the system is reconnected to the production environment. This includes:

- Confirming no unauthorized processes are running (`ps -AFH`, `lsof -i`)
- Confirming no unauthorized network connections exist or are established on reconnection (`netstat -tnp`, `ss -tnp`)
- Confirming no unauthorized persistence mechanisms remain (cron, scheduled tasks, registry autoruns, startup services)
- Confirming file system integrity — recently modified system files should be verified against known-good hashes

> **Why check again here?** Eradication may have been done under time pressure, across multiple systems simultaneously, or by multiple team members. A final pre-production check catches anything that slipped through — it is far better to find a missed artifact here than after the system is live.

---

## 2. Confirming Functionality

Returning a system to production without testing it first risks two outcomes: either the system fails in a way that causes operational disruption, or it appears to work while actually being broken in a subtle way that only surfaces later. Neither is acceptable.

### Functionality Testing
Functionality testing confirms that the system performs its intended business function correctly after restoration:

- **Application testing** — verify that all applications and services running on the system start correctly, respond as expected, and produce correct output
- **Integration testing** — verify that the restored system communicates correctly with the other systems it depends on (databases, authentication services, APIs, downstream systems)
- **User acceptance testing** — where appropriate, have a representative end user or the system's data owner confirm that the system behaves as expected from a business perspective, not just a technical one
- **Performance baseline** — confirm the system is performing within expected parameters (CPU, memory, disk I/O, response times) — anomalous resource consumption post-recovery can indicate a missed artifact

Testing should be done in a controlled way before full traffic is restored — for example, by reconnecting the system to the network in a monitored segment first, or by restoring a subset of users before opening access to all.

### Staged Return to Production
Rather than reconnecting a restored system to full production traffic immediately, a staged approach reduces risk:

```
Isolated testing  →  Limited user access  →  Full production traffic
     (verify)           (observe)                  (confirm clean)
```

Each stage is a checkpoint — if something looks wrong, it is far easier to pull back before full production load has been restored than after.

---

## 3. Preventing Future Incidents

Recovery is not just about restoring what was lost — it is about ensuring the same incident does not happen again. The measures implemented here are the operational translation of the lessons learned from the incident.

### Hardening
Hardening goes beyond patching the specific vulnerability that was exploited. The incident has revealed the real-world attack surface of the affected systems — use that knowledge:

- Apply CIS Benchmark configurations to restored systems before returning them to production (not after)
- Remove software, services, and accounts that are not strictly necessary (attack surface reduction)
- Enforce least-privilege across all accounts — no user or service account should have more permissions than their role requires
- Enforce MFA on all accounts that did not previously have it, prioritizing privileged and remote access accounts
- Review and tighten network access controls — systems should only be reachable from the systems and users that legitimately need access to them

### Scanning
Before and after returning to production, scan the restored system for residual vulnerabilities:

- Run an authenticated vulnerability scan to confirm the patching done during Eradication was successful and no new vulnerabilities were introduced during rebuilding
- Scan for residual IOCs — known malicious file hashes, IP connections, domain lookups — to confirm eradication is complete
- Include the restored system in the regular vulnerability scanning cycle going forward, with higher priority than before if it was previously considered low-risk

### Monitoring
The period immediately following recovery is the highest-risk window for re-compromise — the attacker knows the environment, may have access through vectors not yet discovered, and may attempt to re-enter while the team's guard is lowering. Enhanced monitoring during this window is essential.

- Increase log collection verbosity on restored systems — capture more events than normal during the monitoring window
- Add specific detection rules targeting the exact TTPs used in this incident, so any re-use of the same techniques is immediately flagged
- Monitor the attacker's known IOCs (IPs, domains, hashes) as active watchlist items in the SIEM
- Brief the SOC and relevant system owners on what to watch for — human vigilance is a monitoring layer too

---

## 4. Recovery Decisions

Recovery requires explicit decision-making about timing, testing methodology, monitoring duration, and tooling. These decisions should be made deliberately — not by default or under pressure — and documented.

### Recovery Time Window
When is it appropriate to return a system to production?

The recovery time window is governed by the RTO (Recovery Time Objective) defined during Preparation, balanced against the confidence level that eradication is complete. The tension between these two forces is one of the most common sources of disagreement during incident recovery:

- **Business pressure** pushes for the fastest possible return to production (minimize downtime, revenue impact, user disruption)
- **Security pressure** pushes for sufficient time to verify the system is clean and properly hardened before exposure

Neither can be ignored. The correct answer is the shortest time window that allows thorough verification — not the shortest time window possible, and not an indefinite delay waiting for perfect certainty that will never arrive. The IR lead, data owner, and relevant stakeholders should agree on this window explicitly before recovery begins.

### How to Test and Verify Functionality
The testing approach must be defined before recovery starts, not improvised during it:

- What specific tests will be run, and who will run them?
- What does "pass" look like — what specific outputs, response times, or behaviors confirm the system is working correctly?
- Who has the authority to declare testing complete and authorize return to production?
- What is the rollback plan if testing reveals a problem?

Having clear, pre-agreed criteria for "recovery complete" removes ambiguity and prevents the common scenario where a system is returned to production because no one wanted to make the call that it wasn't ready.

### Duration of Monitoring
How long should enhanced monitoring continue after recovery?

There is no universal answer — it depends on the severity of the incident, the attacker's sophistication, and the confidence level in eradication completeness. Common approaches:

| Incident Severity | Monitoring Duration |
|------------------|-------------------|
| Low (single host, contained quickly) | 2–4 weeks enhanced monitoring |
| Medium (multiple hosts, some lateral movement) | 4–8 weeks, with formal review at the end |
| High/Critical (widespread compromise, advanced attacker) | 90+ days, potentially permanent rule additions |

The monitoring period should have a defined end date and a defined review — not just fade out informally when attention moves on to other priorities. At the end of the monitoring window, a formal review confirms whether any re-compromise attempts were detected and whether it is safe to return monitoring to baseline levels.

### Tools for Monitoring
The tools used during the monitoring window should be the same ones used throughout the incident — familiarity with the tooling matters when speed of detection is critical:

| Tool Type | Examples | What It Covers |
|-----------|---------|---------------|
| **EDR** | LimaCharlie, CrowdStrike, Wazuh | Process execution, file writes, registry changes, network connections at the host level |
| **SIEM** | Splunk, Microsoft Sentinel, Elastic SIEM | Centralized log aggregation and correlation across all systems |
| **Network monitoring** | Zeek, Suricata, firewall logs | Traffic patterns, C2 callbacks, lateral movement between hosts |
| **Vulnerability scanner** | Nessus, OpenVAS | Ongoing detection of new vulnerabilities introduced post-recovery |
| **Threat intelligence platform** | MISP, OpenCTI | Automated IOC matching against active threat feeds |

The monitoring toolset should be validated and confirmed working before recovery begins — discovering that log forwarding is broken or that the EDR agent wasn't reinstalled on the rebuilt system is a problem that should surface during testing, not a week into the monitoring window.

---

## Summary

```
ERADICATION COMPLETE
        │
        ▼
┌───────────────────────────────────┐
│  RETURN TO PRODUCTION             │
│  ─ Confirm all artifacts removed  │
│  ─ Final pre-production sweep     │
│  ─ Staged reconnection            │
└───────────────┬───────────────────┘
                │
                ▼
┌───────────────────────────────────┐
│  CONFIRM FUNCTIONALITY            │
│  ─ Application testing            │
│  ─ Integration testing            │
│  ─ User acceptance testing        │
│  ─ Performance baseline check     │
└───────────────┬───────────────────┘
                │
                ▼
┌───────────────────────────────────┐
│  PREVENT FUTURE INCIDENTS         │
│  ─ Harden (CIS Benchmarks, MFA)   │
│  ─ Scan (vulnerability + IOCs)    │
│  ─ Enhanced monitoring active     │
└───────────────┬───────────────────┘
                │
                ▼
┌───────────────────────────────────┐
│  RECOVERY DECISIONS               │
│  ─ Recovery time window (RTO)     │
│  ─ Pass/fail criteria defined     │
│  ─ Monitoring duration agreed     │
│  ─ Tools validated and running    │
└───────────────┬───────────────────┘
                │
                ▼
        LESSONS LEARNED
```

| Area | Key Activity |
|------|-------------|
| Return to production | Final artifact sweep, staged reconnection, confirm clean state |
| Functionality testing | Application, integration, user acceptance, and performance testing |
| Hardening | CIS Benchmarks, least privilege, MFA, attack surface reduction |
| Scanning | Vulnerability scan post-rebuild, IOC sweep, ongoing scan cycle |
| Monitoring | Enhanced monitoring with defined duration, specific IOC watchlists, EDR/SIEM/network coverage |
| Recovery decisions | Agree on time window, test criteria, monitoring duration, and tooling before starting |
