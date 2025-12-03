#   Email Content Analysis
```
-----------------------------------------------
4. CONTENT DECODING USING CYBERCHEF
-----------------------------------------------
Operations performed:
• From Base64  
• From Quoted Printable  
• URL Decode  
• From HTML Entities  

MIME-Version: 1.0 --> standard version 
Content-Type: 
---> text/plain = text 
---> text/html = HTML 
---> text/MIXED = text + html
Content-Transfer-Encoding: base64 ---> coding
• Under Content-Transfer-Encoding: base64 we can see a part like :

QnVvbmdpb3JubywNCnZvbGV2byBjb211bmljYXJ2aSAsIGNoZSDDqCB1c2Np
dG8gdW4gaW1wb3J0YW50ZSBhZ2dpb3JuYW1lbnRvIGRhIGVzZWd1aXJlIHN1
IEFsY2F0ZWwgb3hvLCBzaWEgZGFsIHB1bnRvIGRpIHZpc3RhIGZ1bnppb25h
bGUgY2hlIGRhbCBwdW50byBkaSB2aXN0YSBkZWxsYSBzaWN1cmV6emEsIGF2
ZW5kbyB2b2kgaWwgY29udHJhdHRvIGRpIGFzc2lzdGVuemEgZG92cmVtbW8g
cHJvZ3JhbW1hcmUgdGFsZSBhZ2dpb3JuYW1lbnRvLg0KTm9uIGNpIHNhcmFu
bm8gY29zdGkgaW4gcXVhbnRvIHBhc3NhdGUgYWxsYSBtYWpvciByZWxlYXNl
LCBs4oCZYWdnaW9ybmFtZW50byB2ZXJyw6Ag ==

--> use cyberchef for decoding https://gchq.github.io/CyberChef/
    > From base64/from Quoted Printable
    > URL DECODE 
    > Defang URL 

• MalwareCube https://github.com/MalwareCube/Email-IOC-Extractor
     > python3 eioc.py ../file path
     > automaticaly we can see spf, dkim, dmarc ecc
     > Extract URL


-----------------------------------------------
5. URL ANALYSIS (MANUAL)
----------------------------------------------- 
• PishTank https://www.phishtank.com/
• URL2PNG https://www.url2png.com/
• VirusTotal https://www.virustotal.com/gui/home/upload
• Use urlscan.io (https://urlscan.io/) to analyze: 
  - screenshot  
  - network requests  
  - redirect chain  
  - domain/IP reputation  
- JoeSandBox https://www.joesandbox.com/#windows

```







