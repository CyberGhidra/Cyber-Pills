# Capture and display

## Packet Time Configuration

The first thing we usually configure is the packet timestamp format.
```
View → Time Display Format → Date and Time of Day → UTC
This allows:
--> Consistent timestamps across systems
--> Easier correlation with logs (SIEM, server logs, firewall logs)

```
## Capture vs Display Filters

Wireshark supports two different types of filters:

### Capture Filters

1) Applied before packets are captured

2) Reduce the amount of traffic collected

3) Use BPF syntax (same as tcpdump)

Example:
```
tcp port 80
```

You can set a capture filter:

When selecting the interface Or before starting the capture


### Display Filters

1) Applied after packets are captured

2) Do not affect the capture itself

3) Use Wireshark display filter syntax

Example:
```
tcp.port == 80
```

You can:
Start a capture without filters or Apply display filters later inside Wireshark


### Creating Filters from Packets (GUI Method)

If you are not familiar with the filter syntax, Wireshark can generate filters automatically.

### Method 1: Prepare as Filter

Right-click on a packet field

Select:
```
Prepare as Filter →
```
The filter syntax is created but not applied yet. Click Apply to activate it


### Method 2: Apply Filter Immediately

Right-click on a packet field

Select:
```
Apply as Filter →
```
The filter is applied immediately based on the selected packet


### Saving Captures

To save a capture:
```
File → Save As
```

### Common formats
```
.pcap
```
Standard format compatible with:

- Wireshark

- tcpdump

- tshark

- Most network analysis tools

### Wireshark Next Generation format
```
.pcapng
```
Supports:

- Multiple interfaces

- Comments and metadata

- Best for Wireshark-only analysis


