# Fundamentals of Ethernet LANs

---

## 1. An Overview of LANs

**Ethernet** is a family of LAN standards that define the **physical and data-link layers** of the world's most popular wired LAN technology. The standards are defined by the **IEEE (Institute of Electrical and Electronics Engineers)** and cover cabling, connectors, protocol rules, and everything else required to build an Ethernet LAN.

### Typical SOHO LAN

A **SOHO (Small Office / Home Office)** LAN typically requires:

- An **Ethernet LAN switch** — provides multiple physical ports for connecting devices.
- **Ethernet cables** — any cable conforming to Ethernet standards.
- A **router** — connects the LAN to the WAN (e.g., the Internet).
- A **wireless access point (AP)** — allows wireless devices to join the LAN (defined by IEEE 802.11 standards, uses radio waves).

```
             [ Internet ]
                  │
               [Router]
                  │
   ┌──────────[Switch]──────────┐
   │           │      │         │
  [PC1]      [PC2]  [PC3]   [Printer]
```

> 💡 In most home networks today, the router, switch, and wireless AP are combined into a **single device** — commonly called a "wireless router."

### Typical Enterprise LAN

Enterprise LANs are larger and more structured:

- **LAN switches** are installed in wiring closets on each floor of a building.
- Ethernet cabling runs from the closet to cubicles and conference rooms.
- Each floor has its own switch, which connects to a **central distribution switch (SWD)**.
- Wireless APs are deployed throughout the building to support mobile devices.

```
3rd Floor:  [PC3] ── [SW3] ──┐
2nd Floor:  [PC2] ── [SW2] ──┤── [SWD] ── To Enterprise Network
1st Floor:  [PC1] ── [SW1] ──┘
```

---

## 2. Ethernet Physical Layer Standards

Ethernet supports many different physical standards, each defining speed, cable type, and maximum distance.

### IEEE Naming Convention

- Formal names start with **802.3** followed by suffix letters (e.g., `802.3u`).
- Informal names identify the speed and cable type:
  - Suffix **T** = UTP copper cable
  - Suffix **X** or **S/L/E** = fiber optic cable

### Common Ethernet Standards

| Speed | Common Name | Informal IEEE Name | Formal IEEE Name | Cable Type / Max Length |
|-------|-------------|-------------------|-----------------|------------------------|
| 10 Mbps | Ethernet | 10BASE-T | 802.3 | Copper UTP, 100 m |
| 100 Mbps | Fast Ethernet | 100BASE-T | 802.3u | Copper UTP, 100 m |
| 1000 Mbps | Gigabit Ethernet | 1000BASE-LX | 802.3z | Fiber, 5000 m |
| 1000 Mbps | Gigabit Ethernet | 1000BASE-T | 802.3ab | Copper UTP, 100 m |
| 10 Gbps | 10 Gig Ethernet | 10GBASE-T | 802.3an | Copper UTP, 100 m |

### Consistent Behavior Across All Links

Although Ethernet includes many physical layer standards, it uses the **same data-link layer standard** across all physical link types. No matter whether data flows over UTP copper or fiber, at any speed, the **Ethernet frame format (header + trailer) is always the same**.

```
PC1 ──(10 Mbps UTP)──► SW1 ──(1 Gbps Fiber)──► SW2 ──(10 Gbps Fiber)──► SW3 ──(100 Mbps UTP)──► PC3
[Eth | Data | Eth]  →  same frame format forwarded across all links  →  [Eth | Data | Eth]
```

---

## 3. Building Physical Ethernet LANs with UTP

### How UTP Transmits Data

UTP cables transmit data using **electricity flowing over copper wires**:

1. **Creating an electrical circuit** — two wires in a twisted pair form a complete loop.
2. **Encoding scheme** — the transmitter varies the electrical signal over time; the receiver interprets those changes as 0s and 1s.

```
Node 1 [Transmitter] ──wire 1──► [Receiver] Node 2
Node 1              ◄──wire 2──  Node 2
(Two wires = one complete electrical circuit, one direction)
```

### Why Twisted Pairs?

Electrical current creates **electromagnetic interference (EMI)**. Interference between wires in the same cable is called **crosstalk**. **Twisting the pairs together** helps cancel out most of the EMI.

### Breaking Down a UTP Ethernet Link

