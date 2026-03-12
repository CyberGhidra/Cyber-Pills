# Fundamentals of WANs and IP Routing

---

## 1. Wide-Area Networks (WANs)

A **WAN (Wide-Area Network)** connects LANs that are geographically separated — for example, a branch office connecting to the company's home office over the Internet or a private WAN link.

WAN technologies define:
- **Physical (Layer 1)** standards for transmitting bits over long distances
- **Data-link (Layer 2)** protocols for framing data across the WAN link

The two main WAN technologies covered in this chapter are:

| WAN Type | Status | Notes |
|----------|--------|-------|
| **Leased-Line WAN** | Older, rarely used today | Has been part of CCNA since the beginning — still tested |
| **Ethernet WAN** | Common today | Uses Ethernet standards over longer distances |

---

## 2. Leased-Line WANs

### What is a Leased Line?

A **leased line** is a WAN service provided by a **telco (telephone company)**. It creates a dedicated point-to-point connection between two routers, one at each end of the link.

```
[Customer Site 1]──Router R1──(leased line, e.g. 1000 miles)──Router R2──[Customer Site 2]
        LAN                          WAN                              LAN
```

The telco runs cabling through its **Central Offices (COs)** underground to create the appearance of a direct cable between the two routers.

### Common Names for Leased Lines

| Name | Meaning |
|------|---------|
| Leased circuit / Circuit | Refers to the electrical circuit between two endpoints |
| Serial link / Serial line | Bits flow serially; routers use **serial interfaces** |
| Point-to-point link | Topology stretches between exactly two points |
| T1 | Specific leased line type — transmits at **1.544 Mbps** |
| WAN link / Link | Generic terms, no specific technology implied |
| Private line | Data sent cannot be copied by other telco customers |

### Data-Link Protocols for Leased Lines

A leased line provides a **Layer 1 service only** — it delivers bits, but does not define its own data-link protocol. The routers at each end must use one of two data-link protocols:

- **HDLC (High-Level Data Link Control)** — older, simpler
- **PPP (Point-to-Point Protocol)** — newer (1990s), more features

### HDLC Frame Format

```
┌──────┬─────────┬─────────┬──────┬──────────┬─────┐
│ Flag │ Address │ Control │ Type │   Data   │ FCS │
│  1   │    1    │    1    │  2   │ Variable │  2  │
└──────┴─────────┴─────────┴──────┴──────────┴─────┘
```

| HDLC/PPP Field | Ethernet Equivalent | Description |
|---------------|--------------------|-|
| Flag | Preamble, SFD | Recognizable bit pattern — signals a new frame is arriving |
| Address | Destination Address | Identifies the destination device (not very useful on point-to-point links) |
| Control | N/A | No longer used for links between routers |
| Type | Type | Identifies the type of Layer 3 packet inside the frame |
| FCS | FCS | Error detection |

> 💡 On a leased line, since there is only **one possible destination** (the router on the other end), the Address field is essentially unused — the destination is always implied.

### How Routers Use a WAN Data Link

The network layer focuses on forwarding **IP packets** end-to-end. The underlying LANs and WANs just move the packet to the next hop:

```
PC1 ──[802.3 frame]──► R1 ──[HDLC frame]──► R2 ──[802.3 frame]──► PC2
         LAN1                  WAN                    LAN2
```

**Step-by-step (PC1 → PC2):**

1. PC1 encapsulates the IP packet in an **Ethernet (802.3) frame** with R1's MAC as destination → sends it on LAN1.
2. R1 **de-encapsulates** the Ethernet frame, extracts the IP packet, then **re-encapsulates** it in an **HDLC frame** → forwards it over the leased line to R2.
3. R2 **de-encapsulates** the HDLC frame, extracts the IP packet, then **re-encapsulates** it in a new **Ethernet frame** with PC2's MAC as destination → delivers it on LAN2.

> 🔑 **Key point:** The IP packet stays the same throughout the journey. Only the **data-link frame changes** at each hop.

---

## 3. Ethernet WANs

### Why Ethernet as a WAN Technology?

Originally, Ethernet was only suitable for LANs (short distances). Over time, the IEEE improved Ethernet cabling distances significantly:
- **1000BASE-LX** supports up to **5 km**
- **1000BASE-ZX** supports up to **70 km**

Today, many WAN service providers (SPs) offer **Ethernet WAN** services using fiber optic Ethernet standards.

### How Ethernet WAN Works

The customer's router connects via a **fiber Ethernet access link** to the SP's nearest **Point of Presence (PoP)**. The SP then uses its internal network to connect the two customer sites.

```
[Customer Site R1]──Fiber Ethernet──[SP1 PoP]──SP Network──[SP2 PoP]──Fiber Ethernet──[Customer Site R2]
        CPE                                                                                    CPE
```

### Common Names for Ethernet WAN Services

