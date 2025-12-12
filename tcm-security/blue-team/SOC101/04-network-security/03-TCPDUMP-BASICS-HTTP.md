# TCPDUMP Basics – HTTP Traffic Analysis
```
This section demonstrates how to analyze HTTP traffic using tcpdump.

1) Terminal 1 – Start a local HTTP server:

   python3 -m http.server 80

2) Terminal 2 – Send an HTTP request for a non-existent file:

   curl http://localhost/file.php

3) Terminal 3 – Capture HTTP traffic:

   sudo tcpdump -i lo -A

4) HTTP Analysis Options

   lo
   → Loopback interface (127.0.0.1)

   -A
   → Displays payload in ASCII format
   Useful for HTTP headers, requests, responses

    OR 

   -X
   → Displays payload in hexadecimal + ASCII
   Useful for binary data or malware analysis

----------------------------------------------------
EXAMPLE -A

12:45:01.123456 IP localhost.54321 > localhost.http: Flags [P.], length 78
GET /file.php HTTP/1.1  → HTTP method used to request a resource
Host: localhost   → Target host
User-Agent: curl/8.5.0 → Client application generating the request
Accept: */*  → Client accepts any response type

12:45:01.124001 IP localhost.http > localhost.54321: Flags [P.], length 64
HTTP/1.0 404 File not found → Server response indicating the requested file does not exist
Server: SimpleHTTP/0.6 Python/3.12.3   → Server software and version
Date: Mon, 09 Dec 2025 12:45:01 GMT  → Timestamp of the server response

-----------------------------------------------------

EXAMPLE -X 
12:45:01.123456 IP localhost.54321 > localhost.http: Flags [P.], length 78
0x0000:  4745 5420 2f66 696c 652e 7068 7020 4854  GET /file.php HT
0x0010:  5450 2f31 2e31 0d0a 486f 7374 3a20 6c6f  TP/1.1..Host: lo
0x0020:  6361 6c68 6f73 740d 0a55 7365 722d 4167  calhost..User-Ag
0x0030:  656e 743a 2063 7572 6c2f 382e 352e 300d  ent: curl/8.5.0.
0x0040:  0a41 6363 6570 743a 202a 2f2a 0d0a 0d0a  .Accept: */*....

12:45:01.124001 IP localhost.http > localhost.54321: Flags [P.], length 64
0x0000:  4854 5450 2f31 2e30 2034 3034 2046 696c  HTTP/1.0 404 Fil
0x0010:  6520 6e6f 7420 666f 756e 640d 0a53 6572  e not found..Ser
0x0020:  7665 723a 2053 696d 706c 6548 5454 502f  ver: SimpleHTTP/
0x0030:  302e 3620 5079 7468 6f6e 2f33 2e31 322e  0.6 Python/3.12.
```
### How to Read Hex + ASCII Output
```
0x0000, 0x0010, 0x0020
→ Offset (byte position within the payload)

Left column (hexadecimal)
→ Raw bytes as transmitted on the network

Right column (ASCII)
→ Human-readable representation of the same bytes

```
### Why HTTP Status Codes Matter
```
- 200 → Successful request
- 301 / 302 → Redirection
- 401 / 403 → Authentication or authorization issues
- 404 → Resource not found (often used in scanning or enumeration)
- 500 → Server-side error

Note:
The presence of repeated 404 responses may indicate scanning or enumeration activity.

```
 
  

 






