# Wireshark – PCAP Analysis Workflow (SOC Style)

This document describes a **practical, step-by-step workflow** for analyzing a PCAP file using Wireshark,
following a **SOC / Blue Team mindset**.

For additional PCAP exercises and real-world malware traffic, refer to:
- https://www.malware-traffic-analysis.net

---

## 1. Opening Wireshark and the PCAP File

Start Wireshark (commonly in lab environments):

```bash
sudo wireshark
```

Assumption:
- The PCAP file has already been captured and saved (`.pcap` or `.pcapng`)

Open the file:
```
File → Open → select capture.pcap
```

---

## 2. Initial UI Setup

### Customize Columns

```
View → Preferences → Appearance → Columns
```

Add:
- **Src Port (unresolved)**
- **Dst Port (unresolved)**

---

### Time Configuration

```
View → Time Display Format → Date and Time of Day → UTC
```

---

## 3. High-Level Analysis (Statistics First)

```
Statistics → Capture File Properties
```

```
Statistics → Conversations → IPv4
```

Look for:
- Highest traffic exchanges between hosts

---

## 4. Identifying Suspicious Traffic

Apply filter:
```text
http
```

Look for multiple or unusual HTTP requests.

---

## 5. Inspecting an HTTP Request

Example suspicious URI:
```
/8660.dat
```

Inspect:
```
Hypertext Transfer Protocol
```

---

## 6. Follow HTTP Stream

```
Follow → HTTP Stream
```

Look for:
- Full request/response
- User-Agent (e.g. curl)

---

## 7. File Signature Identification

Malware often uses generic extensions.

Common magic bytes:
- `MZ` → Windows executable

Refer to:
- Wikipedia → List of file signatures

---

## 8. Exporting the File

```
File → Export Objects → HTTP
```

Export:
- `8660.dat`

---

## 9. Hashing the File

```bash
sha256sum 8660.dat
```

---

## 10. VirusTotal Analysis

Paste the hash into:
- https://www.virustotal.com

Example result:
- Malware family: **QakBot / QuickBot**

---

## 11. MITRE ATT&CK Mapping

Search the malware on:
- https://attack.mitre.org

Understand:
- Tactics
- Techniques
- Behavior

---

## 12. Next Steps

Now that the malware behavior is known:
- Hunt for additional indicators
- Analyze DNS, C2, and lateral movement

---

## SOC Analyst Mindset

> Start broad, then go deep.  
> Let the PCAP tell the story.
