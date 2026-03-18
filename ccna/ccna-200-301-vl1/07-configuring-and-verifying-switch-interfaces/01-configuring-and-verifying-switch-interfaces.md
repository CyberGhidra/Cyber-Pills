# Chapter 7 – Configuring and Verifying Switch Interfaces

---

## 1. IEEE Autonegotiation Concepts

Ethernet NICs and switch ports often support **multiple speeds**:
- **10/100** → supports 10 and 100 Mbps
- **10/100/1000** → supports 10, 100, and 1000 Mbps

**Autonegotiation** allows devices on each end of a link to automatically agree on the **best speed and duplex** without manual configuration.

> 🔑 Autonegotiation prefers **full duplex** over half duplex when both devices support it.

---

## 2. Autonegotiation Under Working Conditions

Autonegotiation uses **Fast Link Pulses (FLPs)** — out-of-band electrical signals sent before any data transmission. FLPs declare:
- All supported speeds
- All supported duplex modes

### Rules when both sides use autonegotiation:
- Both devices send FLP messages declaring their capabilities
- Each device chooses the **fastest speed** supported by both
- Each device chooses **full duplex** if both support it

### Example (Figure 7-1) — Both sides autonegotiating:

```
PC1 (10 only)        → Switch port G1/0/1 → Result: 10 Mbps, Full duplex
PC2 (10/100)         → Switch port G1/0/2 → Result: 100 Mbps, Full duplex
PC3 (10/100/1000)    → Switch port G1/0/3 → Result: 1000 Mbps, Full duplex
```

---

## 3. Autonegotiation When Only One Side Uses It

When one device **disables autonegotiation**, it does not send FLP messages. The other device (still using autonegotiation) falls back to **parallel detection**:

> 🔑 **Key Topic — Parallel Detection Rules:**
> - **Speed:** Detect the neighboring device's speed by analyzing the incoming frames. Use that speed.
> - **Duplex:** Make a default choice based on speed:
>   - 10 or 100 Mbps → use **half duplex**
>   - 1000 Mbps or faster → use **full duplex**

> 📌 **Note:** Ethernet interfaces using speeds more than 1 Gbps always use full duplex.

### Example (Figure 7-2) — One side has autonegotiation disabled:

```
PC1: configured 100 Full  → Switch G1/0/1 detects 100 Mbps → uses 100 Half  ← DUPLEX MISMATCH!
PC2: configured 1000 Full → Switch G1/0/2 detects 1000 Mbps → uses 1000 Full ✓
PC3: configured 10 Half   → Switch G1/0/3 detects 10 Mbps → uses 10 Half ✓
```

> ⚠️ **Best practice:** Always use autonegotiation on **both** ends of a link. If you must disable it, configure the **same speed and duplex on both ends**.

---

## 4. Autonegotiation and LAN Hubs

Hubs do **not** participate in autonegotiation — they do not generate or forward FLP messages. Devices connected to a hub receive no FLPs and fall back to parallel detection, often resulting in **10 Mbps and half duplex**.

This is expected behavior since hubs require half duplex to avoid collisions anyway.

---

## 5. Configuring Speed and Duplex Manually

### Verifying default autonegotiation (Example 7-1)
```
SW1# show running-config
! Lines omitted for brevity
interface GigabitEthernet1/0/1
!
interface GigabitEthernet1/0/2
!
interface GigabitEthernet1/0/3
```
No `speed` or `duplex` commands shown = default autonegotiation is active.

### Confirming autonegotiation is default (Example 7-2)
```
SW1# configure terminal
SW1(config)# interface gigabitEthernet 1/0/1
SW1(config-if)# speed auto
SW1(config-if)# duplex auto
SW1(config-if)# ^Z

SW1# show running-config interface gigabitEthernet 1/0/1
!
interface GigabitEthernet1/0/1
end
```
> IOS does not display default settings in running-config — this confirms `speed auto` and `duplex auto` are the defaults.

### Reading show interfaces status output (Example 7-3):