| Name | Description |
|------|-------------|
| **Ethernet WAN** | Generic name to differentiate from Ethernet LAN |
| **Ethernet point-to-point link** | Emphasizes the two-endpoint topology |
| **Ethernet Line Service (E-Line)** | Term from the Metro Ethernet Forum (MEF) |
| **Ethernet over MPLS (EoMPLS)** | Uses MPLS technology inside the SP network |

### Routing Over an Ethernet WAN Link

When routers use an Ethernet WAN link, they use **standard 802.3 Ethernet headers and trailers** — just like on a LAN. However, the data-link header/trailer changes at each hop:

```
PC1 ──[802.3: dst=R1]──► R1 ──[802.3: src=R1 G0/1, dst=R2 G0/0]──► R2 ──[802.3: dst=PC2]──► PC2
         LAN1                           Ethernet WAN                            LAN2
```

---

## 4. IP Routing

### What is IP Routing?

**IP (Internet Protocol)** focuses on routing data (in the form of **IP packets**) from the source host to the destination host. IP does not concern itself with the physical transmission details — it relies on the lower layers (data-link and physical) for that.

### Network Layer Routing (Forwarding) Logic

Both **hosts** and **routers** participate in IP routing:
- **Hosts** decide whether the destination is local (same LAN) or remote (needs to go to a router).
- **Routers** use their **routing table** to forward packets hop-by-hop toward the destination.

**Example: PC1 (150.150.1.10) sends a packet to PC2 (150.150.4.10)**

```
PC1 (150.150.1.10)
  │  "Destination is in another group — send to nearby router (R1)"
  ▼
Router R1
  │  Routing table: 150.150.4.0 → send to R2 (via Serial0)
  ▼
Router R2
  │  Routing table: 150.150.4.0 → send to R3 (via FastEth0/0)
  ▼
Router R3
  │  Routing table: 150.150.4.0 → send directly (Gigabit0/0, N/A)
  ▼
PC2 (150.150.4.10) ✅
```

### Four-Step Router Forwarding Process

When a frame arrives at a router interface, the router follows these steps:

| Step | Action |
|------|--------|
| **1** | Check the **FCS** field — if errors exist, discard the frame |
| **2** | Discard the old **data-link header and trailer**, leaving only the IP packet |
| **3** | Compare the IP packet's **destination IP address** to the routing table, find the best matching route |
| **4** | **Encapsulate** the IP packet in a new data-link frame appropriate for the outgoing interface, and forward it |

### Host Forwarding Logic

A host (PC) uses simpler logic:
- If the destination IP is **on the same subnet** → send the frame directly to that host.
- If the destination IP is **on a different subnet** → send the frame to the **default router** (also called the **default gateway**).

### Routing Table Example

```
R1 Routing Table:
┌─────────────┬───────────┬─────────────┐
│   Subnet    │ Interface │  Next Hop   │
├─────────────┼───────────┼─────────────┤
│ 150.150.4.0 │  Serial0  │ 150.150.2.7 │
└─────────────┴───────────┴─────────────┘

R2 Routing Table:
┌─────────────┬─────────────┬─────────────┐
│   Subnet    │  Interface  │  Next Hop   │
├─────────────┼─────────────┼─────────────┤
│ 150.150.4.0 │ FastEth0/0  │ 150.150.3.1 │
└─────────────┴─────────────┴─────────────┘

R3 Routing Table:
┌─────────────┬────────────┬──────────┐
│   Subnet    │ Interface  │ Next Hop │
├─────────────┼────────────┼──────────┤
│ 150.150.4.0 │ Gigabit0/0 │   N/A    │
└─────────────┴────────────┴──────────┘
```

---

## 5. How Network Layer Routing Uses LANs and WANs

### IP Packet vs Data-Link Frame

The **IP packet** travels end-to-end, unchanged. The **data-link frame** only covers one hop at a time and is rebuilt at each router.

```
Hop 1 (LAN):  [Eth Header | IP Packet | Eth Trailer]   PC1 → R1
Hop 2 (WAN):  [HDLC Header | IP Packet | HDLC Trailer]  R1 → R2
Hop 3 (LAN):  [Eth Header | IP Packet | Eth Trailer]   R2 → PC2
```

| What changes at each hop? | What stays the same? |
|--------------------------|---------------------|
| Data-link header and trailer | IP packet (source and destination IP addresses) |
| Source and destination MAC addresses | IP header |
| Data-link protocol (Ethernet, HDLC, PPP) | Payload data |

### IPv4 Header

The IPv4 header is **20 bytes** wide and contains (among others):

```
┌─────────┬────────┬──────────┬────────────────┐
│ Version │ Length │ DS Field │ Packet Length  │
├─────────┴────────┼──────────┴────────────────┤
│  Identification  │  Flags  │ Fragment Offset │
├──────────────────┼──────────┬────────────────┤
│   Time to Live   │ Protocol │ Header Checksum│
├──────────────────┴──────────┴────────────────┤
│              Source IP Address               │
├──────────────────────────────────────────────┤
│           Destination IP Address             │
└──────────────────────────────────────────────┘
```

