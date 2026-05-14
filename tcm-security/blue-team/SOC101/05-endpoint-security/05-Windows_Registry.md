# 🗂️ Windows Registry — Complete Guide

---

## What is the Windows Registry?

The **Windows Registry** is a hierarchical database that stores low-level settings for the operating system and for applications that opt to use it. Windows and many applications use the registry to store configuration data, user preferences, hardware information, and system state.

Think of it as the central nervous system of a Windows installation — nearly every installed program, hardware device, and OS feature has some footprint here.

---

## How to Access the Registry

### Using Run Dialog (most common)

1. Press **`Win + R`** to open the Run dialog
2. Type **`regedit`** or **`regedit.exe`**
3. Press **Enter** or click **OK**
4. If prompted by **UAC (User Account Control)**, click **Yes**

```
Win + R → regedit.exe → Enter
```

### Alternative Methods

| Method | Command / Path |
|--------|---------------|
| Start Menu search | Type `regedit` |
| Command Prompt / PowerShell | `regedit.exe` |
| Task Manager → File → Run | `regedit` |
| Direct path | `C:\Windows\regedit.exe` |

> ⚠️ **Administrator privileges** are required to modify most registry keys. Always run `regedit` as Administrator if you intend to make changes.

---

## Registry Files on Disk

The registry is not a single file — it is composed of multiple binary files called **hives**, stored primarily in:

```
C:\Windows\System32\config\
```

| File | Corresponding Hive |
|------|--------------------|
| `SAM` | `HKEY_LOCAL_MACHINE\SAM` |
| `SECURITY` | `HKEY_LOCAL_MACHINE\SECURITY` |
| `SOFTWARE` | `HKEY_LOCAL_MACHINE\SOFTWARE` |
| `SYSTEM` | `HKEY_LOCAL_MACHINE\SYSTEM` |
| `DEFAULT` | `HKEY_USERS\.DEFAULT` |

User-specific hives are stored in each user's profile:

```
C:\Users\<username>\NTUSER.DAT         → HKEY_CURRENT_USER
C:\Users\<username>\AppData\Local\Microsoft\Windows\UsrClass.dat
```

> These files are locked while Windows is running. They can only be read/edited offline (e.g., from a live USB) or via the registry editor itself.

---

## The Five Root Keys (Hives)

The registry is organized into five top-level keys, commonly called **hives**. Each one covers a different scope of the system.

---

### HKEY_CLASSES_ROOT

**Abbreviation:** `HKCR`

This hive maps **file extensions to applications** and defines **COM object** registrations. It controls what happens when you double-click a file — which application opens it and how the shell handles it.

**What it contains:**
- File extension associations (e.g., `.txt` → Notepad, `.pdf` → Acrobat)
- COM/OLE class registrations (CLSIDs)
- Shell verbs (Open, Edit, Print, etc.)
- ProgIDs (Programmatic Identifiers)

**Technical note:** `HKCR` is actually a **merged view** of:
- `HKEY_LOCAL_MACHINE\SOFTWARE\Classes` (system-wide)
- `HKEY_CURRENT_USER\SOFTWARE\Classes` (user-specific, takes precedence)

**Example path:**
```
HKEY_CLASSES_ROOT\.txt
HKEY_CLASSES_ROOT\txtfile\shell\open\command
```

---

### HKEY_CURRENT_USER

**Abbreviation:** `HKCU`

This hive stores **settings and preferences for the currently logged-in user**. Every user account has its own set of data here, loaded from their `NTUSER.DAT` file at login.

**What it contains:**
- Desktop and display preferences
- Installed application settings (per-user)
- Environment variables (user-scoped)
- Keyboard layouts and input settings
- Software preferences for browsers, editors, etc.
- Start menu and taskbar configuration

**Example paths:**
```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_CURRENT_USER\Control Panel\Desktop
HKEY_CURRENT_USER\Environment
```

> Changes here apply **only to the current user** and do not require administrator rights in most cases.

---

### HKEY_LOCAL_MACHINE

**Abbreviation:** `HKLM`

This is one of the most important hives. It contains **system-wide configuration** that applies to all users and to the machine itself, regardless of who is logged in.

**What it contains:**
- Hardware configuration and driver settings
- Installed software (system-wide)
- Security policies
- Services and their configurations
- Boot configuration
- Network settings

**Main subkeys:**

| Subkey | Purpose |
|--------|---------|
| `HARDWARE` | Dynamic hardware detection data (rebuilt at each boot) |
| `SAM` | Security Account Manager — local user accounts & groups (protected) |
| `SECURITY` | Local security policies (protected, accessible by SYSTEM only) |
| `SOFTWARE` | System-wide application settings and registrations |
| `SYSTEM` | System boot, services, drivers, device configuration |

**Example paths:**
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

> ⚠️ Editing `HKLM` **requires administrator privileges** and affects all users on the machine.

---

### HKEY_USERS