| Prefix | Meaning |
|---|---|
| `a-full` | Full duplex, learned via **autonegotiation** |
| `a-1000` | 1000 Mbps, learned via **autonegotiation** |
| `auto` | Will use autonegotiation when link comes up |
| `full` (no "a-") | Full duplex, **manually configured** |
| `1000` (no "a-") | 1000 Mbps, **manually configured** |

### Setting speed and duplex manually (Example 7-5)
```
! On SW1
SW1# configure terminal
SW1(config)# interface g1/0/19
SW1(config-if)# speed 1000
SW1(config-if)# duplex full
SW1(config-if)# ^Z

! On SW2 (same settings on both ends!)
SW2(config)# interface g1/0/20
SW2(config-if)# speed 1000
SW2(config-if)# duplex full
```

> ⚠️ Always configure the **same values on both ends** of the link. Mismatched settings cause duplex mismatch problems.

### show interfaces status — manually configured vs autonegotiated (Example 7-6):
```
SW1# show interfaces g1/0/20 status

Port     Name  Status    Vlan  Duplex  Speed  Type
Gi1/0/19       connected 1     full    1000   10/100/1000BaseTX
```
No `a-` prefix → manually configured, not autonegotiated.

---

## 6. Using Auto-MDIX on Cisco Switches

Traditionally, connecting two switches required a **crossover cable**, while connecting a switch to a PC required a **straight-through cable**. Using the wrong cable caused the link to fail.

**Auto-MDIX** (Automatic Medium-Dependent Interface Crossover) solves this automatically:
- Detects if the wrong cable pinout is used
- Electronically swaps the wire pairs inside the interface
- The link works regardless of cable type

