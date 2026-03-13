# Chapter 4 – Using the Command-Line Interface

---

## 1. Accessing the Cisco Catalyst Switch CLI

The **CLI (Command-Line Interface)** is a text-based interface in which the user types a command and presses Enter. The command is sent to the switch, which executes it and replies with text messages.

Cisco Catalyst switches also support other management methods:
- **Web interface** (browser)
- **Network management software**

This chapter focuses exclusively on using the **Cisco CLI** to monitor and configure switches.

---

## 2. Cisco Catalyst Switches

Cisco LAN switch families (as of 2023):

| Family | Description |
|---|---|
| **Cisco Catalyst** | Typical enterprise switches for use throughout an enterprise |
| **Cisco Nexus** | Designed for data centers; faster ports and high-speed port density |
| **Cisco Meraki** | Cloud-managed switches; simple onboarding via web portal |

> ⚠️ For the CCNA exam, the term *switch* always refers to **Cisco Catalyst switches**, not Nexus or Meraki.

### Switch Interface IDs

Switches identify physical connectors with an **interface type** and a **number**. Cisco Catalyst 9000 switches typically use three-digit identifiers (e.g., `GigabitEthernet 1/0/1`).

**Table 4-2 – LAN Switch Interface ID Examples with Abbreviations**

| Speeds Supported | Common Name | Example Switch Interface ID | Valid Abbreviations |
|---|---|---|---|
| 10 Mbps | Ethernet | Ethernet0/0 | E0/0, Et0/0, Eth0/0 |
| 10/100 Mbps | 10/100 | FastEthernet 0/1 | F0/1, Fa0/1 |
| 10/100/1000 Mbps | 10/100/1000 | GigabitEthernet 1/0/1 | G1/0/1, Gi1/0/1 |
| 1G/2.5G/5G/10G | Multigig | TenGigabit 1/0/2 | T1/0/2, Te1/0/2 |

---

## 3. The Operating System: IOS and IOS XE

### History of the OS in Catalyst Switches

- **CatOS** → first OS used by Catalyst switches (acquired with Crescendo Communications in 1993)
- **IOS** → Cisco migrated switches to IOS, the same OS used by routers, making management unified across all enterprise devices
- **IOS XE** → modern version of IOS with improved internal software architecture; maintains the same familiar CLI

> 📌 **Key Topic:** For CCNA purposes, almost everything about commands and the CLI applies to both IOS and IOS XE. The book uses the generic term *IOS* for both, unless a difference needs to be pointed out.

---

## 4. Accessing the CLI: Console, Telnet, SSH

Three main methods to access the CLI:

| Method | How it works |
|---|---|
| **Console** | Physical direct connection to the switch's console port |
| **Telnet** | Connection over a TCP/IP network (clear text — **not secure**) |
| **SSH** | Connection over a TCP/IP network with **encryption** (recommended) |

> 🔑 **Key Topic:** Telnet and SSH use the existing IP network; the console uses a dedicated physical cable.

---

## 5. Cabling the Console Connection

The console connection uses three main components:
1. The **console port** on the switch
2. A **serial or USB port** on the PC
3. A **cable** connecting the two

### Three cabling scenarios (Figure 4-4)

| Scenario | Switch Side | PC Side | Cable |
|---|---|---|---|
| **SW1** (old) | RJ-45 Console | Serial Port (DB-9) | Rollover Cable (RJ-45) |
| **SW2** (mixed) | RJ-45 Console | USB Port | Rollover Cable + USB converter |
| **SW3** (new) | USB Console (Mini-B) | USB Port | Standard USB cable |

> 💡 **Note:** When using USB connections, you may need to install a **software driver** so that the PC recognizes the Cisco device on the other end.

---

## 6. Configuring a Terminal Emulator

Once physically connected, the PC needs a **terminal emulator software** (e.g., PuTTY, SecureCRT).

### Default console port settings (8N1)

| Parameter | Value |
|---|---|
| Baud rate | **9600 bits/second** |
| Data bits | 8-bit ASCII |
| Parity | None |
| Stop bits | 1 |
| Flow control | None |

> 🔑 **Key Topic:** These parameters are collectively referred to as **8N1** (8 data bits, No parity, 1 stop bit).

---

## 7. Accessing the CLI with Telnet and SSH

- **Telnet**: transmits all data in clear text, including usernames and passwords → **significant security risk** in production networks
- **SSH**: encrypts all traffic contents → **recommended method** for production use

Both require the switch to be configured with an IP address and additional settings (covered in Chapter 6).

---

## 8. User and Enable (Privileged) Modes

All three access methods (Console, Telnet, SSH) place the user in **User EXEC Mode**.

