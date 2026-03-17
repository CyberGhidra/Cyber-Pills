# Chapter 6 – Configuring Basic Switch Management

---

## 1. Securing the Switch CLI

By default:
- A user can connect to the console and reach **enable mode with no password required**
- **Telnet and SSH are disabled** by default — remote users cannot even see a login prompt

This chapter covers four login security methods:
1. Simple shared passwords (no username)
2. Local usernames and passwords
3. External AAA authentication servers
4. Secure Shell (SSH)

---

## 2. Securing User Mode and Enable Mode with Simple Passwords

The simplest method uses a **shared password** — everyone uses the same password, with no username.

Three passwords to configure:

| Password | Protects | Where configured |
|---|---|---|
| Console password | Access from physical console cable | `line console 0` |
| vty password | Access from Telnet/SSH | `line vty 0 15` |
| Enable password | Moving from user mode to enable mode | Global config |

### Figure 6-1 — Password flow:
```
Console cable  → Console password → User Mode → Enable password → Enable Mode
Telnet/SSH     → vty password    ↗
```

### Config Checklist

**Step 1** — Set the enable password:
```
Switch(config)# enable secret MyPassword
```

**Step 2** — Set the console password:
```
Switch(config)# line console 0
Switch(config-line)# password faith
Switch(config-line)# login
Switch(config-line)# exit
```

**Step 3** — Set the Telnet/SSH (vty) password:
```
Switch(config)# line vty 0 15
Switch(config-line)# password hope
Switch(config-line)# login
Switch(config-line)# transport input all
Switch(config-line)# exit
```

### Example 6-1 — Console Login and Movement to Enable Mode
```
Press RETURN to get started.

User Access Verification
Password: faith          ← console password
Switch> enable
Password: love           ← enable password
Switch#
```

> 📌 **Note:** Use `enable secret` rather than the older `enable password` — `enable secret` is more secure as it stores the password encrypted.

### Example 6-2 — Full Basic Password Configuration
```
Switch# configure terminal
Switch(config)# enable secret love

Switch(config)# line console 0
Switch(config-line)# password faith
Switch(config-line)# login
Switch(config-line)# exit

Switch(config)# line vty 0 15
Switch(config-line)# password hope
Switch(config-line)# login
Switch(config-line)# transport input all
Switch(config-line)# end
Switch#
```

### Example 6-3 — Resulting running-config (subset)
```
enable secret 5 $1$OwtI$A58c2XgqWyDNeDnv51mNR.   ← encrypted!

line con 0
 password faith
 login

line vty 0 4
 password hope
 login
 transport input all

line vty 5 15
 password hope
 login
 transport input all
```

> 📌 Note: `show running-config` separates the first 5 vty lines (0–4) from the rest (5–15) for historical reasons.

---

## 3. Securing User Mode Access with Local Usernames and Passwords

Instead of a single shared password, each user gets their **own username and password** stored locally on the switch.

### Key difference from simple passwords:
- `login` → asks only for a password (shared)
- `login local` → asks for **username AND password** (individual)

### Figure 6-3 — Local Username Configuration:
```
Console  → line console 0 → login local → checks local username DB
Telnet   → line vty 0 15  → login local → checks local username DB

Local username database:
  username wendell secret odom
  username chris secret youdda
```

### Config Checklist

**Step 1** — Create local usernames:
```
Switch(config)# username wendell secret odom
Switch(config)# username chris secret youdda
```

**Step 2** — Configure console to use local usernames:
```
Switch(config)# line console 0
Switch(config-line)# login local
Switch(config-line)# no password       ← optional cleanup
Switch(config-line)# exit
```

**Step 3** — Configure vty lines to use local usernames:
```
Switch(config)# line vty 0 15
Switch(config-line)# login local
Switch(config-line)# no password       ← optional cleanup
Switch(config-line)# transport input all
Switch(config-line)# exit
```

### Example 6-4 — Telnet Login with Local Username
```
SW2# telnet 10.9.9.19
Trying 10.9.9.19 ... Open

User Access Verification

Username: wendell
Password:
SW1> enable
Password:
SW1# configure terminal
SW1(config)# ^Z
SW1#
*Mar 1 02:00:56.229: %SYS-5-CONFIG_I: Configured from console by wendell on vty0 (10.9.9.19)
```