| Component | Description |
|-----------|-------------|
| **UTP Cable** | Holds copper wire pairs (2 pairs for 10/100 Mbps, 4 pairs for 1000 Mbps) |
| **RJ-45 Connector** | 8-pin connector at each end of the cable |
| **RJ-45 Port** | Socket on the device (NIC or switch) where the connector plugs in |

Each wire has a **color-coded plastic coating** to help identify pairs (e.g., blue + blue-white striped = one pair).

### Swappable Transceivers

Cisco switches support modular hardware ports called **transceivers**:

| Type | Description |
|------|-------------|
| **GBIC** | Original form factor for Gigabit interfaces; larger size |
| **SFP** | Smaller replacement for GBICs; used on Gigabit interfaces |
| **SFP+** | Same size as SFP, but used on **10-Gbps** interfaces |

---

## 4. UTP Cabling Pinouts

### 10BASE-T and 100BASE-T

These standards use **2 wire pairs** — one pair per direction:

- **PC NIC transmits on pins 1, 2** → Switch receives on pins 1, 2
- **Switch transmits on pins 3, 6** → PC NIC receives on pins 3, 6

### Straight-Through Cable

Connects pin 1 → pin 1, pin 2 → pin 2, etc. Used when the two devices transmit on **different pin pairs**.

```
End A                End B
Pin 1 ───────────── Pin 1
Pin 2 ───────────── Pin 2
Pin 3 ───────────── Pin 3
Pin 6 ───────────── Pin 6
```

### Crossover Cable

Crosses the pairs: pins 1,2 on one end → pins 3,6 on the other. Used when both devices transmit on the **same pin pairs**.

```
End A                End B
Pin 1 ───────────── Pin 3
Pin 2 ───────────── Pin 6
Pin 3 ───────────── Pin 1
Pin 6 ───────────── Pin 2
```

### Which Cable to Use?

| Connection | Cable Type |
|------------|-----------|
| PC ↔ Switch | Straight-through |
| PC ↔ Router | Straight-through |
| Switch ↔ Switch | Crossover |
| Switch ↔ Hub | Crossover |
| PC ↔ PC | Crossover |
| Router ↔ Router | Crossover |

**Rule:** same pin pairs on both ends → **Crossover** | different pin pairs → **Straight-through**

### Pin Pairs by Device Type (10/100BASE-T)

| Transmits on Pins 1, 2 | Transmits on Pins 3, 6 |
|------------------------|------------------------|
| PC NICs | Hubs |
| Routers | Switches |
| Wireless APs (Ethernet interface) | — |

### 1000BASE-T (Gigabit Ethernet)

Uses **all 4 wire pairs** and allows simultaneous transmit/receive on each pair. The crossover cable must cross **both** pair sets: A↔B (pins 1,2 ↔ 3,6) **and** C↔D (pins 4,5 ↔ 7,8).

---

## 5. Choosing the Right Cable

### Auto-MDIX

**Auto-MDIX** (*automatic medium-dependent interface crossover*) automatically detects wrong cable pinouts and electronically corrects them. Introduced with Gigabit Ethernet in 1998.

- If a straight-through cable is used where a crossover is needed, Auto-MDIX senses it and internally swaps the wire pairs.
- Most modern installations use **four-pair UTP cables** so all links support Gigabit Ethernet.

> 📝 **CCNA Exam Note:** Even though Auto-MDIX exists in practice, you are still expected to know the correct cable type for each connection for the exam.

---

## 6. Building Physical Ethernet LANs with Fiber

### Why Use Fiber?

UTP supports up to **100 meters** per link. When greater distances are needed — or in electrically noisy environments — **fiber-optic cabling** is the better choice.

### How Fiber Transmits Data

Fiber-optic cables use **glass** as the medium. Light passes through the glass fiber, varying over time to encode 0s and 1s. A **fiber-optic cable** holds a thin glass fiber in the middle, surrounded by cladding and protective layers.

**Components of a fiber-optic cable:**

```
[ Outer Jacket ][ Strengthener ][ Buffer ][ Cladding ][ Core ]
```

The **cladding** reflects light back into the **core**, keeping the light signal inside the cable as it travels.

### Multimode vs Single-Mode Fiber

