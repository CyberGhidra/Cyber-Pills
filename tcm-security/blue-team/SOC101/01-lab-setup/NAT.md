# NAT Network Configuration (Isolated Lab Network)

This section explains how to configure a NAT Network in VirtualBox to allow virtual machines to communicate with each other while remaining isolated from the host’s local network.

---

## 1. Create a NAT Network

1. Open VirtualBox
2. Go to: **File → Tools → Network Manager**
3. Select the **NAT Networks** tab
4. Click **Create**

Configure the following:

- **Name:** "TCM" for example
- **IPv4 Prefix:** 192.168.1.0/24 for example
- Enable **DHCP**

Apply the changes.

---

## 2. Attach Virtual Machines to the NAT Network

For each virtual machine:

1. Open **Settings**
2. Go to **Network**
3. Adapter 1:
   - Enable **Network Adapter**
   - Attached to: **NAT Network**
   - Name: **TCM**
   - Check **Cable Connected**

Click OK.

---

## 3. Disable Additional Network Adapters

If other network adapters (Adapter 2, 3, etc.) are enabled:

- Disable them to ensure proper network isolation.

---

## Expected Result

- All VMs connected to the **TCM NAT Network** can communicate with each other.
- VMs have internet access (through NAT).
- The lab remains isolated from the host’s internal LAN.

---

## Verification

Inside a VM, run:

```powershell
ipconfig
