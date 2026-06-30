# Linux Process Analysis

---

## 1. The `/proc` Filesystem — Foundation of Process Analysis

```bash
ls /proc
```

**What this does:**
`/proc` is a *virtual* (pseudo) filesystem — it doesn't exist on disk, it's generated live by the kernel and represents the current state of the system in real time. Inside `/proc`, every running process has its own numbered directory (matching its PID), e.g. `/proc/3610`. This is the foundation almost every process-inspection tool (`ps`, `top`, `lsof`) actually reads from under the hood.

> **MITRE ATT&CK reference:** Investigating `/proc` is a core technique under T1057 (Process Discovery) from the defender's perspective — used here for legitimate forensic triage.

---

## 2. `ps` — Process Status

```bash
ps        # without sudo: only shows YOUR processes/session
sudo ps   # with sudo: needed to see other users' processes clearly
```

From there, several useful variations:

```bash
sudo ps -u        # user-oriented format
sudo ps -AFH      # all processes, full format, hierarchical (tree-like)
sudo ps -aux      # classic "BSD-style" all-users, all-processes view
sudo ps -AFH | less   # same as above, paged for easier reading
```

**Flag breakdown:**
- `-A` — select *all* processes (not just the current user/terminal).
- `-F` — "full" format: extra columns like PPID, CPU%, memory, start time, full command line.
- `-H` — display process hierarchy (children indented under parents).
- `-u` — user-oriented output (shows %CPU, %MEM, owner).
- `-aux` — a very common combo: `a` (all users), `u` (user-oriented), `x` (include processes without a controlling terminal — important for daemons/background processes, and for catching something like a reverse shell that detached from its terminal).
- `| less` — pipes the output into a pager so you can scroll through it instead of it flooding the terminal.

Scrolling through `sudo ps -AFH | less`, you spot the suspicious entry:
```
tcm   3610  2941  0  ...  ./notmalware.elf
```
Here `3610` is the PID, `2941` is the PPID (parent process ID) — meaning `notmalware.elf` was spawned *by* process 2941, which is your next pivot point.

### Drilling into a specific PID

```bash
sudo ps -p 3610
```
Output:
```
PID  TTY      STAT   TIME COMMAND
3610 pts/0    Sl     0:00 ./notmalware.elf
```
Shows the basic status of just that one process — its controlling terminal (`pts/0`), state (`Sl` = sleeping, multi-threaded), and the exact command used to launch it.

```bash
sudo ps -p 3610 -F
```
Same as above but with the **full** format — adds PPID, CPU/memory usage, and start time, giving a complete snapshot of that single process.

### Tracing the parent

```bash
sudo ps --ppid 2941
```
Lists all **child processes** of PID 2941 — useful to see if the parent spawned multiple suspicious children, not just the one you already found.

```bash
sudo ps -p 2941 -F
```
Inspects the parent process itself in full detail — in many shell/initial-access scenarios, this will turn out to be your shell (`bash`) or a terminal emulator, confirming that the malware was launched manually/interactively from that shell session rather than via some other persistence mechanism.

> **Detection tip:** Mapping PID → PPID chains is one of the most powerful manual triage techniques — it lets you reconstruct the **process execution chain** (what launched what), which is exactly what EDR "process tree" views do automatically.

---

## 3. `pstree` — Visualizing the Process Tree

```bash
pstree
```
Displays all processes on the system as an actual tree diagram (parent → children), much easier to read at a glance than scrolling through flat `ps` output.

```bash
pstree -p -s 3610
```
**Flag breakdown:**
- `-p` — show PIDs next to each process name.
- `-s 3610` — show only the tree **leading up to** PID 3610 (i.e., its full ancestor chain: which process started the shell, which started the next process, and so on up to `notmalware.elf`).

This instantly gives you the full lineage of the malicious process without manually chaining multiple `ps --ppid` lookups.

---

## 4. `top` — Real-Time Process Monitoring

```bash
top
sudo top
sudo top -u tcm -c
sudo top -u tcm -c -o TIME+
```

