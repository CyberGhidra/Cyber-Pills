# Wireshark – Command Basics (Display Filters)

⚠️ **Display Filters**
- Used in the **top filter bar** inside Wireshark
- ❌ NOT tcpdump syntax
- Used for **analysis**, not capture
  
---

## IP / Host Filters

```text
ip.addr == 192.168.1.10
```
→ Matches source OR destination IP

```text
ip.src == 192.168.1.10
```
→ Traffic sent from the IP

```text
ip.dst == 192.168.1.10
```
→ Traffic received by the IP

```text
ip.addr == 192.168.1.0/24
```
→ Traffic within a subnet

---

## TCP / UDP Filters

```text
tcp
udp
```
→ Show only TCP or UDP traffic

```text
tcp.port == 80
udp.port == 53
```
→ Filter by port

```text
tcp.srcport == 443
tcp.dstport == 22
```
→ Filter by source or destination port

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```
→ New TCP connections / SYN scan detection

---

## HTTP Filters

```text
http
```
→ All HTTP traffic

```text
http.request
http.response
```
→ Separate requests and responses

```text
http.request.method == "GET"
http.request.method == "POST"
```
→ HTTP methods

```text
http.host contains "login"
```
→ Login-related domains

```text
http.request.uri contains "admin"
```
→ Admin panel access attempts

---

## DNS Filters

```text
dns
```
→ All DNS traffic

```text
dns.qry.name == "example.com"
```
→ Specific domain query

```text
dns.qry.name contains "google"
```
→ Keyword-based domain search

```text
dns.flags.response == 0
```
→ DNS queries only (no responses)

---

## ICMP Filters

```text
icmp
```
→ All ICMP traffic

```text
icmp.type == 8
```
→ Echo request (ping)

```text
icmp.type == 0
```
→ Echo reply

---

## ARP Filters

```text
arp
```
→ ARP traffic

```text
arp.opcode == 1
```
→ who-has request

```text
arp.opcode == 2
```
→ is-at reply

---

## Credential Hunting (SOC Focus)

```text
http contains "password"
http contains "username"
```
→ Clear-text credential discovery

```text
frame contains "password"
```
→ Raw payload search (any protocol)

---

## Malware / C2 Indicators

```text
dns.qry.name contains ".xyz"
```
→ Suspicious TLD detection

```text
http.user_agent contains "curl"
```
→ Automated tools / scripts

```text
tcp.len > 1000
```
→ Large data transfers

```text
tcp.analysis.retransmission
```
→ Network issues or evasion attempts

---

## File Upload / Download

```text
http.response.code == 200
http.response.code == 404
```
→ HTTP response status

```text
http.content_type contains "application"
```
→ File transfer detection

```text
http.request.method == "POST"
```
→ Uploads / credential submission

---

## Time & Performance Analysis

```text
frame.time_delta > 1
```
→ Delayed packets

```text
tcp.analysis.duplicate_ack
tcp.analysis.out_of_order
```
→ TCP performance issues

---

## Specific Conversations

```text
ip.src == 192.168.1.10 && tcp.dstport == 4444
```
→ Isolate host-to-port communication

```text
!(arp || dns)
```
→ Remove background noise

---

## TLS / HTTPS Filters

```text
tls
```
→ Encrypted traffic

```text
tls.handshake
```
→ TLS handshake packets

```text
tls.handshake.type == 1
```
→ Client Hello

```text
tls.handshake.extensions_server_name contains "google"
```
→ SNI extraction (SOC critical)

---

## Combined Filters (Exam Style)

```text
ip.addr == 10.0.2.15 && (http || dns)
```

```text
tcp.port == 4444 || tcp.port == 1337
```

```text
frame.len > 1000 && tcp
```

