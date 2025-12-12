## TCPDUMP Basics – HTTPS Traffic Analysis

This section demonstrates how HTTPS traffic appears in tcpdump
and how it differs from HTTP traffic.

---

### Steps
```
1) Identify the network interface connected to the Internet

On most Linux systems, the active network interface is:
- `enp0s3` → Ethernet interface

You can verify it using:
->bash
ip link show

2) Start capturing traffic on the network interface:
   sudo tcpdump -i enp0s3
   This captures all inbound and outbound network traffic.

3) Generate HTTPS traffic using curl:
   curl https://it.wikipedia.org
   This sends an HTTPS request to Wikipedia.
   
4) Open a browser and navigate to:
   https://it.wikipedia.org
   While browsing, tcpdump will display packets exchanged
   between your system and the Wikipedia servers.

```
## Filters 
Filters allow tcpdump to capture only specific traffic,
reducing noise and making analysis faster and more effective.

### `-i`
Specifies the network interface to capture traffic from.
```
sudo tcpdump -i enp0s3
```
### `-n`
Disables DNS and service name resolution.
Shows IP addresses and port numbers instead of hostnames.
```
sudo tcpdump -i enp0s3 -n
```
### `-A`
Displays payload in ASCII format. Useful for HTTP headers, requests, responses
```
sudo tcpdump -i lo -A
```
### `-X`
Displays payload in hexadecimal + ASCII. Useful for binary data or malware analysis
```
sudo tcpdump -i lo -X
```

### `host`
Matches traffic where the source OR destination matches the host.
```
sudo tcpdump -i enp0s3 -n host example.com
```
OUTPUT
```
sudo tcpdump -i enp0s3 -n host 8.8.8.8

12:01:15 IP 192.168.1.10.52344 > 8.8.8.8.53: UDP, length 42
12:01:15 IP 8.8.8.8.53 > 192.168.1.10.52344: UDP, length 58

```
### `src`
Matches packets coming from a specific IP.
```
sudo tcpdump -i enp0s3 -n src <ip address>
```
OUTPUT
```
sudo tcpdump -i enp0s3 -n src 192.168.1.10

12:05:22 IP 192.168.1.10.49122 > 142.250.184.78.443: TCP Flags [S]
```
### `dst`
Matches packets going to a specific IP.
```
sudo tcpdump -i enp0s3 -n dst <ip address>
```
### `src net`
Matches traffic coming from a subnet.
```
sudo tcpdump -i enp0s3 -n src net <subnet ex 10.0.0.0/8>
```
### `port`
Matches traffic using a specific port (source OR destination)
```
sudo tcpdump -i enp0s3 -n port <specific port>
```
OUTPUT
```
sudo tcpdump -i enp0s3 -n port 443

12:06:10 IP 192.168.1.10.49321 > 172.217.22.14.443: TCP Flags [P.]
```
### `tcp, udp, icmp`
Limits capture to a specific protocol.
```
sudo tcpdump -i enp0s3 -n tcp
```
OUTPUT
```
sudo tcpdump -i enp0s3 -n icmp

12:07:01 IP 192.168.1.10 > 8.8.8.8: ICMP echo request
12:07:01 IP 8.8.8.8 > 192.168.1.10: ICMP echo reply
```
### `-w`
Writes captured packets to a file (pcap format).
```
sudo tcpdump -i enp0s3 -n -w <capture.pcap>
```
### `-r`
Reads packets from a previously saved capture file.
```
tcpdump -r capture.pcap
```
### `--count`
Displays only the total number of packets captured or read.
```
tcpdump -r capture.pcap --count
```
### `-c`
Stops tcpdump after capturing a specific number of packets.
```
sudo tcpdump -i enp0s3 -c 2
```
### `-t`
Removes timestamps completely.
```
tcpdump -r capture.pcap -t
```
### `-tt`
Shows timestamps as seconds since epoch (Unix time).
```
tcpdump -tt -r capture.pcap
```
### `-ttt`
Shows time difference between packets.
```
tcpdump -ttt -r capture.pcap
```
### `-tttt`
Shows full human-readable date and time.
```
tcpdump -tttt -r capture.pcap
```
### `-grep`
Searches for specific strings in the output.
```
tcpdump -A -r capture.pcap | grep "password"
```
### `-E`
Allows advanced patterns (OR conditions, groups).
```
tcpdump -A -r capture.pcap | grep -E "GET|POST"
```
```