**Flag breakdown:**
- `top` (and `sudo top`) — live, auto-refreshing view of CPU/memory usage per process, sorted by resource consumption by default. Useful for spotting a process unexpectedly consuming CPU (e.g., a cryptominer or noisy reverse shell).
- `-u tcm` — filter to only show processes owned by user `tcm`.
- `-c` — show the full command line instead of just the process name (so you see `./notmalware.elf` instead of a generic name).
- `-o TIME+` — sort the output by cumulative CPU time (`TIME+`) instead of the default (instantaneous %CPU) — useful for finding long-running background processes that don't spike CPU but have quietly accumulated runtime.

### Generating noise for comparison

In a separate terminal:
```bash
for i in {1..10}; do bash & done
```
**What this does:** a simple `for` loop that spawns 10 background `bash` processes (`&` backgrounds each one). This is purely to populate the process list with extra "noise" so you can practice picking the real target out of a busier, more realistic list — simulating a system with many legitimate processes running alongside a hidden malicious one.

Switching back to the first terminal/`top` session, you'd see the new `bash` processes appear in real time.

---

## 5. Deep Dive via `/proc/<PID>`

```bash
cd /proc
ls
ls -al /proc/self
```

**What this does:**
- `/proc/self` is a special symlink that always points to the directory of *whichever process is currently reading it* — a quick way to see the `/proc` structure for your own current shell process without needing to know its PID.

```bash
cd 3610
ls
```
Inside a PID's directory you'll find numerous virtual files/symlinks, the most useful for analysis being:

| File | Purpose |
|------|---------|
| `cmdline` | The exact command-line arguments used to launch the process |
| `cwd` | Symlink to the process's current working directory |
| `environ` | The process's environment variables (raw, NUL-separated) |
| `exe` | Symlink to the actual binary file that was executed |
| `status` / `stat` | Detailed process state info |

```bash
cat cmdline
ls
```
`cmdline` shows you exactly what was typed/passed to launch the process — confirms it was run as `./notmalware.elf` with no extra arguments.

```bash
ls -al cwd
```
Shows the working directory the process was launched *from* — useful for locating related files (drop location, staging folder, etc.).

```bash
cat environ
cat environ | tr '\0' '\n'
```
**What this does:** `environ` stores environment variables separated by NUL bytes (`\0`) instead of newlines, so `cat`-ing it raw looks like a jumbled single line. Piping through `tr '\0' '\n'` converts those NUL separators into newlines, making the variable list human-readable (`PATH=...`, `HOME=...`, `USER=...`, etc.) — sometimes revealing how/where the process was launched (e.g., via SSH, cron, a specific shell session).

```bash
ls -al exe
```
Shows where `exe` (a symlink) points — i.e., the full path to the actual binary on disk that's backing this running process.

```bash
sha256sum exe
sha256sum /home/tcm/malware/notmalware.elf
```
Hashing via `/proc/<PID>/exe` and via the direct file path should produce the **same hash** — confirming the running process and the on-disk file are one and the same (this matters because, as shown later, the file can be deleted while the process keeps running off the deleted inode).

```bash
strings exe
```
*(Note: the command is `strings`, not `string` — extracts all printable text strings from the binary.)* This can reveal embedded IPs, URLs, function names, error messages, or other artifacts hinting at the payload's behavior — a quick, low-effort form of static analysis without needing a disassembler.

---

## 6. Surviving Deletion — The "Deleted but Running" Trick

```bash
cp /home/tcm/malware/notmalware.elf /home/tcm/malware/notmalware_bak.elf
```
Makes a backup copy of the binary — this becomes important in a moment, demonstrating a key Linux behavior: **a running process keeps a binary "alive" in memory via its open file handle even after the file is deleted from disk.**

```bash
ps -AF | grep notmalware
ps -AF | grep notmalware | grep -v grep
```
**What this does:** searches the full process list for anything matching "notmalware". Since `grep` itself shows up in the process list (matching its own search term), `grep -v grep` ("invert match") filters that noise out, leaving only the real result.

```bash
ps -AFH | grep notmalware | grep -v grep
sudo lsof -p 3610
```
Re-confirms the process and lists every file handle it has open — including its backing executable.