> 📌 When a user exits configuration mode, IOS generates a **log message** that identifies the username — useful for auditing who made changes.

---

## 4. Securing User Mode Access with External Authentication Servers (AAA)

Using local usernames requires configuring every switch separately. A better option for large networks is an external **AAA server**.

### How AAA works (Figure 6-4):
```
User (Telnet/SSH)
       ↓ 1. login: wendell/odom
      SW1
       ↓ 2. asks AAA server: is wendell/odom valid?
    AAA Server (RADIUS or TACACS+)
       ↓ 3. Approved!
      SW1
       ↓ 4. Command prompt shown to user
```

### Benefits of AAA:
- Centralized username/password management
- Users can change their own passwords
- Easy to revoke access when someone leaves
- Most production networks use AAA servers today

### Protocols used:
- **RADIUS** → open standard, widely supported
- **TACACS+** → Cisco proprietary, more granular control

---

## 5. Securing Remote Access with Secure Shell (SSH)

**Telnet** sends everything in **clear text** — passwords are visible to anyone capturing packets (man-in-the-middle attack).

**SSH** encrypts all traffic between client and switch — **always prefer SSH over Telnet** in production.

### SSH requires local username authentication
SSH cannot use simple shared passwords — it requires `login local` with usernames.

### SSH-Specific Configuration (Figure 6-5):

Three extra commands are needed on top of the local username config:

```
hostname sw1                        ← Step 1a: set hostname
ip domain name example.com          ← Step 1b: set domain name
crypto key generate rsa             ← Step 1c: generate RSA encryption keys
```

IOS creates the **FQDN** (Fully Qualified Domain Name) by combining hostname + domain name:
`sw1.example.com` → used as input to generate the RSA key.

### Example 6-5 — Full SSH Configuration
```
SW1# configure terminal

SW1(config)# hostname SW1
SW1(config)# ip domain name example.com
SW1(config)# crypto key generate rsa
The name for the keys will be: SW1.example.com

Choose the size of the key modulus [512 to 2048]:
How many bits in the modulus [1024]: 1024
Generating 1024 bit RSA keys...
[OK]

! Optional: force SSH version 2 only (recommended)
SW1(config)# ip ssh version 2

! Configure vty lines
SW1(config)# line vty 0 15
SW1(config-line)# login local
SW1(config-line)# transport input all
SW1(config-line)# exit

! Define local usernames
SW1(config)# username wendell secret odom
SW1(config)# username chris secret youdaman
SW1(config)# ^Z
SW1#
```

### transport input options:

| Command | Effect |
|---|---|
| `transport input all` | Allow both Telnet and SSH |
| `transport input ssh` | Allow SSH only (recommended in production) |
| `transport input telnet` | Allow Telnet only |
| `transport input none` | Allow neither |

> ⚠️ In production, use `transport input ssh` to disable Telnet completely.

### Example 6-6 — Verifying SSH Status
```
SW1# show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3

SW1# show ssh
Connection  Version  Mode  Encryption   Hmac      State           Username
0           2.0      IN    aes126-cbc   hmac-sha1 Session started wendell
0           2.0      OUT   aes126-cbc   hmac-sha1 Session started wendell
%No SSHv1 server connections running.
```

---

## 6. Enabling and Securing the WebUI

The WebUI (HTTP/HTTPS server) allows management via a web browser.

### Recommended configuration:
```
! Disable HTTP (not encrypted, port 80)
SW1(config)# no ip http server

! Enable HTTPS (encrypted, port 443)
SW1(config)# ip http secure-server

! Use local usernames for WebUI authentication
SW1(config)# ip http authentication local

! Create admin user with privilege level 15 (full access)
SW1(config)# username admin privilege 15 secret MyAdminPass
```

### Privilege levels:
- **Level 0** → User mode
- **Level 15** → Enable mode (full access)

