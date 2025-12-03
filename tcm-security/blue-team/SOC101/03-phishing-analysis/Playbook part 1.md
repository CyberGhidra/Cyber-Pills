# EMAIL PHISHING ANALYSIS HEADER

This document summarizes the complete workflow for manual and automated phishing email analysis following the methodology from TCM Security SOC101,
including all Python tools used (emldump, oledump, pdfid, pdf-parser, eioc).
```
-----------------------------------------------
1. EMAIL ACQUISITION
-----------------------------------------------
• Obtain the email in .eml format  
• Or extract full headers for manual inspection  
• Download SublimeText https://www.sublimetext.com/ --> tools > install package controll > Control+shift+P ( search bar ) > install package > email header
• select ( down - right part ) email header

-----------------------------------------------
2. HEADER ANALYSIS (MANUAL)
-----------------------------------------------
Checks performed:
• Received bottom to top
• Authentication-Results:
  • SPF result  
  • DKIM signature  
  • DMARC alignment  
• From vs Return-Path mismatch 

IF SPF = FAIL > spoofing 
   DKIM = FAIL > spoofing or manipolation
   DKIM = NONE > Suspicious
   DMARC = FAIL > spoofing 
   DMARC = NONE > Weak domain 
   
-----------------------------------------------
3. CONTENT SEARCH (MANUAL)
-----------------------------------------------
CTRL+F inside the .eml or decoded message:
• Find URLs ---> When no URLs are found: Base64 blocks , Quoted-Printable
• Find encoded strings  
• Identify suspicious redirect patterns 

TOOLS = MxToolBox https://mxtoolbox.com/

```

