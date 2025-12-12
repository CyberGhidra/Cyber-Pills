# Network security
```
Common Ports :
FTP: 21
SSH: 22
TELNET: 23
SMTP: 25 
DNS: 53
HTTP: 80
POP3: 110
RPC (MS RPC): 135
NETBIOS: 139
IMAP: 143
LDAP: 389
HTTPS: 443
SMB: 445  
RDP: 3389
HTTP-ALT / Proxy: 8080

## Install tcpdump

sudo apt update
sudo apt install tcpdump


TCPDUMP needs to know which NIC (Network Interface Card) to capture traffic from.
- ip link show 
example :
1: lo        → loopback (127.0.0.1)
```

''
2: enp0s3    → ethernet
3: wlan0 / wlp2s0 → wireless