### Difference between the two modes

| Mode | Prompt | Description |
|---|---|---|
| **User EXEC Mode** | `hostname>` | Allows the user to look around but not change configuration |
| **Enable (Privileged) Mode** | `hostname#` | Full access, including powerful commands like `reload` |

> 🔑 **Key Topic:** A prompt ending with `>` means user mode; a prompt ending with `#` means enable mode.

### Moving between modes

```
Switch> enable          → enters Enable Mode (may prompt for password)
Switch# disable         → returns to User Mode
Switch# reload          → reboots IOS (only available from Enable Mode)
```

### Example 4-1 – Privileged Mode Command Rejected in User Mode

```
Press RETURN to get started.

User Access Verification

Password:
Certskills1>
Certskills1> reload
Translating "reload"
% Unknown command or computer name, or unable to find computer address
Certskills1> enable
Password:
Certskills1#
Certskills1# reload

Proceed with reload? [confirm] y
00:08:42: %SYS-5-RELOAD: Reload requested by console. Reload Reason: Reload Command.
```

---

## 9. Password Security for CLI Access

By default, a Cisco switch requires no password for console access. It is good practice to configure passwords.

### Sample basic configuration (Example 4-2)

```
Certskills1# show running-config
!
hostname Certskills1
!
enable secret love          ← password to enter Enable Mode
!
line console 0
 login
 password faith             ← password for console access
```

- `enable secret <password>` → protects access to privileged mode
- `line console 0` → identifies the console port
- `login` → enables password checking
- `password <password>` → sets the console password

---

## 10. Accessing the CLI with the WebUI

Cisco also provides a **WebUI (Web User Interface)**, accessible via browser at the switch's IP address.

- Allows configuration and verification without knowing the CLI
- Includes an **Administration > Command Line Interface** page to use the CLI directly from the browser
- Engineers generally prefer SSH because CLI output alignment is much better

---

## 11. CLI Help Features

You do not need to memorize all IOS commands. The CLI provides built-in help tools:

**Table 4-3 – Cisco IOS Software Command Help**

| What You Enter | What Help You Get |
|---|---|
| `?` | Lists all commands available in the current mode |
| `command ?` | Lists all first parameter options for the command (space before `?`) |
| `com?` | Lists commands that start with `com` |
| `command parm?` | Lists parameters beginning with `parm` (no space before `?`) |
| `command parm<Tab>` | IOS completes the rest of the word (Tab key) |
| `command parm1 ?` | Lists all next parameters with a brief explanation of each |

### Key sequences for command edit and recall

**Table 4-4 – Key Sequences for Command Edit and Recall**

| Keyboard Command | What Happens |
|---|---|
| Up arrow or `Ctrl+P` | Displays the most recently used command; press again for older commands |
| Down arrow or `Ctrl+N` | Moves forward to more recently entered commands |
| Left arrow or `Ctrl+B` | Moves the cursor backward without deleting characters |
| Right arrow or `Ctrl+F` | Moves the cursor forward without deleting characters |
| `Backspace` | Moves the cursor backward, deleting characters |

> 💡 IOS stores by default the last **10 commands** in the history buffer.

---

## 12. The debug and show Commands

### `show` command
The **`show`** command is by far the most popular Cisco IOS command. It lists the currently known facts about the switch's operational status — like a **photograph** at one point in time.

Example:
```
Certskills1> show mac address-table dynamic
```
Displays the **MAC address table** the switch uses to make forwarding decisions.

### `debug` command
The **`debug`** command provides real-time information about the operation of the switch — like a **live video camera feed**. Once issued, IOS sends messages over time as events continue to occur. Most verification commands used throughout this book are `show` commands.

---

## 13. Configuring Cisco IOS Software

### Configuration Mode

**Configuration Mode** accepts *configuration commands* that change the active configuration. Changes take effect immediately each time you press Enter.

```
Switch# configure terminal    → enters Configuration Mode
Switch(config)#               → Configuration Mode prompt
Switch(config)# end           → exits Configuration Mode (or Ctrl+Z)
```

> ⚠️ Changes in Configuration Mode update the **running-config** (RAM) immediately with every Enter keystroke!

**Figure 4-11 – CLI Configuration Mode Versus EXEC Modes**

```
User Mode  ──enable──►  Enable Mode  ──configure terminal──►  Configuration Mode
           ◄─disable──               ◄──────────end / Ctrl+Z──────────────────────
```

---

## 14. Configuration Submodes and Contexts

Configuration Mode supports multiple **submodes**:

**Table 4-5 – Common Switch Configuration Modes**

