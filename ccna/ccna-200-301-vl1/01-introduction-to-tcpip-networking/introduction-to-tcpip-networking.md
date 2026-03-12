# Introduction to TCP/IP Networking

---

## 1. The TCP/IP Model

The **TCP/IP model** organizes networking functions into **5 layers**. Each layer has a specific role and uses defined protocols to communicate.

| # | Layer | Key Protocols |
|---|-------|--------------|
| 5 | **Application** | HTTP, POP3, SMTP |
| 4 | **Transport** | TCP, UDP |
| 3 | **Network** | IP, ICMP |
| 2 | **Data Link** | Ethernet (framing) |
| 1 | **Physical** | Ethernet (cables), Wi-Fi |

> 💡 **Tip:** A common mnemonic to remember the layers from bottom to top: **"Please Do Not Throw Sausage Pizza Away"**

---

## 2. Application Layer

The **Application Layer** provides services directly to software applications running on a device (e.g., a web browser like Chrome).

### How HTTP Works — A Practical Example

The **HyperText Transfer Protocol (HTTP)** is one of the most common Application Layer protocols. It defines how a web browser and a web server communicate.

Here is a simplified HTTP exchange:

```
Web Browser  ──── GET /home.html (HTTP Request) ────────────────►  Web Server
Web Browser  ◄─── HTTP 200 OK | Data: home.html (HTTP Response) ──  Web Server
Web Browser  ◄─── Data: more files for home.html ─────────────────  Web Server
```

**Step-by-step breakdown:**

1. **Request** — The browser sends a `GET` request asking for `home.html`, including an HTTP header with metadata.
2. **Response** — The server replies with an HTTP header containing status code `200 OK` (meaning the request was successful) along with the requested data.
3. **Data transfer** — Additional files needed to render the page (images, CSS, JS) are sent in subsequent messages.

---

## 3. Transport Layer

The **Transport Layer** is responsible for end-to-end communication between two hosts. It decides *how* data is delivered — reliably or quickly.

It uses two main protocols:

### TCP — Transmission Control Protocol

- **Reliable** delivery: guarantees that all data arrives correctly and in order.
- Uses **error detection and retransmission**: if a packet is lost, it is sent again.
- Higher overhead due to its control mechanisms.
- **Use cases:** web browsing (HTTP/HTTPS), email, file transfers.

### UDP — User Datagram Protocol

- **Fast** but **unreliable**: packets may be lost without retransmission.
- Lower overhead — no connection setup, no acknowledgements.
- **Use cases:** video streaming (YouTube), online gaming, VoIP calls.

### TCP Error Recovery — Example

TCP uses **sequence numbers** to track packets and detect losses:

```
Web Server  ──── TCP SEQ=1 | HTTP 200 OK | Data: home.html ──────►  Web Browser
Web Server  ──── TCP SEQ=2 | Data ──────────────── X (lost) ─────►  Web Browser
Web Browser ◄─── "Please resend packet 2" (TCP ACK/NACK) ──────────  Web Server
Web Server  ──── TCP SEQ=2 | Data (retransmitted) ───────────────►  Web Browser
```

1. The server sends packet 1 successfully.
2. Packet 2 is lost in transit.
3. The browser detects the gap in sequence numbers and asks for retransmission.
4. The server resends packet 2 — this mechanism is called **TCP error recovery**.

---

## 4. Key Concepts: Layer Interaction

### Same-Layer Interaction *(between different hosts)*

Two devices on different computers use the **same protocol** to communicate at the same layer. The protocol defines a **header** that both devices use to understand each other's intentions.

```
Host A                          Host B
[ Application ]  ◄── HTTP ───►  [ Application ]
[ Transport   ]  ◄── TCP  ───►  [ Transport   ]
[ Network     ]  ◄── IP   ───►  [ Network     ]
```

### Adjacent-Layer Interaction *(within the same host)*