The most important fields for routing are the **Source IP Address** and **Destination IP Address**.

---

## 6. IP Addressing and Subnets

### IP Networks and Subnets

TCP/IP groups IP addresses so that all addresses on the same physical network belong to the same **IP network** or **IP subnet** — similar to how a ZIP code groups nearby postal addresses.

**Two foundational rules of subnetting:**

> - Two IP addresses **not separated by a router** must be in the **same subnet**.
> - Two IP addresses **separated by at least one router** must be in **different subnets**.

### Example — Address Grouping

| Location | Address Range |
|----------|--------------|
| Top Ethernet (LAN1) | Starts with 150.150.1._ |
| R1–R2 Serial link | Starts with 150.150.2._ |
| R2–R3 Ethernet WAN | Starts with 150.150.3._ |
| Bottom Ethernet (LAN2) | Starts with 150.150.4._ |

By grouping addresses this way, each router only needs **one routing table entry per subnet** — not one entry per individual IP address. This makes routing tables much smaller and more efficient.

---

## 7. How IP Routing Protocols Work

### Why Routing Protocols?

Routers need to know routes to all subnets in the network. The best way is to configure all routers with the same **IP routing protocol** (e.g., OSPF). The routers then automatically share routing information with each other.

### Three-Step Process for Learning Routes

| Step | Action |
|------|--------|
| **1** | Each router adds a route to its routing table for each **directly connected subnet** (automatically, no protocol needed) |
| **2** | Each router's routing protocol **advertises its known routes** to neighboring routers |
| **3** | Each router **listens** to routing updates from neighbors and adds those routes to its routing table |

### Example — How R1 Learns About Subnet 150.150.4.0

```
Step A: Subnet 150.150.4.0 exists, connected to Router R3
Step B: R3 adds it to its own routing table automatically
Step C: R3 sends a routing update (advertisement) to R2
Step D: R2 adds the route to its routing table
Step E: R2 sends a routing update to R1
Step F: R1 adds the route → Interface: Serial0, Next Hop: 150.150.2.7
```

> 📝 When multiple routes to the same subnet exist, routers pick the best one based on a value called **metric** and place it in the routing table.

---

## 8. Other Network Layer Features (DNS, ARP, Ping)

### DNS — Domain Name System

Users type **hostnames** (e.g., `www.google.com`) instead of IP addresses. **DNS** resolves hostnames to IP addresses automatically.

**How DNS works:**

```
PC11 ──①── "What is the IP of Server1?" ──────────────► DNS Server
PC11 ◄──②── "Server1 = 10.1.2.3" ────────────────────── DNS Server
PC11 ──③── [IP Packet to 10.1.2.3] ──────────────────► Server1
```

1. PC11 sends a **DNS query** to the DNS server asking for the IP address of `Server1`.
2. The DNS server replies with the IP address (`10.1.2.3`).
3. PC11 can now send IP packets directly to `Server1` using that address.

> 💡 No single DNS server knows all names — DNS servers worldwide work together, forwarding queries until the server that knows the answer provides the IP address.

---

### ARP — Address Resolution Protocol

When a host or router needs to send an IP packet inside an Ethernet frame, it knows the **destination IP address** but not the **destination MAC address**. **ARP** solves this problem.

**How ARP works:**

```
R3 broadcasts: "Who has IP 150.150.4.10? Tell me your MAC address!"
               ──── ARP Request (Ethernet Broadcast) ────────────► All devices on LAN

PC2 replies:   "I have 150.150.4.10! My MAC is 0200.2222.2222"
               ◄─── ARP Reply (Ethernet Unicast to R3) ──────────── PC2
```

- The **ARP Request** is sent as an **Ethernet broadcast** (all devices on the LAN receive it).
- The **ARP Reply** is sent as a **unicast** back to the requester.
- Both devices then store the IP-to-MAC mapping in their **ARP cache (ARP table)** to avoid repeating the process.

> 📝 You can view the ARP cache on most operating systems with the command: `arp -a`

---

### Ping — Testing IP Connectivity

**Ping** uses **ICMP (Internet Control Message Protocol)** to test basic IP connectivity between two hosts.

**How ping works:**

```
Hannah ──── ICMP Echo Request [Eth | IP | ICMP] ────► Harold
Hannah ◄─── ICMP Echo Reply   [Eth | IP | ICMP] ──── Harold
```

- Hannah sends an **ICMP Echo Request** to Harold's IP address.
- Harold replies with an **ICMP Echo Reply**.
- If the reply arrives → the IP network between the two hosts is working ✅
- Ping tests **Layers 1, 2, and 3** of the OSI model — it does not rely on any application.

> 📝 ICMP defines many other messages beyond ping — it is used broadly to help manage and control IP networks.

---
