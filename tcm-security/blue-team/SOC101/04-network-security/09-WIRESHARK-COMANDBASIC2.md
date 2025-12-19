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

If you mark multiple packets, you can display only marked packets using:

```text
frame.marked == 1
```

---

## Unmarking Packets

To unmark packets:
```
Ctrl + M
```

---

## Wireshark Statistics

Access statistics from:
```
Statistics → ...
```

### Capture File Properties
```
Statistics → Capture File Properties
```

### Resolved Addresses
```
Statistics → Resolved Addresses
```

### Protocol Hierarchy
```
Statistics → Protocol Hierarchy
```

### Conversations
```
Statistics → Conversations
```

### Endpoints
```
Statistics → Endpoints
```

### HTTP Requests
```
Statistics → HTTP → Requests
```

---

## Exporting Data from a Capture

Wireshark allows exporting extracted information directly from a capture file.

Access export options from:
```
File → Export Objects
```

Depending on the protocol, you can export:
- HTTP objects
- SMB files
- FTP transfers
- URLs and requested resources

Example:
```
File → Export Objects → HTTP
```

This allows you to:
- Extract downloaded files
- Export URLs requested during the capture
- Quickly review accessed resources without manual filtering

---