On a **single device**, a lower layer provides a **service** to the layer directly above it. The upper layer requests the lower layer to perform a function on its behalf.

```
[ Application ]
      │  requests service
      ▼
[ Transport   ]
      │  requests service
      ▼
[ Network     ]
```

---

## 5. Network Layer

The **Network Layer** is primarily defined by the **Internet Protocol (IP)**. Its two main responsibilities are **addressing** and **routing**.

### IP and the Postal Service Analogy

IP works much like the postal service:

| Postal Service | TCP/IP Network Layer |
|---|---|
| Each address is unique (house, apartment) | Each host has a unique **IP address** |
| Post offices forward letters toward the destination | **Routers** forward **packets** toward the destination |
| Postal infrastructure (trucks, planes, sorting) | Network infrastructure (cables, switches, routers) |

The **Application and Transport layers** act like the person sending a letter — they don't need to know the exact route. The **Network layer** acts like the postal service — it must understand the infrastructure and make routing decisions.

### IP Addressing Basics

Every device on a TCP/IP network (called an **IP host**) needs a **unique IP address**. Addresses are written in **dotted-decimal notation (DDN)** — four numbers separated by periods.

**Example:**

```
Larry  →  IP address: 1.1.1.1
Bob    →  IP address: 2.2.2.2
Archie →  IP address: 3.3.3.3
```

IP also groups addresses into ranges (similar to ZIP codes in postal systems):

```
Addresses starting with 1._._._  →  Larry's network (upper left)
Addresses starting with 2._._._  →  Bob's network (right)
Addresses starting with 3._._._  →  Archie's network (bottom)
```

### IP Routing Basics

**Routers** are networking devices that connect different IP networks and forward packets toward the correct destination.

> 📝 **Note:** The term *IP host* refers to any device — regardless of size or power — that has an IP address and connects to a TCP/IP network.

**Basic routing example** (Larry → Bob, through R1 and R2):

```
Larry (1.1.1.1) ──①──► Router R1 ──②──► Router R2 ──③──► Bob (2.2.2.2)

IP | TCP | HTTP > IP contains addresses 1.1.1.1 and 2.2.2.2
```


| Step | Device | Action |
|------|--------|--------|
| ① | Larry | Sends the IP packet to R1 (nearest router), without knowing the full path |
| ② | Router R1 | Reads destination (`2.2.2.2`), checks routing table, forwards to R2 |
| ③ | Router R2 | Repeats same logic, delivers packet directly to Bob |

This process of forwarding IP packets hop-by-hop is called **IP routing**.

---

## 6. Data Link & Physical Layers

The **Data Link** and **Physical** layers work together to deliver data across a single physical link (e.g., an Ethernet cable or Wi-Fi connection).

- The **Physical layer** defines the cabling and electrical signals (or radio waves) used to transmit raw bits.
- The **Data Link layer** defines the rules for using the physical medium — how to format data into **frames** and how to detect errors on the link.

### Encapsulation at the Data Link Layer — Example

When Larry (`1.1.1.1`) sends a packet to Router R1 over Ethernet, four steps occur:

| Step | Where | Action |
|------|-------|--------|
| 1 | Larry | **Encapsulates** the IP packet into an Ethernet **frame** (adds header + trailer) |
| 2 | Larry | Transmits the bits electrically over the Ethernet cable |
| 3 | Router R1 | Receives the electrical signal and reconstructs the bits |
| 4 | Router R1 | **De-encapsulates** the frame — removes Ethernet header and trailer, extracts the IP packet |

```
Larry:    [ Ethernet Header | IP Packet | Ethernet Trailer ]  ──►  Router R1
R1:       strips header/trailer  →  extracts  [ IP Packet ]
```

> 📝 **Note:** Unlike other layers, the Data Link layer adds both a **header** (at the beginning) and a **trailer** (at the end) of the message.

---

## 7. Data Encapsulation

### What is Encapsulation?

