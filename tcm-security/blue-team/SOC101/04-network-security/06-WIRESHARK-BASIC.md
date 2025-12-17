# WIRESHARK BASIC

## Netcat (NC) Server + Wireshark

Terminal 1 – Start NC listener (server)
```
nc -lvp 1234
```
Explanation
-l → listen mode
-v → verbose
-n → no DNS resolution
-p 1234 → listening port

Terminal 2 – Connect to NC server (client)
```
nc 127.0.0.1 1234
```

Terminal 3 – Start Wireshark (Loopback interface)
```
3 terminal
sudo wireshark (interface lo = loopback)
```