**Abbreviation:** `HKU`

This hive contains **registry data for all user profiles** that are loaded on the system. Each user's profile appears as a subkey identified by their **Security Identifier (SID)**.

**What it contains:**
- A `.DEFAULT` key (used for the default profile and services running before login)
- One subkey per loaded user, named by their SID (e.g., `S-1-5-21-...`)
- `HKEY_CURRENT_USER` is simply an alias that points to the current user's SID entry here

**Example paths:**
```
HKEY_USERS\.DEFAULT
HKEY_USERS\S-1-5-21-3623811015-3361044348-30300820-1013
```

> This hive is primarily used by system administrators when managing multiple user profiles, or in forensic/scripting scenarios.

---

### HKEY_CURRENT_CONFIG

**Abbreviation:** `HKCC`

This hive contains information about the **current hardware profile** in use by the system at boot time. It is a lightweight, dynamic hive.

**What it contains:**
- Current display settings (resolution, color depth)
- Current printer and font settings
- Hardware profile in use

**Technical note:** `HKCC` is an alias that maps to:
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Hardware Profiles\Current
```

It does **not** store persistent data on disk as a separate file — it is constructed at runtime.

---

## Keys vs Values

Understanding the difference between **Keys** and **Values** is fundamental to working with the registry.

### Keys

A **Key** is a **container** — analogous to a folder in a file system. Keys can contain:
- Other keys (called **subkeys**)
- One or more **values**

Keys form the hierarchical structure of the registry. A full key path looks like:

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion
```

Each segment separated by `\` is a key or subkey. The leftmost segment is always one of the five root hives.

**Properties of a Key:**
- Has a name
- Can have zero or more subkeys
- Can have zero or more values
- Has a `Last Written` timestamp (useful in forensics)

---

### Values

A **Value** is the **actual data entry** stored inside a key — analogous to a file inside a folder. Every value has three components:

| Component | Description |
|-----------|-------------|
| **Name** | The identifier of the value (e.g., `EnableLUA`) |
| **Type** | The data type (e.g., `REG_DWORD`, `REG_SZ`) |
| **Data** | The actual content stored |

Each key has a special **default value** (shown as `(Default)` in regedit) which may or may not be set.

---

**Summary — Key vs Value:**

```
KEY  (like a folder)
│
├── Subkey  (another folder inside)
│
└── Value Name │ Value Type │ Value Data
    ───────────────────────────────────
    "Wallpaper" │ REG_SZ     │ "C:\bg.jpg"
    "Count"     │ REG_DWORD  │ 0x00000003
```

---

## Value Types

The registry supports several data types for values. Each type defines how the data is stored and interpreted.

---

### String Value (`REG_SZ`)

The most common type. Stores a **fixed, plain Unicode text string**.

- **SZ** stands for *String Zero-terminated*
- Used for readable text: names, paths, descriptions
- Cannot contain environment variables that expand dynamically

**Example:**
```
Name:  "Wallpaper"
Type:  REG_SZ
Data:  "C:\Users\Manuel\Pictures\background.jpg"
```

**Use cases:** Application names, file paths, display strings, username settings.

---

### Expandable String Value (`REG_EXPAND_SZ`)

Similar to `REG_SZ`, but supports **environment variable references** that are expanded at runtime.

- Environment variables are written as `%VARIABLE_NAME%`
- Windows replaces them with their actual value when the data is read
- Useful for paths that should remain portable across users or system drives

**Example:**
```
Name:  "IconPath"
Type:  REG_EXPAND_SZ
Data:  "%SystemRoot%\System32\shell32.dll"
```

At runtime, `%SystemRoot%` expands to `C:\Windows` (or wherever Windows is installed).

**Use cases:** System paths, user profile references, portable application paths.

---

### Multi-String Value (`REG_MULTI_SZ`)

Stores **multiple strings** in a single value, separated by null characters (`\0`), with a double null terminator at the end.

- Appears in regedit as a list of lines
- Each line is an independent string entry
- Order is preserved

**Example:**
```
Name:  "DependOnService"
Type:  REG_MULTI_SZ
Data:  (line 1) "RpcSs"
       (line 2) "DCOM"
       (line 3) "PlugPlay"