| Feature | Multimode (MM) | Single-Mode (SM) |
|---------|---------------|-----------------|
| Core diameter | Larger | Much smaller (~1/5 of MM) |
| Light source | LED transmitter | Laser transmitter |
| How light travels | Multiple angles (modes) | Single straight angle |
| Max distance | Up to ~500 m (varies) | Up to tens of km |
| Cost | Less expensive | More expensive (SFP/SFP+ hardware) |
| Best use | Short distances, same building | Long distances, between buildings |

### Fiber Direction

Fiber is **unidirectional** — light travels one way per fiber. To support full two-way communication, **two fiber cables** are needed: one for each direction (Tx connects to Rx on the other end).

```
Device A [Tx] ──fiber 1──► [Rx] Device B
Device A [Rx] ◄──fiber 2── [Tx] Device B
```

### 10-Gbps Fiber Standards

| Standard | Cable Type | Max Distance |
|----------|-----------|--------------|
| 10GBASE-S | Multimode (MM) | 400 m |
| 10GBASE-LX4 | Multimode (MM) | 300 m |
| 10GBASE-LR | Single-Mode (SM) | 10 km |
| 10GBASE-E | Single-Mode (SM) | 30 km |

### UTP vs Fiber — Comparison

| Criteria | UTP | Multimode Fiber | Single-Mode Fiber |
|----------|-----|----------------|------------------|
| Relative cost | Low | Medium | High |
| Switch port cost | Low | Medium | High |
| Max distance | 100 m | 500 m | 40 km |
| EMI susceptibility | Some | None | None |
| Risk of signal interception | Some | None | None |

---

## 7. Sending Data in Ethernet Networks

### Ethernet Data-Link Protocols

All Ethernet physical standards share the **same data-link layer protocol**, defining a common **Ethernet frame** format: header + data + trailer.

### Ethernet Frame Format

```
┌──────────┬─────┬─────────────┬────────────┬──────┬──────────────┬─────┐
│ Preamble │ SFD │ Destination │   Source   │ Type │ Data and Pad │ FCS │
│    7     │  1  │      6      │     6      │  2   │   46–1500    │  4  │
└──────────┴─────┴─────────────┴────────────┴──────┴──────────────┴─────┘
         ◄────────────── Header ──────────────────►◄── Trailer ──►
                                                    (bytes)
```

| Field | Bytes | Description |
|-------|-------|-------------|
| **Preamble** | 7 | Synchronization |
| **SFD** (Start Frame Delimiter) | 1 | Signals that the next byte is the Destination MAC address |
| **Destination MAC Address** | 6 | Identifies the intended recipient |
| **Source MAC Address** | 6 | Identifies the sender |
| **Type** | 2 | Identifies the network layer protocol inside (e.g., IPv4 = `0x0800`, IPv6 = `0x86DD`) |
| **Data and Pad** | 46–1500 | The payload (L3 packet); minimum 46 bytes, maximum 1500 bytes |
| **FCS** (Frame Check Sequence) | 4 | Error detection |

> 📝 **MTU Note:** The maximum IP packet that can fit inside an Ethernet frame is **1500 bytes** — this is the Ethernet **Maximum Transmission Unit (MTU)**.

### EtherType Field

The **Type** field (also called *EtherType*) identifies what type of Layer 3 packet is inside the frame:

```
[ Eth Header | IPv4 Packet | Eth Trailer ]  →  Type = 0x0800
[ Eth Header | IPv6 Packet | Eth Trailer ]  →  Type = 0x86DD
```

This allows a host to send IPv4 frames and IPv6 frames on the same LAN — each frame carries a different EtherType value.

### Error Detection with FCS

The **Frame Check Sequence (FCS)** field in the Ethernet trailer allows the receiving node to detect whether any bits changed during transmission (due to interference or a bad NIC).

- The **sender** applies a math formula to the frame and stores the result in the FCS field.
- The **receiver** applies the same formula and compares its result with the FCS value.
- If they match → frame is good. If they don't → **frame is discarded**.

> ⚠️ Note: Ethernet performs **error detection**, not **error recovery**. TCP (at the transport layer) handles recovery by detecting the missing data and retransmitting it.

### Sending a Frame in a Modern Ethernet LAN (Full Duplex)

**Example: PC1 sends an Ethernet frame to PC2**

