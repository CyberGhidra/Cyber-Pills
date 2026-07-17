# Introduction to FTK Imager

**Download page:**
- The direct download URL changes with each release. Navigate to [https://www.exterro.com/ftk-product-downloads/](https://www.exterro.com/ftk-product-downloads/) and look for the latest **"FTK Imager <version number>"** entry.

---

## What Is FTK Imager?

**FTK Imager** (Forensic Toolkit Imager) is a free forensic imaging and evidence preview tool developed by Exterro (formerly AccessData). It is one of the industry-standard tools for acquiring forensically sound copies of storage media — used by law enforcement, corporate IR teams, and digital forensics professionals worldwide.

**What FTK Imager does:**
- Creates **forensic images** (bit-for-bit copies) of physical drives, logical drives, folders, or individual files — in multiple formats (E01, AD1, RAW/DD)
- **Mounts forensic images** for read-only browsing without altering them
- **Previews evidence** on a live system — browsing the file system, viewing deleted files, and examining file contents — without modifying anything
- **Hashes evidence** (MD5, SHA-1, SHA-256) to verify integrity at acquisition time and at any subsequent point
- **Captures memory** (RAM) from a live Windows system

The core principle behind FTK Imager aligns directly with everything covered in Chain of Custody: it reads evidence without writing to it, produces a verified copy, and documents the hash at acquisition — making the resulting image forensically admissible.

---

## Installation

Download the installer from the Exterro page above. Installation is straightforward and follows a standard Windows setup wizard — accept the license agreement, choose the installation directory (default is fine for lab use), and complete the installation. No special configuration is required before first launch.

---

## Interface Tour

On first launch, FTK Imager presents a clean, toolbar-based interface divided into a few main panes. The tour below follows the structure of the menus and toolbar.

### File Menu

The **File** menu contains the primary forensic acquisition and evidence management functions:

| Option | What It Does |
|--------|-------------|
| **Image Mounting** | Mount an existing forensic image (E01, AD1, DD) as a read-only virtual drive — allows browsing the image contents as if it were a physical disk, without modifying it |
| **Create Disk Image** | The core acquisition function — creates a forensic image of a physical drive, logical drive, image file, folder, or individual files. Produces a verified, hashed forensic copy. |
| **Decrypt AD1 Image** | Decrypts an encrypted AD1 forensic image created by FTK Imager or FTK (the full forensic suite) |
| **Add Evidence Item** | Adds an existing evidence source (physical drive, logical drive, image file, folder) to the evidence tree for browsing and preview — without creating a new image |
| **Add All Attached Devices** | Automatically detects and adds all storage devices currently connected to the forensic workstation to the evidence tree |

### View Menu
Controls the layout and visibility of the interface panes — allows toggling the evidence tree, file list, properties panel, and hex viewer on or off to customize the workspace depending on the task.

### Mode Menu
Controls how file contents are displayed in the viewer pane:

| Mode | Description |
|------|-------------|
| **Automatic** | FTK Imager selects the most appropriate display format based on the file type (images render as images, text files render as text, etc.) |
| **Text** | Forces display as plain text — useful for examining the readable content of files that automatic mode might render differently |
| **Hex** | Displays the raw hexadecimal bytes of the file — essential for low-level analysis, examining file headers, identifying file types by magic bytes, or investigating obfuscated content |

### Toolbar
The toolbar provides quick-access icons for the most commonly used functions — these mirror the options found in the **File** menu (Create Disk Image, Add Evidence Item, etc.) and allow one-click access without navigating through menus.

### Custom Content Sources
Accessible from the toolbar, this panel allows building a custom evidence collection from specific paths rather than imaging an entire drive. The evidence selector works with:

```
Evidence: Filesystem | Path | File
```

This is useful when you only need to acquire a specific folder or set of files rather than an entire disk — for example, collecting only a user's AppData folder or a specific log directory, which dramatically reduces acquisition time and image size for targeted investigations.

---

## Hands-On: Creating a Virtual Hard Disk for Testing

Imaging a real 500 GB drive in a lab environment would take far too long. Instead, the lab creates a small **Virtual Hard Disk (VHD)** — a simulated disk stored as a file on the host system — to demonstrate FTK Imager's acquisition and recovery capabilities in a realistic but time-efficient way.

### Step 1 — Create the VHD in Disk Management

Open **Disk Management** (`diskmgmt.msc`):

```
Action → Create VHD
```

Configure the VHD with these settings:

| Setting | Value |
|---------|-------|
| **Location** | Desktop — save as `suspected.vhd` |
| **Size** | 500 MB |
| **Format** | VHD |
| **Type** | Fixed size |

Click **OK**. Disk Management will create the VHD file and attach it — it now appears in Disk Management as **Disk 1** (500 MB, unallocated).

### Step 2 — Initialize and Format the VHD

Right-click **Disk 1** → **Initialize Disk**:
- Select **MBR** (Master Boot Record) partitioning scheme → **OK**

Right-click the unallocated space → **New Simple Volume**:
- Leave all size settings at default (use the full 500 MB)
- Assign drive letter: **E**
- Format settings: leave defaults, but change the **Volume Label** to `suspect`
- Click **Next** → **Finish**

The `suspect` volume (drive E:) now appears in both Disk Management and Windows Explorer as an empty, formatted disk.

### Step 3 — Add Test Content

The `suspect` drive is currently empty. To make the demonstration meaningful, add some files to it — a few photos, documents, or any sample files. These will be the "evidence" FTK Imager will acquire and later recover even after deletion.

---

## Acquiring the VHD with FTK Imager

### Creating the Forensic Image

In FTK Imager:

```
File → Create Disk Image
```

When prompted for the source type, select **Physical Drive**. FTK Imager lists all physical drives currently attached to the system — select the VHD (Disk 1, 500 MB).

Configure the image destination:
- Choose the image format (**E01** is the most common forensic format — it supports compression, encryption, and built-in hashing)
- Set the output path and file name
- Set a case number, evidence number, examiner name, and notes — this metadata is embedded in the image and forms part of the forensic documentation
- Enable hash verification (MD5 and/or SHA-256) — FTK Imager will hash the source drive during acquisition and hash the resulting image, then compare them to confirm the image is a perfect copy

Click **Start**. FTK Imager reads the source drive sector by sector and writes the image, computing hashes throughout. At completion, it displays the source hash and the image hash — they must match.

### Browsing the Evidence Tree

Once the image is added as an evidence item (or during acquisition preview), FTK Imager displays the **evidence tree** in the left pane:

```
suspected.vhd [Physical Drive]
└── Partition 1 [suspect]  (E:)
    ├── [root]
    │   ├── photo1.jpg
    │   ├── document.docx
    │   └── ...
    └── [orphan files]
```

Clicking any item in the tree shows its contents in the file list pane, and clicking a file shows a preview (image, text, or hex depending on the selected mode). This is the **preview** functionality — examining evidence without modifying it.

---

## Demonstrating Deleted File Recovery

This is one of the most important demonstrations FTK Imager provides — and it directly illustrates why forensic imaging captures more than a standard file copy.

### The Scenario

1. On the `suspect` drive (E:), **delete one of the test files** (e.g., delete `photo1.jpg`) using Windows Explorer — move it to the Recycle Bin and then empty the Recycle Bin, so it is fully deleted from the file system's perspective.

2. Return to FTK Imager and **re-acquire the drive** (or simply preview the drive again with Add Evidence Item).

### What FTK Imager Shows

In the evidence tree, the deleted file **still appears** — typically marked with a red X or listed under a separate section:

```
└── Partition 1 [suspect]
    ├── [root]
    │   └── document.docx
    └── [Deleted Files]
        └── photo1.jpg  ← still visible, content intact
```

**Why this works:** when a file is deleted by the operating system, the OS does not immediately overwrite the data. It simply marks the file's directory entry as available and flags the disk sectors the file occupied as free space — available for future writes. The actual data remains physically present on the disk until those sectors are reused by new data.

FTK Imager reads every sector of the disk — including sectors marked as free — and recovers the file system metadata and data content from those sectors, making the deleted file visible and its contents accessible for analysis.

> **Forensic implication:** this is why forensic imaging captures far more than a standard file copy. A standard Windows Explorer copy only copies files the OS considers currently existing. A forensic image copies everything, including what the OS considers gone.

> **Attacker perspective:** simply deleting incriminating files is not an effective way to destroy digital evidence. Attackers who want to truly eliminate evidence need to use tools like `sdelete` (secure delete with multiple overwrites) or full disk encryption with key destruction — even then, forensic analysis of a live system or memory dump may still surface artifacts.

---

## Summary

| Feature | Purpose |
|---------|---------|
| **Create Disk Image** | Bit-for-bit forensic acquisition with hash verification |
| **Add Evidence Item** | Browse and preview evidence without imaging |
| **Image Mounting** | Read-only mount of existing forensic images |
| **Hex / Text / Auto modes** | Flexible evidence viewing at different levels of detail |
| **Custom Content Sources** | Targeted acquisition of specific paths or files |
| **Deleted file recovery** | Reads sectors marked free to surface deleted files |
| **Hash verification** | MD5/SHA-256 at acquisition — proves image integrity |
| **VHD testing approach** | Create a small virtual disk to practice acquisition without waiting hours |

**Key takeaway:** FTK Imager is the bridge between raw storage media and usable forensic evidence. It acquires without altering, verifies without trusting, and surfaces data that the operating system considers gone — making it an essential first step in any digital forensics investigation before analysis begins in tools like Autopsy or Volatility.
