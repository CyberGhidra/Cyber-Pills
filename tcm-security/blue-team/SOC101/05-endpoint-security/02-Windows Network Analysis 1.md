# Windows Network Analysis 1

## 1. Starting Metasploit in Quiet Mode

First, start the Metasploit Framework using quiet mode:

```bash
msfconsole -q
```

The -q option starts Metasploit in quiet mode and suppresses the startup banner and most initial messages.
---

## 2. Configuring and Launching the Exploit

Configure the exploit in the same way as in the previous lab **Create Malware**.

After setting all the required parameters, run the exploit to deliver the payload to the target system.

---

## 3. Re-hosting the Exploit (if Needed)

If the exploit file is no longer available or the hosting service was stopped, restart the HTTP server on the attacker machine (Ubuntu):

```bash
python3 -m http.server 8000
```

This command creates a simple HTTP server on port **8000**, allowing the Windows machine to download the exploit again.

---

## 4. Verifying the Connection

Open the exploit from the Windows machine.

If everything works correctly, the Ubuntu terminal should show that **a connection has been established**.

---

## Network Analysis (Lab Activity)

## 5. Checking Shared Folders

On the Windows machine run:

```cmd
net view \\127.0.0.1
```

This command lists the **shared resources on the local machine** (using the loopback address).

At this stage **no shared folders should appear**.

---

## 6. Opening a Shell from the Meterpreter Session

Return to the Ubuntu terminal and run:

```bash
shell
```

This opens a Windows command shell from the active Meterpreter session.

You should see something similar to:

```cmd
C:\Users\tcm\Downloads>
```

This is the directory where the malware was originally downloaded.

---

## 7. Creating and Sharing a Folder

Create a new folder:

```cmd
mkdir prova
```

Then share it with:

```cmd
net share prova=C:\Users\tcm\Downloads\prova
```

This command creates a **network share named `prova`** pointing to the Downloads directory.

---

## 8. Verifying the Shared Folder

Return to Windows and run again:

```cmd
net view \\127.0.0.1
```

Now the output should display the shared folder:

```
prova
```

---

## 9. Checking Active Sessions

To see if there are active connections to the computer:

```cmd
net session
```

This command lists **all computers currently connected to shared resources on the system**.

---

## 10. Connecting to the Shared Folder

From the attacker system connect to the shared folder:

```cmd
net use X: \\127.0.0.1\prova
```

This maps the shared folder as drive **X:**.

---

## 11. Verifying the Session

Run again on Windows:

```cmd
net session
```

Example output:

```
Computer Name   Username   Time
127.0.0.1       tcm        <time>
```

This confirms that there is an active connection to the shared resource.

---

## Listing Network Drives

To view all mapped network drives:

```cmd
net use
```

This command shows **all network shares currently connected to the system**.

---

## Checking Active Network Connections

Use:

```cmd
netstat
```

or

```cmd
netstat -anob
```

`netstat -anob` displays:

- Active connections
- IP addresses
- Ports
- Protocols
- The executable associated with each connection

This command is useful for **detecting suspicious processes and network activity**.

---

## TCPView

TCPView is a graphical tool similar to:

```cmd
netstat -anob
```

However TCPView allows you to:

- View all active TCP and UDP connections
- Identify the processes using each connection
- Filter connections
- Close connections
- Terminate processes

This makes it a powerful tool for **real-time network monitoring and analysis**.

---

## Checking Windows Architecture

Before downloading tools like TCPView, check the system architecture:

```cmd
wmic os get osarchitecture
```

Example output:

```
64-bit
```

This tells you which version of the tool you should install (32-bit or 64-bit).