```

**Use cases:** Lists of services, driver dependencies, allowed values, search paths.

---

### DWORD (`REG_DWORD`)

Stores a **32-bit unsigned integer** (Double Word). The value ranges from `0` to `4,294,967,295`.

- Displayed in regedit as hexadecimal (e.g., `0x00000001`) with the decimal equivalent in parentheses
- Commonly used as flags, switches, and numeric settings
- `0` typically means **disabled/false**; `1` typically means **enabled/true**

**Example:**
```
Name:  "EnableLUA"
Type:  REG_DWORD
Data:  0x00000001  (1)
```

**Byte order:** Stored in **little-endian** format by default (`REG_DWORD_LITTLE_ENDIAN`). A big-endian variant exists (`REG_DWORD_BIG_ENDIAN`) but is rarely used.

**Use cases:** Feature flags, timeout values, port numbers, boolean settings (0/1), version numbers.

---

### QWORD (`REG_QWORD`)

Stores a **64-bit unsigned integer** (Quad Word). The value ranges from `0` to `18,446,744,073,709,551,615`.

- Also displayed in hexadecimal in regedit
- Used when a 32-bit integer is not large enough
- Stored in **little-endian** format

**Example:**
```
Name:  "LastSuccessfulTime"
Type:  REG_QWORD
Data:  0x01DA9C7F4A3B2E10
```

**Use cases:** Timestamps (Windows FILETIME format), large counters, 64-bit numeric values.

---

### Binary Value (`REG_BINARY`)

Stores **raw binary data** of any length. Displayed in regedit as a hex dump (pairs of hex digits).

- No inherent structure — the consuming application defines the format
- Used for complex data structures that don't fit into other types
- Common for hardware configuration, security descriptors, and serialized data

**Example:**
```
Name:  "NetworkSettings"
Type:  REG_BINARY
Data:  28 00 00 00 03 00 00 00 F4 01 00 00 ...
```

**Use cases:** Hardware configuration blobs, encryption keys, serialized structs, icon data, calibration data.

---

### None (`REG_NONE`)

A special type with **no defined data type**. Essentially stores arbitrary binary data with no type semantics.

- Rarely used in practice
- Sometimes appears as a placeholder or in legacy/undocumented keys
- Treated as raw binary when read

---

### Value Types — Quick Reference

| Type Name | Registry Constant | Size | Description |
|-----------|------------------|------|-------------|
| String Value | `REG_SZ` | Variable | Plain Unicode string |
| Expandable String | `REG_EXPAND_SZ` | Variable | String with `%ENV_VAR%` support |
| Multi-String | `REG_MULTI_SZ` | Variable | List of null-separated strings |
| DWORD | `REG_DWORD` | 4 bytes | 32-bit integer |
| QWORD | `REG_QWORD` | 8 bytes | 64-bit integer |
| Binary | `REG_BINARY` | Variable | Raw binary data |
| None | `REG_NONE` | Variable | Untyped raw data |

---

## Common Operations

### Navigate to a Key Quickly

In `regedit`, press **`Ctrl + G`** (or use Edit → Go to…) and paste a full key path to jump directly to it.

### Export a Key (Backup)

Right-click any key → **Export** → save as a `.reg` file. This creates a text-based backup you can reimport.

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\MyApp]
"Version"="2.1.0"
"Enabled"=dword:00000001
```

### Import a `.reg` File

Double-click the `.reg` file, or use:
```
regedit /s myfile.reg
```
The `/s` flag runs silently without confirmation prompts.

### Delete a Key or Value

Right-click → **Delete**. Deletions are **immediate and permanent** — there is no undo in regedit.

---

## Registry via Command Line

### `reg` Command (built-in)

```cmd
:: Query a value
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ProductName

:: Add/modify a DWORD value
reg add "HKCU\Software\MyApp" /v "Enabled" /t REG_DWORD /d 1 /f

:: Delete a value
reg delete "HKCU\Software\MyApp" /v "OldSetting" /f

:: Export a key
reg export "HKCU\Software\MyApp" backup.reg

:: Import a .reg file
reg import backup.reg
```

### PowerShell

```powershell
# Read a value
Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion" -Name "ProductName"

# Create a new key
New-Item -Path "HKCU:\Software\MyApp" -Force

# Set a value
Set-ItemProperty -Path "HKCU:\Software\MyApp" -Name "Enabled" -Value 1 -Type DWord

# Delete a value
Remove-ItemProperty -Path "HKCU:\Software\MyApp" -Name "OldSetting"
```

> PowerShell uses **`HKLM:`** and **`HKCU:`** (with a colon) as the drive notation for registry paths.

---

## Safety & Best Practices

> ⚠️ **The Windows Registry is a critical system component. Incorrect edits can make Windows unstable or unbootable.**

### Always Back Up Before Editing

Export the key (or the entire registry) before making any changes:

- **Single key:** Right-click → Export
- **Entire registry:** File → Export → select "All" at the bottom of the dialog

### Create a System Restore Point

Before significant changes, create a restore point:
```
Control Panel → System → System Protection → Create
```

### Minimize Scope

Edit only the specific keys and values you need. Avoid bulk changes unless you know exactly what each one does.

### Understand What You're Changing

Never apply `.reg` files or registry "tweaks" from untrusted sources without reading and understanding their contents first.

### Use `HKCU` Over `HKLM` When Possible

Changes in `HKEY_CURRENT_USER` are scoped to a single user and are easier to revert. They also don't require administrator privileges in most cases.

### Test in a Virtual Machine

For experimental changes, test on a VM snapshot before applying to a production system.


