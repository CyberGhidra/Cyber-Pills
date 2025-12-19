# Wireshark – Command Basics 2 (Follow Streams, Marking & Packet Analysis)

This section focuses on **analyzing conversations**, **marking packets**, and **basic packet analysis**
during PCAP investigations. These features are heavily used in SOC investigations and exams.

---

## Following HTTP Conversations

One of the most powerful features in Wireshark is the ability to reconstruct a full conversation.

### Example: HTTP Conversation

1. Identify an HTTP packet (for example):
   - Packet number: `366`
   - Method: `GET`
   - URI: `/dashboard`

2. Right-click on the packet

3. Select:
```
Follow → HTTP Stream
```

Wireshark will:
- Reassemble the entire HTTP conversation
- Show both **request and response**
- Display headers and payload in order

This is useful to:
- Analyze authentication flows
- Inspect session behavior
- Identify malicious requests or data exfiltration

---

## Other Follow Stream Options

Wireshark also supports:
- `Follow → TCP Stream`
- `Follow → UDP Stream`
- `Follow → TLS Stream`

These are useful when HTTP is not present or when analyzing lower-level traffic.

---

## Marking Packets

During analysis, you may want to **mark important packets** for later review.

### How to Mark a Packet

1. Right-click on a packet
2. Select:
```
Mark Packet
```

The packet will be highlighted and flagged as marked.

---

## Viewing All Marked Packets

If you mark multiple packets (for example, 5 packets), you can display only those packets using the following display filter:

```text
frame.marked == 1
```

This allows you to:
- Quickly review important packets
- Focus only on suspicious or relevant traffic

---

## Unmarking Packets

To remove a packet mark:

- Select the marked packet
- Press:
```
Ctrl + M
```

This toggles the mark on or off.

---

## Practical Packet Analysis Example

This is a **basic SOC-style workflow** for analyzing a single packet.

### Step 1: Identify the Packet

Example:
- Packet number: `366`
- Protocol: `HTTP`
- Info: `GET /dashboard HTTP/1.1`

Questions to ask:
- Who is talking?
- What protocol is being used?
- Is this normal behavior?

---

### Step 2: Analyze Layers (Top to Bottom)

#### Frame
- Frame number
- Arrival time (UTC)
- Frame length

#### Ethernet
- Source MAC
- Destination MAC

#### IP
- Source IP
- Destination IP
- TTL (possible OS fingerprinting)

#### TCP
- Source port
- Destination port
- Flags (SYN, ACK, PSH, FIN)
- Sequence and acknowledgment numbers

#### Application (HTTP)
- Method (GET / POST)
- Requested resource
- Host header
- User-Agent

---

### Step 3: Inspect Payload

- Check headers for:
  - Authentication tokens
  - Cookies
  - Suspicious User-Agents
- Look for:
  - Clear-text credentials
  - Suspicious parameters
  - Encoded data

---

### Step 4: Follow the Stream

Right-click the packet:
```
Follow → HTTP Stream
```

- Validate request/response flow
- Confirm server behavior
- Look for redirects, errors, or data leakage

---

### Step 5: Mark Important Packets

- Mark the initial request
- Mark the server response
- Mark any suspicious packet

Filter later using:
```text
frame.marked == 1
```

---

## Common Workflow Example

1. Identify suspicious HTTP request
2. Analyze packet layers
3. Follow HTTP stream
4. Mark relevant packets
5. Filter marked packets
6. Write investigation notes

---

## SOC Analyst Mindset

> Do not analyze packets in isolation.  
> Reconstruct conversations and context.

---

> Mark what matters.  
> Follow the stream, not just the packet.
