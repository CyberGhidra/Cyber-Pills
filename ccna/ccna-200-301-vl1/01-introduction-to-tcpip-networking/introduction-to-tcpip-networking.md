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

---

### TCP Error Recovery — Example

TCP uses **sequence numbers** to track packets and detect losses:

```
Web Server  ──── TCP SEQ=1 | HTTP 200 OK | Data: home.html ──────►  Web Browser
Web Server  ──── TCP SEQ=2 | Data ──────────────── X (lost) ─────►  Web Browser
Web Browser ◄─── "Please resend packet 2" (TCP ACK/NACK) ──────────  Web Browser
Web Server  ──── TCP SEQ=2 | Data (retransmitted) ───────────────►  Web Browser
```

1. The server sends packet 1 successfully.
2. Packet 2 is lost in transit.
3. The browser detects the gap in sequence numbers and asks for retransmission.
4. The server resends packet 2.

This mechanism is called **TCP error recovery** (or **retransmission**).

---

## 4. Key Concepts: Layer Interaction

Understanding how layers interact is fundamental to the TCP/IP model.

### Same-Layer Interaction *(between different hosts)*

Two devices on different computers use the **same protocol** to communicate with each other at the same layer.

- The protocol defines a **header** that both devices use to understand each other's intentions.
- Example: a browser and a web server both "speak" HTTP at the Application Layer.

```
Host A                          Host B
[ Application ]  ◄── HTTP ───►  [ Application ]
[ Transport   ]  ◄── TCP  ───►  [ Transport   ]
[ Network     ]  ◄── IP   ───►  [ Network     ]
```

### Adjacent-Layer Interaction *(within the same host)*

On a **single device**, a lower layer provides a **service** to the layer directly above it.

- The upper layer requests the lower layer to perform a function on its behalf.
- Example: the Transport Layer asks the Network Layer to route its segment to the correct destination.

```
[ Application ]
      │  requests service
      ▼
[ Transport   ]
      │  requests service
      ▼
[ Network     ]
```