**Encapsulation** is the process by which each layer adds its own **header** (and sometimes **trailer**) to the data it receives from the layer above, before passing it down.

### The 5-Step Encapsulation Process

| Step | Layer | Action | Result |
|------|-------|--------|--------|
| 1 | **Application** | Creates data + application header (e.g., HTTP) | — |
| 2 | **Transport** | Wraps data in a TCP or UDP header | **Segment** |
| 3 | **Network** | Wraps segment in an IP header | **Packet** |
| 4 | **Data Link** | Wraps packet in link header + trailer | **Frame** |
| 5 | **Physical** | Encodes frame as bits, transmits on medium | Bits |

### Names of TCP/IP Messages (PDUs)

Each layer has a specific name for its unit of data, called a **PDU (Protocol Data Unit)**:

```
[ TCP Header | Data ]                →  Segment  (Transport Layer)
[ IP Header | TCP Header | Data ]    →  Packet   (Network Layer)
[ LH | IP | TCP | Data | LT ]       →  Frame    (Data Link Layer)
```

> `LH` = Link Header &nbsp;&nbsp; `LT` = Link Trailer

When analyzing a specific layer, everything after its own header is simply called **"data"** — internal details of upper layers are irrelevant at that level.

---
### Packet Journey Example — Larry sends a web page to Bob
```
Larry (1.1.1.1)
      │
      │  [Eth.H | IP | TCP | Data | Eth.T]  ← Ethernet frame to R1
      ▼
  Router R1
      │  de-encapsulates the Ethernet frame (removes header and trailer)
      │  reads IP header → destination 2.2.2.2 → forwards to R2
      │  re-encapsulates into a NEW Ethernet frame toward R2
      │  [Eth.H | IP | TCP | Data | Eth.T]  ← new frame!
      ▼
  Router R2
      │  same logic → reads IP → Bob is here!
      │  re-encapsulates toward Bob
      ▼
  Bob (2.2.2.2)
      │  de-encapsulates everything, layer by layer
      │  Layer 2 → removes Ethernet header/trailer
      │  Layer 3 → removes IP header
      │  Layer 4 → removes TCP header, reassembles data
      │  Layer 5 → receives home.html
```

## 8. OSI Model

### What is OSI?

The **OSI (Open Systems Interconnection)** model is an older networking framework that was once a competitor to TCP/IP. OSI is no longer used as an active networking standard, but its **layer terminology** remains widely used across the networking industry.

### Why Learn OSI?

During the late 1980s and early 1990s, many vendors and protocol documents adopted OSI layer names and numbers. That terminology is still in everyday use — so understanding OSI is essential for reading networking documentation and discussing concepts with other engineers.

### OSI vs TCP/IP — Layer Comparison

| OSI # | OSI Name | TCP/IP (Common) | TCP/IP (RFC 1122) |
|-------|----------|-----------------|-------------------|
| 7 | Application | Application (5–7) | Application (5–7) |
| 6 | Presentation | ↑ | ↑ |
| 5 | Session | ↑ | ↑ |
| 4 | Transport | Transport | Transport |
| 3 | Network | Network | Internet |
| 2 | Data Link | Data Link | Link (1–2) |
| 1 | Physical | Physical | ↑ |

### Key Points

- Even though the world runs TCP/IP today, we still use **OSI layer numbers** as a reference.
  - HTTP → **Layer 7 protocol**
  - IP → **Layer 3 protocol**
  - Ethernet → **Layer 2 protocol**
- The bottom four layers (1–4) have the **same names and meanings** in both OSI and TCP/IP.
- OSI's layers 5 (Session), 6 (Presentation), and 7 (Application) are all mapped to TCP/IP's single **Application layer**.

> 📝 **CCNA Exam Note:** The exam no longer explicitly tests OSI vs TCP/IP model comparisons, but OSI terminology appears throughout all networking discussions — you need to know it.