### Default behavior:
- Cisco Catalyst switches use **auto-MDIX by default** (`mdix auto`)
- The `mdix auto` command does not appear in running-config (it's a default)

### Example 7-7 — Auto-MDIX working (straight-through between two switches):
```
SW1# show running-config interface GigabitEthernet 1/0/19
!
interface GigabitEthernet1/0/19
end

SW1# show interfaces 1/0/19 status
Port      Status    Vlan  Duplex   Speed  Type
Gi1/0/19  connected 1     a-full   a-1000 10/100/1000BaseTX
```
Link works even with wrong cable — auto-MDIX fixed it automatically.

### Example 7-8 — Disabling Auto-MDIX causes link failure:
```
SW1(config)# int g1/0/19
SW1(config-if)# no mdix auto
! Interface immediately goes down!

SW1# show interfaces g1/0/19 status
Port      Status      Vlan  Duplex  Speed  Type
Gi1/0/19  notconnect  1     auto    auto   10/100/1000BaseTX
```

> 📌 To re-enable: `SW1(config-if)# mdix auto`

---

## 7. Managing Switch Interface Configuration

This section covers additional interface subcommands: `description`, `shutdown`, and removing configuration with `no`.

---

## 8. The description and interface range Commands

### description — add a label to an interface (Example 7-9):
```
SW1# configure terminal
SW1(config)# interface GigabitEthernet 1/0/1
SW1(config-if)# description Link to PC1, using autonegotiation
SW1(config-if)# ^Z

SW1# show interfaces g1/0/1 status
Port     Name                          Status    Vlan  Duplex  Speed
Gi1/0/1  Link to PC1, using            connected 1     a-full  a-1000

SW1# show interfaces g1/0/1
GigabitEthernet1/0/1 is up, line protocol is up (connected)
  Hardware is Gigabit Ethernet, address is 4488.165a.f201
  Description: Link to PC1, using autonegotiation
```

### interface range — configure multiple interfaces at once (Example 7-10):
```
SW1# configure terminal
SW1(config)# interface range g1/0/2 - 10
SW1(config-if-range)# description Interface not in use
SW1(config-if-range)# ^Z
```
IOS applies the subcommand to **every interface** in the range (G1/0/2 through G1/0/10).

> 📌 `interface range` does not appear in running-config — IOS expands it into individual interface entries.

### show interfaces description (Example 7-11):
```
SW1# show interfaces description
Interface   Status    Protocol  Description
Vl1         up        up
Gi0/0       admin down down
Gi1/0/1     up        up        Link to PC1, using autonegotiation
Gi1/0/2     down      down      Interface not in use
Gi1/0/3     down      down      Interface not in use
```

---

## 9. Administratively Controlling Interfaces with shutdown

| Command | Effect |
|---|---|
| `shutdown` | Administratively disables the interface |
| `no shutdown` | Re-enables the interface |

### Example 7-12 — Disabling an interface:
```
SW1# configure terminal
SW1(config)# interface GigabitEthernet 1/0/1
SW1(config-if)# shutdown

*Oct 6 16:33:14.911: %LINK-5-CHANGED: Interface GigabitEthernet1/0/1, changed state to administratively down
*Oct 6 16:33:15.911: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/1, changed state to down
```

### Example 7-13 — Status after shutdown:
```
SW1# show interfaces g1/0/1 status
Port     Name              Status    Vlan  Duplex  Speed
Gi1/0/1  Link to PC1       disabled  1     auto    auto

SW1# show interfaces g1/0/1
GigabitEthernet1/0/1 is administratively down, line protocol is down (disabled)
```

> 📌 `disabled` in `show interfaces status` = shutdown command is configured.
> `administratively down` in `show interfaces` = same thing.

---

## 10. Removing Configuration with the no Command

To revert interface settings to defaults, use the `no` version of each command:

| Command | What it removes |
|---|---|
| `no speed` | Reverts to `speed auto` |
| `no duplex` | Reverts to `duplex auto` |
| `no description` | Removes the description |
| `no shutdown` | Re-enables a shutdown interface |

### Example 7-15 — Removing all interface config:
```
! Before: interface has description, speed, duplex, shutdown
SW1# configure terminal
SW1(config)# interface gigabitethernet 1/0/21
SW1(config-if)# no speed
SW1(config-if)# no duplex
SW1(config-if)# no description
SW1(config-if)# no shutdown
SW1(config-if)# ^Z

SW1# show running-config interface g1/0/21
!
interface GigabitEthernet1/0/21
end
! All subcommands removed — back to defaults
```

### Shortcut — reset all at once:
```
SW1(config)# default interface g1/0/21
```
Resets **all** interface subcommands to their default values in one command.

> 📌 `show running-config` and `show startup-config` do NOT display default settings. Use `show running-config all` to see everything including defaults (produces ~10x more output).

---

## 11. Analyzing Switch Interface Status and Statistics

Two sets of status codes exist:

| Command | Status Format |
|---|---|
| `show interfaces` | **line status / protocol status** (e.g., up/up) |
| `show interfaces description` | **line status / protocol status** (e.g., up/up) |
| `show interfaces status` | **single word** (e.g., connected, notconnect, disabled) |

- **Line status** → Layer 1 (physical) is working
- **Protocol status** → Layer 2 is working

---

## 12. Interface Status Codes

> 🔑 **Key Topic — Table 7-2: LAN Switch Interface Status Codes**

| Line Status | Protocol Status | Interface Status | Typical Root Cause |
|---|---|---|---|
| administratively down | down | disabled | `shutdown` command is configured on the interface |
| down | down | notconnect | No cable; bad cable; wrong cable pinouts with MDIX disabled; speed mismatch; neighboring device is powered off, shutdown, or error disabled |
| up | down | notconnect | Not expected on LAN switch physical interfaces |
| down | down (err-disabled) | err-disabled | Port security (or other feature) has disabled the interface |
| up | up | connected | The interface is working normally ✓ |

### Common causes of notconnect state:
- No cable installed
- Cable damaged (e.g., crushed under carpet/chair)
- Wrong cable pinout with auto-MDIX disabled
- Speed mismatch between devices
- Neighboring device powered off or shut down
- Electromagnetic interference (EMI) from other equipment
- Fiber cable bent too sharply (macrobending)

---

## 13. The Duplex Mismatch Issue

A **duplex mismatch** occurs when:
- Both sides reach **up/up (connected)** state
- But one side uses **full duplex** and the other uses **half duplex**

The link appears to work but performs poorly with errors and retransmissions.

### How duplex mismatch happens (Figure 7-7):
```
SW2: interface G1/0/20 configured with speed 100, duplex full
     → autonegotiation DISABLED on SW2

SW1: interface G1/0/19 uses autonegotiation (default)
     → receives no FLP messages from SW2
     → parallel detection: senses 100 Mbps
     → default duplex for 100 Mbps = HALF DUPLEX

Result:
SW1 G1/0/19 → 100 Mbps HALF duplex
SW2 G1/0/20 → 100 Mbps FULL duplex
= DUPLEX MISMATCH!
```

### How to identify duplex mismatch:
- Interface is **up/up (connected)** — not obvious it's broken
- **Late collisions** counter incrementing on the half-duplex interface
- Use `show interfaces` and check the late collision counter

> 🔑 **Key Topic:** Late collisions on a half-duplex interface = classic duplex mismatch symptom.

### How to avoid duplex mismatch:
- Use **autonegotiation on both ends** (preferred)
- If disabling autonegotiation, configure **same speed and duplex on both ends**

---

## 14. Common Layer 1 Problems on Working Interfaces

Even on up/up (connected) interfaces, Layer 1 problems can cause errors. Use `show interfaces` to check counters.

### Example 7-16 — Interface counters for Layer 1 problems:
```
SW1# show interfaces gi1/0/1
  0 runts, 0 giants, 0 throttles
  0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
  0 output errors, 0 collisions, 3 interface resets
  0 babbles, 0 late collision, 0 deferred
```

> 🔑 **Key Topic — Counter definitions:**

| Counter | Description |
|---|---|
| **Runts** | Frames smaller than 64 bytes (min frame size). Collisions can cause runts. |
| **Giants** | Frames larger than 1518 bytes (max frame size). |
| **Input Errors** | Total of many counters: runts, giants, no buffer, CRC, frame, overrun, ignored |
| **CRC** | Frames that failed FCS math check — caused by collisions or cable interference |
| **Frame** | Frames with illegal format (e.g., ending with a partial byte) — caused by collisions |
| **Packets Output** | Total frames forwarded out the interface |
| **Output Errors** | Frames the switch tried to transmit but failed |
| **Collisions** | Total collisions when the interface was transmitting |
| **Late Collisions** | Collisions after the 64th byte — points to **duplex mismatch** |

### Diagnosing problems from counters:

| Symptom | Likely Cause |
|---|---|
| Increasing **late collisions** on half-duplex interface | Duplex mismatch |
| Increasing **CRC** errors, no collision increase | Cable interference (EMI) |
| Increasing **runts** and **CRC** | Collisions (check duplex settings) |

---

## 15. Command References

### Table 7-5 — Switch Interface Configuration Commands

| Command | Mode / Purpose |
|---|---|
| `interface type port-number` | Changes context to a specific interface mode |
| `interface range type port-number - end-port-number` | Changes context to a range of consecutively numbered interfaces |
| `shutdown` / `no shutdown` | Interface mode — disables or enables the interface |
| `speed {10 \| 100 \| 1000 \| auto}` | Interface mode — manually sets speed or enables autonegotiation |
| `duplex {auto \| full \| half}` | Interface mode — manually sets duplex or enables autonegotiation |
| `description text` | Interface mode — adds a text label to the interface |
| `no duplex` / `no speed` / `no description` | Reverts to default: `speed auto`, `duplex auto`, no description |
| `default interface interface-id` | Global mode — resets ALL interface subcommands to defaults |
| `[no] mdix auto` | Interface mode — enables (`mdix auto`) or disables (`no mdix auto`) auto-MDIX |

### Table 7-6 — Chapter 7 EXEC Command Reference

| Command | Purpose |
|---|---|
| `show running-config` | Lists current configuration, omitting most default settings |
| `show running-config interface type number` | Shows running-config excerpt for a specific interface only |
| `show running-config all` | Shows running-config including all default settings (~10x more output) |
| `show interfaces [type number] status` | Lists one line per interface with description, status, duplex, speed |
| `show interfaces [type number]` | Lists detailed status and statistical information (counters) |
| `show interfaces description` | Lists one line per interface with two-part status and description |

---

*Source: CCNA 200-301 Official Cert Guide, Volume 1 — Chapter 7 (pp. 161–182)*