```
PC1 ──(10BASE-T, Full)──► SW1 ──(1000BASE-T, Full)──► SW2 ──(100BASE-T, Full)──► PC2
 ①                              ②                            ③                      ④
```

1. **PC1** builds the frame using its own MAC as source and PC2's MAC as destination, then sends it.
2. **SW1** receives the frame and forwards it out its G0/1 (Gigabit) interface toward SW2.
3. **SW2** receives the frame and forwards it out its F0/2 (Fast Ethernet) interface toward PC2.
4. **PC2** receives the frame, recognizes its own MAC address as the destination, and processes it.

Each link runs at a **different speed** — but the same Ethernet frame travels across all of them.

---

## 8. Ethernet Addressing — MAC Addresses

### What is a MAC Address?

**Ethernet addresses** (also called **MAC addresses** or *Media Access Control addresses*) are **6-byte (48-bit)** numbers used to identify each device on an Ethernet LAN.

- Displayed as **12 hexadecimal digits** (e.g., `00:60:2F:3A:07:BC`)
- Cisco devices may format them with periods: `0000.0C12.3456`

### MAC Address Structure

```
┌─────────────────────────────┬─────────────────────────────┐
│    OUI (24 bits / 6 hex)    │  Vendor Assigned (24 bits)  │
│        00 60 2F             │          3A 07 BC           │
└─────────────────────────────┴─────────────────────────────┘
```

- **OUI (Organizationally Unique Identifier)** — first 3 bytes, assigned by the IEEE to each manufacturer.
- **Vendor Assigned** — last 3 bytes, assigned uniquely by the manufacturer to each NIC.
- This guarantees every MAC address in the world is **globally unique**.

### MAC Address Terminology

| Term | Description |
|------|-------------|
| **MAC address** | Media Access Control. 802.3 (Ethernet) defines the MAC sublayer |
| Ethernet / NIC / LAN address | Other common names for a MAC address |
| **Burned-in address (BIA)** | Permanent MAC address stored in the NIC's ROM chip at manufacturing time |
| **Unicast address** | A MAC address that identifies a single LAN interface |
| **Broadcast address** | `FFFF.FFFF.FFFF` — delivered to **all** devices on the LAN |
| **Multicast address** | Delivered to a **subset** of devices on the LAN that subscribe to a specific multicast group |

---

## 9. Full Duplex and Half Duplex

### Definitions

| Mode | Behavior |
|------|----------|
| **Half Duplex** | Device must wait before sending if it is currently receiving. Cannot send and receive at the same time. |
| **Full Duplex** | Device can send and receive simultaneously. No waiting required. |

### Full Duplex — Modern Ethernet LANs

With all PCs and LAN **switches** (no hubs), every node can use **full duplex**. All nodes can send and receive at the same time on their port — no restrictions.

### Half Duplex — LAN Hubs

**LAN hubs** are older Layer 1 devices (no longer common) that work differently from switches:

- A hub **repeats every incoming electrical signal** out all other ports.
- It has no concept of MAC addresses or frames — it just broadcasts electricity.
- If **two devices transmit simultaneously**, their signals **collide** and become garbled.

To prevent collisions, devices connected to a hub must use **half-duplex logic** and the **CSMA/CD** algorithm:

**CSMA/CD (Carrier Sense Multiple Access with Collision Detection):**

| Step | Action |
|------|--------|
| 1 | Listen until the Ethernet is not busy |
| 2 | Begin sending the frame |
| 3 | If a collision is detected: send a **jamming signal**, wait a **random time**, then go back to Step 1 |

### Full vs Half Duplex — When to Use Which

```
[SW1] ──Full──► [SW2] ──Full──► [PC A]   ← switch-to-switch and switch-to-PC: Full duplex
                      ──Half──► [Hub] ──Half──► [PC C]  ← hub connections: Half duplex
```

- **Switch ↔ Switch** → Full duplex
- **Switch ↔ PC** → Full duplex
- **Switch ↔ Hub** → Half duplex (the switch port connected to the hub must use half duplex)
- **Hub ↔ PC** → Half duplex

### Ethernet Shared Media vs Point-to-Point

| Term | Meaning |
|------|---------|
| **Ethernet shared media** | Networks using hubs — all devices share bandwidth, require CSMA/CD |
| **Ethernet point-to-point** | Networks using switches — each link is independent, full duplex, no collisions |

---
