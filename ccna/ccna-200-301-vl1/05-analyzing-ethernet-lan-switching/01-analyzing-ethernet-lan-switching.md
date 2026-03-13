# Chapter 5 – Analyzing Ethernet LAN Switching

---

## 1. LAN Switching Concepts

A modern Ethernet LAN connects user devices and servers into switches, with switches connecting to each other. The network is typically divided into two areas:

- **Campus LAN** → end-user devices connect to LAN switches (left side)
- **Data Center LAN** → servers sit in a closed room, connected to the campus LAN

Each switch independently makes forwarding decisions to move Ethernet frames toward their destination. To forward traffic from a user device to a server and back, each switch performs the same kind of logic, independently from each other.

---

## 2. Overview of Switching Logic

The primary job of a LAN switch is to **forward Ethernet frames to the correct destination MAC address**.

> 🔑 **Key Topic:** Switches perform three main actions:

1. **Forwarding/Filtering** — Decide when to forward a frame or when to filter (not forward) it, based on the **destination MAC address**
2. **Learning** — Prepare to forward future frames by learning the **source MAC address** of each received frame
3. **Loop Prevention** — Cooperate with all switches to prevent endless looping of frames using **Spanning Tree Protocol (STP)**

The first action is the switch's primary job; the other two are overhead functions.

> 📌 **Note:** Throughout this book, the terms *switch port* and *switch interface* are synonymous.

---

## 3. Forwarding Known Unicast Frames

To decide whether to forward a frame, a switch uses a **MAC address table** (also called switching table, bridging table, or CAM table).

The switch compares the frame's **destination MAC address** to this table and decides:
- **Forward** → send the frame out the matching output port
- **Filter** → do not forward the frame (if destination is on the same port it arrived from)

### Example – Single Switch Forwarding (Figure 5-3)

```
Fred sends a frame destined for Barney (0200.2222.2222)

MAC Address Table:
0200.1111.1111  →  F0/1
0200.2222.2222  →  F0/2   ← match found!
0200.3333.3333  →  F0/3
0200.4444.4444  →  F0/4

Result:
→ Forward out F0/2 only
→ Filter (do not send) on F0/3 and F0/4
```

### Example – Two Switches Forwarding (Figures 5-4 and 5-5)

Each switch makes an **independent** forwarding decision based on its own MAC address table:

```
Fred sends frame to Wilma (0200.3333.3333)

SW1 MAC Table:              SW2 MAC Table:
0200.1111.1111 → F0/1      0200.1111.1111 → G0/2
0200.2222.2222 → F0/2      0200.2222.2222 → G0/2
0200.3333.3333 → G0/1  ←  0200.3333.3333 → F0/3  ←
0200.4444.4444 → G0/1      0200.4444.4444 → F0/4

SW1: frame enters F0/1 → forwards out G0/1
SW2: frame enters G0/2 → forwards out F0/3 → reaches Wilma ✓
```

> 📌 **Note:** The term *forward-versus-filter decision* emphasizes the switch's choice to forward the frame out some ports but not others.

Frames with a destination MAC address that is known to the switch are called **known unicast frames**.

---

## 4. Learning MAC Addresses

The networking staff does **not** need to manually type MAC table entries. Switches learn MAC addresses automatically by examining **incoming frames**.

### How learning works:

When a frame arrives at a switch port, the switch looks at the **source MAC address**:
- If the source MAC is **not yet** in the MAC table → add a new entry (MAC address + incoming interface)
- If the source MAC is **already** in the table → no action needed

### Example – Learning Process (Figure 5-6)

```
MAC Table starts EMPTY

Step 1: Fred sends a frame to Barney
  → Frame enters port F0/1
  → Source MAC: 0200.1111.1111
  → Switch adds: 0200.1111.1111 → F0/1

MAC Table after Step 1:
0200.1111.1111  F0/1

Step 2: Barney replies to Fred
  → Frame enters port F0/2
  → Source MAC: 0200.2222.2222
  → Switch adds: 0200.2222.2222 → F0/2

MAC Table after Step 2:
0200.1111.1111  F0/1
0200.2222.2222  F0/2
```

> Learning always occurs by looking at the **source MAC address** in the frame and recording the **incoming interface** as the associated port.

---

## 5. Flooding Unknown Unicast and Broadcast Frames

What happens when the switch receives a frame whose destination MAC is **not** in the MAC table?

### Unknown Unicast Frame
- The switch **floods** the frame — forwards a copy out **all interfaces except the one it arrived on**
- The destination device will likely reply, and the switch can then learn that device's MAC address
- This frame is called an **unknown unicast frame**

### Broadcast Frame
- Frames destined to `FFFF.FFFF.FFFF` (Ethernet broadcast address) are **always flooded**
- This delivers a copy to all devices in the LAN

### Example – Flooding (Figure 5-7)