```bash
lsof +L1
```
**What this does:** `+L1` tells `lsof` to show only files that have a **link count of less than 1** — in other words, files that have been **deleted from disk but are still held open by a running process**. This is a classic forensic/anti-forensic indicator: an attacker (or analyst, here) deletes the original binary off disk to hide it from filesystem-based scans, while the process itself keeps running unaffected because Linux doesn't actually free the inode until the last open handle closes.

```bash
pwd
ls
ls -al exe
```
At this point, listing the directory shows the original `notmalware.elf` is gone, and `ls -al exe` (inside `/proc/3610/`) now shows something like:
```
lrwxrwxrwx 1 tcm tcm 0 ... exe -> /home/tcm/malware/notmalware.elf (deleted)
```
The `(deleted)` tag is the unmistakable sign that the running process's backing file no longer exists on disk — yet the process is fully functional, because the kernel kept the data in memory/the underlying inode alive via the open file descriptor.

> **MITRE ATT&CK reference:** T1070.004 (Indicator Removal: File Deletion) — attackers delete the dropped binary post-execution specifically to defeat disk-based AV/forensic scans while the malicious process keeps running in memory.

---

## 7. Restoring the Binary from the Backup

```bash
cd /home/tcm/malware
ls
```
At this point only the backup (`notmalware_bak.elf`) and unrelated files (e.g. the `.exe`) remain — the original `notmalware.elf` was deleted.

```bash
mv notmalware_bak.elf notmalware.exe
ls
```
Renames the backup copy to `notmalware.exe`. This demonstrates that, since the backup was made *before* deletion, you (the analyst) retain a clean copy of the malicious binary for further static analysis, hashing, or submission to VirusTotal — even though the attacker attempted to remove all on-disk traces.

> **Practical takeaway:** This is exactly why, during live incident response, it's good practice to capture (copy/hash) a suspicious binary as early as possible — once it's deleted from disk, your only recovery path is via `/proc/<PID>/exe` *while the process is still running*. If the process is killed first, that recovery window is gone.

---

## 8. Summary — Investigation Workflow

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `ps -AFH`, `ps -aux` | Enumerate all running processes, spot the suspicious one |
| 2 | `ps -p <PID> -F`, `ps --ppid <PPID>` | Drill into the process and trace its parent/children |
| 3 | `pstree -p -s <PID>` | Visualize the full ancestor chain |
| 4 | `top -u <user> -c` | Monitor live resource usage and confirm the process is active |
| 5 | `/proc/<PID>/{cmdline,cwd,environ,exe}` | Extract launch command, working dir, env vars, and binary path |
| 6 | `sha256sum`, `strings` | Fingerprint and statically inspect the binary |
| 7 | `lsof -p <PID>`, `lsof +L1` | Detect deleted-but-running binaries |
| 8 | Backup copy + `mv` | Recover a sample for further analysis despite on-disk deletion |

**Key takeaway:** `/proc` is the single source of truth for live process state on Linux — almost every other tool (`ps`, `top`, `lsof`) is effectively a friendlier interface over it. Knowing how to read `/proc/<PID>` directly lets you recover information (and even entire deleted binaries) that disk-based forensic tools would completely miss.

---

## 9. Additional Notes / Things Worth Adding to Your Study

- **Process states** (seen in `ps`/`top` output, e.g. `S`, `R`, `Z`, `D`): worth memorizing — particularly `Z` (zombie) and `D` (uninterruptible sleep), which are common indicators of process or I/O issues and occasionally relevant in malware analysis (e.g., orphaned zombie processes after a crashed C2 connection).
- **`/proc/<PID>/maps` and `/proc/<PID>/mem`**: not covered in these notes but worth knowing — these expose the process's memory mappings and raw memory itself, useful for advanced analysis (e.g., dumping an in-memory-only/fileless payload that was never written to disk at all).
- **`auditd`**: in a real production environment, file deletions and process executions like these would ideally be logged via the Linux Audit framework (`auditd`) for after-the-fact investigation, since live `/proc` inspection only works while the process is still running.
- **Correlation with Linux Network Analysis**: this topic pairs directly with the previous one — PID `3610` here is the same process identified via `netstat`/`ss`/`lsof -i` in the network analysis lab, showing how network and process analysis converge on the same artifact from two different angles.