| Prompt | Name of Mode | Context-Setting Command(s) to Reach This Mode |
|---|---|---|
| `hostname(config)#` | Global | None — first mode after `configure terminal` |
| `hostname(config-line)#` | Line | `line console 0` or `line vty 0 15` |
| `hostname(config-if)#` | Interface | `interface type number` |
| `hostname(config-vlan)#` | VLAN | `vlan number` |

### Example 4-4 – Navigating Between Different Configuration Modes

```
Switch# configure terminal
Switch(config)# hostname Fred
Fred(config)# line console 0
Fred(config-line)# password hope
Fred(config-line)# interface FastEthernet 0/1
Fred(config-if)# speed 100
Fred(config-if)# exit
Fred(config)#
```

- **Global commands** (e.g., `hostname`) apply to the entire switch
- **Subcommands** (e.g., `speed`) apply only to the selected interface or line

---

## 15. Storing Switch Configuration Files

### Types of memory in Cisco switches

| Memory | Contents | Notes |
|---|---|---|
| **RAM** (DRAM) | Running-config (active config) | Volatile: cleared on reload/power loss |
| **Flash** | Cisco IOS Software image | Permanent; can also store backup config files |
| **ROM** | Bootstrap program | Loads IOS at boot time |
| **NVRAM** | Startup-config | Permanent; used at boot time |

**Figure 4-13 – Cisco Switch Memory Types**

```
[ RAM ]              [ Flash ]         [ ROM ]           [ NVRAM ]
Working Memory       Cisco IOS         Bootstrap         Startup
& Running-Config     Software          Program           Configuration
```

### The two main configuration files

**Table 4-6 – Names and Purposes of the Two Main Cisco IOS Configuration Files**

| Configuration Filename | Purpose | Where It Is Stored |
|---|---|---|
| `startup-config` | Stores the initial configuration used anytime the switch reloads Cisco IOS | NVRAM |
| `running-config` | Stores the currently used configuration commands; changes dynamically in configuration mode | RAM |

> 🔑 **Key Topic:** Using configuration mode changes **only the running-config file**. If the switch loses power, all unsaved changes are lost!

---

## 16. Copying and Erasing Configuration Files

### Saving the configuration

To make changes permanent, copy the running-config to the startup-config:

```
Switch# copy running-config startup-config
```

This overwrites the `startup-config` file in NVRAM with the current `running-config`.

### Erasing the configuration

To start over with a clean configuration (useful in labs):

```
write erase
erase startup-config
erase nvram:
```

After erasing the startup-config, use `reload` to reboot the switch with an empty configuration.

> 📌 **Note:** Cisco IOS does not have a command that directly erases the running-config. To clear it, erase the startup-config and then `reload` the switch.

---

## 17. Command References

### Table 4-9 – Chapter 4 Configuration Commands

| Command | Mode and Purpose |
|---|---|
| `line console 0` | Global — changes context to console configuration mode |
| `login` | Line (console and vty) — tells IOS to prompt for a password (no username) |
| `password pass-value` | Line — sets the password required on that line for login |
| `interface type port-number` | Global — changes context to interface configuration mode (e.g., `interface FastEthernet 0/1`) |
| `speed value` | Interface subcommand — sets the Ethernet interface speed |
| `hostname name` | Global — sets the switch's hostname; also used as the first part of the command prompt |
| `exit` | Moves back to the next higher mode in configuration mode |
| `end` | Exits configuration mode and returns to enable mode from any configuration submode |
| `Ctrl+Z` | Two-key combination that does the same thing as the `end` command |

---

### Table 4-10 – Chapter 4 EXEC Command Reference

| Command | Purpose |
|---|---|
| `no debug all` / `undebug all` | Privileged mode EXEC — disables all currently enabled debugs |
| `reload` | Privileged mode EXEC — reboots the switch or router |
| `copy running-config startup-config` | Privileged mode EXEC — saves the active config, replacing the startup-config |
| `copy startup-config running-config` | Privileged mode EXEC — merges the startup-config with the currently active config in RAM |
| `show running-config` | Privileged mode EXEC — lists the contents of the running-config file |
| `write erase` / `erase startup-config` / `erase nvram:` | Privileged mode EXEC — erases the startup-config file |
| `quit` | EXEC — disconnects the user from the CLI session |
| `show startup-config` | Privileged mode EXEC — lists the contents of the startup-config file |
| `enable` | User mode EXEC — moves the user to enable (privileged) mode; prompts for a password if configured |
| `disable` | Privileged mode EXEC — moves the user from privileged mode to user mode |
| `configure terminal` | Privileged mode EXEC — moves the user into configuration mode |
| `show mac address-table` | EXEC — lists the contents of a switch forwarding (MAC) table |

---
