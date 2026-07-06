# Linux Cron Jobs

**Reference:** [https://crontab.guru/](https://crontab.guru/) — interactive cron expression editor, useful for quickly reading or building cron schedules without memorizing the syntax.

---

## 1. What Is a Cron Job?

A **cron job** is a task scheduled to run automatically at a defined time or interval on a Linux/Unix system. The cron daemon (`crond`) runs silently in the background and checks the crontab files every minute — if a scheduled entry matches the current time, it executes the corresponding command.

Cron is used legitimately for things like:
- Automated backups
- Log rotation
- System maintenance scripts
- Software update checks

However, from a security perspective, cron is also one of the most common **persistence mechanisms** an attacker uses: by adding a malicious entry to a crontab, the payload re-executes automatically on a schedule — even after the original process is killed or the system reboots.

> **MITRE ATT&CK reference:** T1053.003 (Scheduled Task/Job: Cron) — attackers use cron to maintain persistent access or repeatedly execute malicious payloads.

---

## 2. Cron Syntax — Reading a Cron Expression

Every cron entry follows this structure:

```
MIN  HOUR  DOM  MON  DOW  COMMAND
```

| Field | Name | Allowed Values |
|-------|------|---------------|
| `MIN` | Minute | 0–59 |
| `HOUR` | Hour | 0–23 |
| `DOM` | Day of Month | 1–31 |
| `MON` | Month | 1–12 |
| `DOW` | Day of Week | 0–7 (0 and 7 both = Sunday) |
| `COMMAND` | Command/script to run | Any executable or shell command |

### Special symbols

| Symbol | Meaning |
|--------|---------|
| `*` | Any / every value |
| `*/n` | Every *n* units (e.g. `*/5` = every 5 minutes) |
| `,` | List of values (e.g. `1,15` = 1st and 15th) |
| `-` | Range (e.g. `1-5` = Monday through Friday) |

### Worked example

```
05 00 * * * /tmp/backup.sh
```

Reading it field by field:

| Field | Value | Meaning |
|-------|-------|---------|
| MIN | `05` | At minute 05 |
| HOUR | `00` | At hour 00 (midnight) |
| DOM | `*` | Every day of the month |
| MON | `*` | Every month |
| DOW | `*` | Any day of the week |
| COMMAND | `/tmp/backup.sh` | Run this script |

**In plain English:** *Run `/tmp/backup.sh` every day at 00:05 (12:05 AM).*

> **Security note:** `/tmp` is world-writable by any user — a script placed there and scheduled via cron is an obvious red flag. Legitimate backup scripts live in `/usr/local/bin`, `/opt`, or similar controlled directories, not in `/tmp`.

---

## 3. Where Cron Jobs Live — System Level

### 3.1 `/etc/crontab` — The Main System Crontab

```bash
cat /etc/crontab
```

This is the primary system-wide crontab file. Unlike user crontabs, `/etc/crontab` has an extra field: the **username** under which the command runs.

```
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7)
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *  *  *  *  root    cd / && run-parts --report /etc/cron.hourly
25 6  *  *  *  root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
```

The commented header is left in the file as a built-in reminder of the syntax — the lab instructor goes through it to reinforce how each field maps to what you learned in Section 2. The `run-parts` command executes all scripts inside a directory (e.g. `/etc/cron.daily`) — more on that below.

### 3.2 `/etc/cron.*` — Scheduled Directories

```bash
ls -al /etc | grep cron
```

Example output:
```
drwxr-xr-x  2 root root  4096 Jan 10 10:00 cron.d
drwxr-xr-x  2 root root  4096 Jan 10 10:00 cron.daily
drwxr-xr-x  2 root root  4096 Jan 10 10:00 cron.hourly
drwxr-xr-x  2 root root  4096 Jan 10 10:00 cron.monthly
drwxr-xr-x  2 root root  4096 Jan 10 10:00 cron.weekly
-rw-r--r--  1 root root  1042 Jan 10 10:00 crontab
```

Linux maintains separate directories for tasks that should run at fixed intervals, without needing an exact time:

| Directory | When scripts inside it run |
|-----------|--------------------------|
| `/etc/cron.hourly/` | Once per hour |
| `/etc/cron.daily/` | Once per day |
| `/etc/cron.weekly/` | Once per week |
| `/etc/cron.monthly/` | Once per month |
| `/etc/cron.d/` | Individual files with full cron syntax (package-managed jobs often land here) |

Any script dropped into one of these directories gets automatically executed on that schedule — no editing of crontab files required. This also makes them a potential attacker target: dropping a malicious script into `/etc/cron.daily/` gives daily persistent execution with no crontab entry to find.

### 3.3 Inspecting and Hashing `/etc/cron.daily/`

```bash
ls -al /etc/cron.daily/
```

Lists the scripts currently scheduled to run daily, along with their permissions and ownership. Scripts here should be:
- Owned by `root`
- Have restricted permissions (not world-writable)
- Have recognizable, legitimate names (e.g., `logrotate`, `apt-compat`, `dpkg`)

Any script with unusual ownership, a generic name, or recently modified timestamps is worth investigating further.

```bash
sha256sum /etc/cron.daily/*
```

Hashes every file in `/etc/cron.daily/` in one shot. The purpose is the same as hashing a suspicious binary: once you have the hash, you can:
- Track the file over time (if the hash changes unexpectedly, something modified it).
- Submit it to VirusTotal or compare against known-good baselines.
- Document it as part of an incident report.

This is baselining — knowing what "normal" looks like so changes stand out.

---

## 4. Adding a Malicious Cron Entry (Attacker Simulation)

```bash
ls /home/tcm/malware/
```

This confirms the malicious payload (`notmalware.elf`) is still in place — the same binary from the previous labs — and is ready to be configured as a persistent scheduled task.

```bash
crontab -e
```

**What this does:** Opens the **current user's** crontab for editing (no `sudo` = editing *your own* crontab, in this case the `tcm` user's). On this system it opens in GNU nano. Inside the editor, the attacker adds:

```
*/5 * * * * /home/tcm/malware/notmalware.elf
```

Reading the expression:
| Field | Value | Meaning |
|-------|-------|---------|
| MIN | `*/5` | Every 5 minutes |
| HOUR | `*` | Every hour |
| DOM | `*` | Every day |
| MON | `*` | Every month |
| DOW | `*` | Every day of the week |
| COMMAND | `/home/tcm/malware/notmalware.elf` | Re-execute the reverse shell payload |

**In plain English:** *Re-launch the reverse shell payload every 5 minutes, around the clock, every day.*

This is why cron is such an effective persistence mechanism — even if a defender kills the `notmalware.elf` process, it restarts automatically within 5 minutes without any further attacker interaction.

After adding the line, save and exit nano (`Ctrl+O` → `Enter` → `Ctrl+X`, or `Y` to confirm). The cron daemon picks up the change immediately.

---

## 5. Finding the Cron Entry as a Defender

Now switching to the **defender's perspective** — how do you find this malicious scheduled task?

### 5.1 Locating Stored User Crontabs

```bash
sudo ls -al /var/spool/cron/crontabs/
```

**What this does:** User crontab files (created via `crontab -e`) are stored as files inside `/var/spool/cron/crontabs/`, one file per user (e.g., `/var/spool/cron/crontabs/tcm`). This directory is readable only by root — hence the `sudo`. Listing it shows you which users on the system have active crontabs, which is itself meaningful: most regular users shouldn't have scheduled jobs, so any non-root user with a crontab file is worth investigating.

### 5.2 Reading a Specific User's Crontab

```bash
sudo crontab -u tcm -l
```

**What this does:**
- `-u tcm` — operate on the crontab belonging to user `tcm` (not the current user).
- `-l` — list (print) its contents without opening an editor.

Output:
```
*/5 * * * * /home/tcm/malware/notmalware.elf
```

This is the direct, clean way to read another user's scheduled tasks as root — exactly what you'd do during an incident response investigation to check whether an attacker established cron-based persistence on a compromised account.

> **Detection tip:** During triage, running `sudo crontab -u <username> -l` for every non-system user on the machine is a fast, targeted way to detect cron-based persistence without having to manually inspect raw spool files.

---

## 6. Full Investigation Checklist — Where to Look for Malicious Cron Jobs

| Location | Command | What to look for |
|----------|---------|-----------------|
| System crontab | `cat /etc/crontab` | Unexpected entries, unfamiliar commands |
| Cron directories | `ls -al /etc/cron.*` | Scripts with odd names, wrong ownership, recent mtime |
| Per-script hashing | `sha256sum /etc/cron.daily/*` | Unexpected hash changes vs. baseline |
| Package-managed jobs | `ls -al /etc/cron.d/` | Non-package entries dropped by an attacker |
| User crontab list | `sudo ls -al /var/spool/cron/crontabs/` | Users who shouldn't have crontabs |
| Read user crontab | `sudo crontab -u <user> -l` | Malicious entries on compromised accounts |

---

## 7. Summary — The Attacker's Playbook vs. the Defender's Response

```
ATTACKER                              DEFENDER
--------                              --------
Generate payload (msfvenom)     →     Detect running process (ps, top)
Execute payload (./notmalware)  →     Trace network connection (netstat, ss, lsof)
Add cron entry (crontab -e)     →     Audit crontabs (crontab -u <user> -l)
Delete binary from disk         →     Recover via /proc/<PID>/exe + lsof +L1
Payload re-executes every 5min  →     Hash + submit to VirusTotal, kill + remove entry
```

**Key takeaway:** Cron is native, trusted by the OS, and runs without any ongoing attacker interaction — making it one of the simplest and most reliable persistence mechanisms on Linux. As a defender, auditing crontabs should be one of the first steps in any Linux-based incident response investigation.

---

## 8. Additional Notes / Things Worth Adding to Your Study

- **`anacron`**: complements cron for systems that are not always on (e.g., laptops) — it runs missed daily/weekly/monthly jobs on the next boot rather than skipping them. Worth knowing its config file at `/etc/anacrontab`.
- **`systemd` timers**: on most modern Linux distros, systemd timers are increasingly replacing traditional cron for scheduled tasks. They offer better logging (visible via `journalctl`) and dependency handling — an attacker can also abuse these. Check `systemctl list-timers` as part of a full persistence audit.
- **`crontab.guru`**: use this throughout your studies to decode or build any cron expression instantly — it gives a plain-English explanation of what any expression means.
- **Correlation with previous labs**: the payload used here (`notmalware.elf`, PID 3610) is the same binary traced in the Linux Network Analysis and Linux Process Analysis labs — cron is simply the persistence layer that would keep relaunching it automatically if the process were killed.