> 📌 If you log into the WebUI with a `privilege 15` username, you get full access including CLI configuration mode. Without `privilege 15`, you can log in but cannot use advanced features.

---

## 7. Enabling IPv4 for Remote Access

To allow Telnet, SSH, or WebUI access, the switch needs an **IP address**. Without an IP address, remote management is impossible.

> 📌 The switch IP address has **nothing to do** with how it forwards Ethernet frames — it exists only for management traffic (control plane).

---

## 8. Host and Switch IP Settings (SVI)

A switch does not assign an IP to a physical port. Instead it uses a **virtual NIC** concept called:
- **SVI (Switch Virtual Interface)**
- Also called **VLAN interface**

### Concept (Figure 6-6):
```
Physical ports (Fa0/1, Fa0/2...) → forward Ethernet frames (Layer 2)

VLAN 1 interface (virtual NIC) → management IP address
  interface vlan 1
  ip address 192.168.1.8 255.255.255.0
```

By default, all physical ports are in **VLAN 1**, so the VLAN 1 interface can communicate through any physical port.

### Default gateway
The switch also needs a **default gateway** to communicate with hosts outside its local subnet — same concept as a PC needing a default gateway.

```
To reach hosts in same subnet    → send directly
To reach hosts in other subnets  → send to default gateway (router)
```

---

## 9. Configuring IPv4 on a Switch — Static

### Config Checklist

```
Step 1: Enter VLAN 1 interface mode
Step 2: Assign IP address and mask
Step 3: Enable the interface with no shutdown
Step 4: Set the default gateway
Step 5: (Optional) Set DNS server
```

### Example 6-7 — Static IP Configuration
```
Emma# configure terminal
Emma(config)# interface vlan 1
Emma(config-if)# ip address 192.168.1.200 255.255.255.0
Emma(config-if)# no shutdown

00:25:07: %LINK-3-UPDOWN: Interface Vlan1, changed state to up
00:25:08: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up

Emma(config-if)# exit
Emma(config)# ip default-gateway 192.168.1.1
```

> 📌 The `no shutdown` command is required — the VLAN interface is **administratively down** by default. Without it, the interface stays down and the switch cannot use the IP address.

---

## 10. Configuring IPv4 on a Switch — DHCP

The switch can also learn its IP settings automatically from a DHCP server.

### Config Checklist

```
Step 1: Enter VLAN 1 interface mode, enable with no shutdown
Step 2: Use ip address dhcp instead of a static address
```

### Example 6-8 — DHCP IP Configuration
```
Emma# configure terminal
Emma(config)# interface vlan 1
Emma(config-if)# ip address dhcp
Emma(config-if)# no shutdown
Emma(config-if)# ^Z
Emma#

00:38:20: %LINK-3-UPDOWN: Interface Vlan1, changed state to up
00:38:21: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
```

> 📌 The DHCP-learned IP address is **not stored** in the running-config file.

---

## 11. Verifying IPv4 on a Switch

### Example 6-9 — Verifying DHCP-Learned Information
```
Emma# show dhcp lease
Temp IP addr: 192.168.1.101    for peer on Interface: Vlan1
Temp sub net mask: 255.255.255.0
  DHCP Lease server: 192.168.1.1, state: 3 Bound
Temp default-gateway addr: 192.168.1.1

Emma# show interfaces vlan 1
Vlan1 is up, line protocol is up
  Hardware is EtherSVI, address is 0019.e86a.6fc0
  Internet address is 192.168.1.101/24
  MTU 1500 bytes...

Emma# show ip default-gateway
192.168.1.1
```

### Key verification commands:

| Command | What it shows |
|---|---|
| `show interfaces vlan 1` | Interface status + IP address + mask |
| `show dhcp lease` | DHCP-learned IP, mask, gateway |
| `show ip default-gateway` | The configured default gateway |
| `show running-config` | Static IP (if configured statically) |

> ⚠️ If you forget `no shutdown`, the VLAN 1 interface stays **administratively down** and the switch cannot be reached remotely!

---

## 12. Miscellaneous Settings Useful in the Lab

These three commands reduce frustration when working at the CLI in a lab environment:

### logging synchronous
IOS can interrupt your typing with syslog messages at any time. This command tells IOS to wait for a natural break before displaying them:
```
SW1(config)# line console 0
SW1(config-line)# logging synchronous
```

### exec-timeout
By default, IOS disconnects after **5 minutes of inactivity**. In a lab, set it to never time out:
```
SW1(config-line)# exec-timeout 0 0     ← 0 minutes, 0 seconds = never
```

### no ip domain-lookup
If you mistype a command, IOS tries to resolve it as a hostname via DNS — this can cause a ~1 minute wait. Disable it:
```
SW1(config)# no ip domain-lookup
```

### History buffer
```
SW1(config-line)# history size 20      ← store last 20 commands
SW1# terminal history size 20          ← for current session only
SW1# show history                      ← view history buffer
```

### Example 6-10 — Complete Lab Productivity Template
```
no ip domain-lookup
!
line console 0
 exec-timeout 0 0
 logging synchronous
 history size 20
!
line vty 0 15
 exec-timeout 0 0
 logging synchronous
 history size 20
```

---

## 13. Command References

### Table 6-5 — Login Security Commands

| Command | Mode / Purpose |
|---|---|
| `line console 0` | Changes context to console configuration mode |
| `line vty first-vty last-vty` | Changes context to vty configuration mode for the range listed |
| `login` | Console/vty mode — tells IOS to prompt for a password (no username) |
| `password pass-value` | Console/vty mode — sets the shared password |
| `login local` | Console/vty mode — tells IOS to prompt for username AND password, checked against local username DB |
| `username name secret pass-value` | Global — defines a local username and password for `login local` |

### Table 6-5 (continued) — SSH Configuration Commands

| Command | Mode / Purpose |
|---|---|
| `crypto key generate rsa [modulus 512..2048]` | Global — creates RSA keys stored in flash memory for SSH |
| `transport input [telnet \| ssh \| all \| none]` | vty line mode — defines which protocols are allowed |
| `ip domain name fqdn` | Global — sets the domain name (used to build FQDN for SSH keys) |
| `hostname name` | Global — sets the switch hostname (also used in command prompt) |
| `ip ssh version 2` | Global — forces SSH version 2 only |

### Table 6-6 — Switch IPv4 Configuration Commands

| Command | Mode / Purpose |
|---|---|
| `interface vlan number` | Global — enters VLAN interface mode |
| `ip address ip-address subnet-mask` | VLAN interface mode — statically sets IP address and mask |
| `ip address dhcp` | VLAN interface mode — configures switch as DHCP client |
| `ip default-gateway address` | Global — sets the default gateway IPv4 address |
| `ip name-server server-ip-1 server-ip-2...` | Global — sets DNS server IP addresses |

### Table 6-7 — Other Switch Configuration Commands

| Command | Mode / Purpose |
|---|---|
| `hostname name` | Global — sets the switch hostname |
| `enable secret pass-value` | Global — sets the enable mode password |
| `history size length` | Line config mode — sets number of commands in history buffer |
| `logging synchronous` | Console or vty mode — syncs syslog messages with show output |
| `[no] logging console` | Global — disables/enables display of log messages to console |
| `exec-timeout minutes [seconds]` | Console or vty mode — sets inactivity timeout |
| `no ip domain-lookup` | Global — disables DNS client on the switch |

### Table 6-8 — EXEC Command Reference

| Command | Purpose |
|---|---|
| `show running-config` | Lists the currently used configuration |
| `show running-config \| begin line vty` | Shows output starting from the first line containing "line vty" |
| `show dhcp lease` | Lists DHCP-learned IP address, mask, and default gateway |
| `show crypto key mypubkey rsa` | Lists the public RSA key created for SSH |
| `show ip ssh` | Lists SSH server status including version |
| `show ssh` | Lists status of current SSH connections in and out |
| `show interfaces vlan number` | Lists interface status, IP address, and mask |
| `show ip default-gateway` | Lists the switch's default gateway setting |
| `terminal history size x` | Changes history buffer size for current login session only |
| `show history` | Lists commands in the current history buffer |

---