```
Fred sends a frame, MAC table is empty

Step 1: Frame arrives on F0/1
Step 2: Switch floods copies out F0/2, F0/3, F0/4
        (all ports except the incoming F0/1)
```

---

## 6. Avoiding Loops Using Spanning Tree Protocol

> 🔑 **Key Topic:** The third primary feature of LAN switches is **loop prevention**, implemented by **Spanning Tree Protocol (STP)**.

### Why loops are dangerous

Without STP, flooded frames in networks with **redundant links** would loop forever:
- Each switch floods the frame to all other ports
- The frame loops around the switches indefinitely
- This continual flooding can completely **congest the LAN** to the point of making it unusable

### How STP works

STP causes each interface on a switch to settle into either:
- **Blocking state** → the interface cannot forward or receive data frames
- **Forwarding state** → the interface can send and receive data frames normally

By blocking a correct subset of interfaces, STP ensures that **only a single active logical path exists** between each pair of LANs, preventing loops while maintaining connectivity.

> 📌 STP is covered in depth in Chapter 9, "Spanning Tree Protocol Concepts."

---

## 7. LAN Switching Summary

> 🔑 **Key Topic:** Complete summary of switch forwarding logic:

**Step 1 — Forward frames based on destination MAC address:**
- **a.** If the destination MAC is a broadcast, multicast, or **unknown unicast** → the switch **floods** the frame
- **b.** If the destination MAC is a **known unicast** (found in MAC table):
  - **i.** If the outgoing interface is **different** from the incoming interface → **forward** the frame out the outgoing interface
  - **ii.** If the outgoing interface is the **same** as the incoming interface → **filter** (do not forward) the frame

**Step 2 — Learn MAC addresses based on source MAC address:**
- **a.** For each received frame, note the source MAC address and incoming interface ID
- **b.** If not yet in the MAC table → add an entry

**Step 3 — Use STP to prevent loops** by causing some interfaces to block, meaning they do not send or receive frames.

---

## 8. Verifying and Analyzing Ethernet Switching

A Cisco Catalyst switch comes from the factory **ready to switch frames** — just connect the power cable and Ethernet cables, and the switch starts forwarding frames immediately.

Default settings that make this work:
- All interfaces are **enabled by default**, ready to work once a cable is connected
- All interfaces are assigned to **VLAN 1**
- 10/100 and 10/100/1000 interfaces use **autonegotiation** by default
- MAC learning, forwarding, and flooding logic all work **by default**
- **STP is enabled by default**

---

## 9. Demonstrating MAC Learning

Use the `show mac address-table dynamic` command to see dynamically learned MAC addresses.

### Example 5-1 – show mac address-table dynamic

```
SW1# show mac address-table dynamic

          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0200.1111.1111    DYNAMIC     Fa0/1
   1    0200.2222.2222    DYNAMIC     Fa0/2
   1    0200.3333.3333    DYNAMIC     Fa0/3
   1    0200.4444.4444    DYNAMIC     Fa0/4
Total Mac Addresses for this criterion: 4

SW1#
```

**Reading the table:**
- **Vlan** → the VLAN the MAC address belongs to
- **Mac Address** → the learned MAC address
- **Type** → DYNAMIC means learned automatically; STATIC means manually configured
- **Ports** → the interface from which the frame with that source MAC arrived

> 📌 **Note about VLANs:** Switches forward Ethernet frames inside a VLAN. If a frame enters via a port in VLAN 1, the switch will forward or flood that frame only out ports also in VLAN 1. VLANs are covered in detail in Chapter 8.

---

## 10. Switch Interfaces

Use `show interfaces status` to check the status of all switch interfaces at once.

### Example 5-2 – show interfaces status

```
SW1# show interfaces status

Port    Name    Status      Vlan  Duplex  Speed    Type
Fa0/1           connected   1     a-full  a-100    10/100BaseTX
Fa0/2           connected   1     a-full  a-100    10/100BaseTX
Fa0/3           connected   1     a-full  a-100    10/100BaseTX
Fa0/4           connected   1     a-full  a-100    10/100BaseTX
Fa0/5           notconnect  1     auto    auto     10/100BaseTX
...
Gi0/1           notconnect  1     auto    auto     10/100/1000BaseTX
Gi0/2           notconnect  1     auto    auto     10/100/1000BaseTX
```

**Status column values:**
- `connected` → cable connected, port is functional
- `notconnect` → no cable installed (or other problem)

**Duplex/Speed with "a-" prefix** → value was set via **autonegotiation**

### Interface counters – Example 5-3

```
SW1# show interfaces f0/1 counters

Port      InOctets   InUcastPkts  InMcastPkts  InBcastPkts
Fa0/1     1223303    10264        107          18

Port      OutOctets  OutUcastPkts OutMcastPkts OutBcastPkts
Fa0/1     3235055    13886        22940        437
```

Lists statistics about incoming and outgoing unicast, multicast, and broadcast frames on a specific interface.

---

## 11. Finding Entries in the MAC Address Table

