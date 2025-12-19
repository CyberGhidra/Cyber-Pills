# Wireshark – Command Basics 2 (Follow Streams, Marking & Statistics)

This section focuses on **following conversations**, **marking packets**, and **using Wireshark statistics**.
These features are commonly used during SOC investigations and PCAP analysis.

---

## Following HTTP Conversations

One of the most powerful features in Wireshark is the ability to reconstruct a full conversation.

### Example: HTTP Conversation

1. Identify an HTTP packet (example):
   - Packet number: `366`
   - Method: `GET`
   - URI: `/dashboard`

2. Right-click on the packet

3. Select:
```
Follow → HTTP Stream
```

Wireshark will:
- Reassemble the full HTTP conversation
- Show both **request and response**
- Display headers and payload in order

This is useful to:
- Understand user behavior
- Analyze authentication flows
- Identify suspicious or malicious requests

---

## Marking Packets

During analysis, you may want to **mark important packets**.

### How to Mark a Packet

1. Right-click on a packet
2. Select:
```
Mark Packet
```

The packet will be highlighted and flagged as marked.

---

## Viewing All Marked Packets

If you mark multiple packets (for example, 5 packets), you can display only marked packets using the following display filter:

```text
frame.marked == 1
```

This allows you to quickly review only relevant packets.

---

## Unmarking Packets

To unmark packets:

- Select the marked packet
- Press:
```
Ctrl + M
```

This toggles the packet mark on or off.

---

## Wireshark Statistics

Wireshark provides multiple **statistics views** that help summarize and understand traffic.

Access statistics from:
```
Statistics → ...
```

---

### Capture File Properties

```
Statistics → Capture File Properties
```

Provides:
- Capture duration
- First and last packet timestamps
- Number of packets
- Capture interfaces

---

### Resolved Addresses

```
Statistics → Resolved Addresses
```

Shows:
- IP to hostname resolution
- MAC to vendor mapping

---

### Protocol Hierarchy

```
Statistics → Protocol Hierarchy
```

Shows:
- Protocol distribution
- Percentage of traffic per protocol

---

### Conversations

```
Statistics → Conversations
```

Shows:
- Communication pairs
- Traffic volume per conversation

---

### Endpoints

```
Statistics → Endpoints
```

Shows:
- Packet and byte counts per host

---

### HTTP Requests

```
Statistics → HTTP → Requests
```

Shows:
- HTTP methods
- Requested URIs

---

## SOC Analyst Tip

- Start with **Statistics** for the big picture
- Use **Follow HTTP Stream** for context
- Use **Marked packets** to build timelines

---

> First get the big picture.  
> Then zoom into the packets that matter.