In real networks with hundreds of MAC addresses, you need to filter the MAC table output.

### Search by MAC address – Example 5-4

```
SW1# show mac address-table dynamic address 0200.1111.1111

          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0200.1111.1111    DYNAMIC     Fa0/1
Total Mac Addresses for this criterion: 1
```

### Search by interface – Example 5-5

```
SW1# show mac address-table dynamic interface fastEthernet 0/1

          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0200.1111.1111    DYNAMIC     Fa0/1
Total Mac Addresses for this criterion: 1
```

### Search by VLAN – Example 5-6

```
SW1# show mac address-table dynamic vlan 1
! → shows all 4 MAC addresses in VLAN 1

SW1# show mac address-table dynamic vlan 2
! → shows empty table (no devices in VLAN 2)
```

---

## 12. Managing the MAC Address Table (Aging, Clearing)

MAC addresses do **not** remain in the table indefinitely. The switch removes entries in three ways:

### 1. Aging
- Default aging time: **300 seconds (5 minutes)**
- Each entry has a timer that resets to 0 every time a frame with that source MAC is received
- If no frame arrives within 300 seconds → the entry is removed
- Check aging time with:
```
SW1# show mac address-table aging-time
Global Aging Time:  300
```

### 2. Table Full
- The MAC table uses **CAM (Content-Addressable Memory)** — a fast physical memory
- If the table fills up, the switch removes the **oldest entries** to make space
- Check table size with:
```
SW1# show mac address-table count

Mac Entries for Vlan 1:
Dynamic Address Count  :  4
Static Address Count   :  0
Total Mac Addresses    :  4

Total Mac Address Space Available:  7299
```

### 3. Manual Clear
Remove dynamic entries manually (useful in labs):

```
! Clear all dynamic entries
SW1# clear mac address-table dynamic

! Clear by VLAN
SW1# clear mac address-table dynamic vlan 1

! Clear by interface
SW1# clear mac address-table dynamic interface Fa0/1

! Clear by MAC address
SW1# clear mac address-table dynamic address 0200.1111.1111
```

> ⚠️ `clear` is an **enable mode** command (requires `#` prompt).

---

## 13. MAC Address Tables with Multiple Switches

Each switch builds and maintains its **own independent MAC address table**. Together, they forward frames so that frames eventually arrive at the correct destination.

### Example 5-8 – MAC Tables on Two Switches

```
SW1# show mac address-table dynamic
Vlan  Mac Address      Type     Ports
   1  0200.1111.1111   DYNAMIC  Fa0/1   ← Fred connected to SW1
   1  0200.2222.2222   DYNAMIC  Fa0/2   ← Barney connected to SW1
   1  0200.3333.3333   DYNAMIC  Gi0/1   ← Wilma reachable via SW2
   1  0200.4444.4444   DYNAMIC  Gi0/1   ← Betty reachable via SW2

SW2# show mac address-table dynamic
Vlan  Mac Address      Type     Ports
   1  0200.1111.1111   DYNAMIC  Gi0/2   ← Fred reachable via SW1
   1  0200.2222.2222   DYNAMIC  Gi0/2   ← Barney reachable via SW1
   1  0200.3333.3333   DYNAMIC  Fa0/3   ← Wilma connected to SW2
   1  0200.4444.4444   DYNAMIC  Fa0/4   ← Betty connected to SW2
```

Notice that SW1 lists Wilma and Betty pointing to G0/1 (the uplink to SW2), while SW2 lists Fred and Barney pointing to G0/2 (the uplink to SW1). Each switch only knows **its own perspective**.

---

## 14. Command References

### Table 5-4 – Chapter 5 EXEC Command Reference

| Command | Mode / Purpose / Description |
|---|---|
| `show mac address-table` | Shows all MAC table entries of all types |
| `show mac address-table dynamic` | Shows all dynamically learned MAC table entries |
| `show mac address-table dynamic vlan vlan-id` | Shows dynamically learned entries for a specific VLAN |
| `show mac address-table dynamic address mac-address` | Shows the dynamically learned entry for a specific MAC address |
| `show mac address-table dynamic interface interface-id` | Shows all dynamically learned entries associated with a specific interface |
| `show mac address-table count` | Shows the number of entries in the MAC table and the total remaining empty slots |
| `show mac address-table aging-time` | Shows the global and per-VLAN aging timeout for inactive MAC table entries |
| `show interfaces id counters` | Lists packet counters for the listed interface ID |
| `show interfaces status` | Lists one line per interface with basic status and operating info |
| `clear mac address-table dynamic [vlan vlan-number] [interface interface-id] [address mac-address]` | Clears dynamic MAC table entries: all, by VLAN, by interface, or by MAC address |

### Configuration Command

| Command | Purpose |
|---|---|
| `mac address-table aging-time time-in-seconds [vlan vlan-number]` | Sets the aging time for MAC table entries globally or per VLAN |

---
