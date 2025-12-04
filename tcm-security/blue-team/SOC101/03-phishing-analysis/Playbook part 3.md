#   Email Attachment Analysis
```
-----------------------------------------------
6. ATTACHMENT EXTRACTION AND IOC EXTRACTION
-----------------------------------------------

-----------------------------------------------
6.1 eioc.py (MalwareCube Email IOC Extractor)
-----------------------------------------------
Python command:
    python3 eioc.py email.eml

Extracts:
• URLs  
• Hashes (MD5, SHA1, SHA256)  
• Detects presence of attachments and extracts their hashes  
  (does NOT extract/save the actual files)
• IP indicators

-----------------------------------------------
6.2 emldump.py (extract attachments from .eml) https://github.com/DidierStevens/DidierStevensSuite/blob/master/emldump.py
-----------------------------------------------
Python command:
    python3 ../file path/emldump.py <email.eml>

OUTPUT LIKE:
    1: M multipart/mixed  
    2: M multipart/related  
    3: 1528 text/html  
    4: 114688 application/octet-stream (filename.iso)

Extract item 4:
    python3 ../file path/emldump.py <email.eml> -s 4 -d > <filename>

→ This creates the extracted file in the current folder (example: filename.iso)

Calculate hash:
    sha256sum <filename>     (Linux/macOS)
or Windows:
    Get-FileHash .\<filename>

Use VirusTotal / Hybrid Analysis to check reputation.

-----------------------------------------------
6.3 oledump.py – identify macro streams
-----------------------------------------------
List all OLE/VBA streams:
    python3 ../file path/oledump.py <suspicious.doc>

OUTPUT LIKE:
  1:       114 '\x01CompObj'
  2:      4096 '\x05DocumentSummaryInformation'
  3:      4096 '\x05SummaryInformation'
  4:      6509 '1Table'
  5:       409 'Macros/PROJECT'
  6:        65 'Macros/PROJECTwm'
  7: M    3716 'Macros/VBA/Module1'
  8: m     924 'Macros/VBA/ThisDocument'
  9:      2601 'Macros/VBA/_VBA_PROJECT'
 10:       563 'Macros/VBA/dir'
 11:      4096 'WordDocument'

EXPLANATION:
1st column  = stream number  
2nd column = indicator:  
              M = macro stream with code  
              m = macro present but not auto-open  
              A = auto-executable macro (AutoOpen / Document_Open)  
3rd column = stream size

Extract specific stream (example: stream 7):
    python3 oledump.py <suspicious.doc> -s 7 -S

If macro is corrupted/offuscata:
    python3 oledump.py <suspicious.doc> -s 7 --vbadecompresscorrupt

Used to:
• Reveal VBA macros  
• Find AutoOpen()  
• Detect PowerShell commands  
• Identify downloaders / dropper logic  

-----------------------------------------------
7. PDF MALWARE ANALYSIS
-----------------------------------------------

-----------------------------------------------
7.1 pdfid.py – static indicator scan
-----------------------------------------------
    python3 ../file path/pdfid.py suspicious.pdf

Detects:
• /JS  
• /JavaScript  
• /OpenAction  
• /Launch  
• /EmbeddedFile  

-----------------------------------------------
7.2 pdf-parser.py – object-level PDF analysis
-----------------------------------------------
List objects:
    python3 ../file path/pdf-parser.py suspicious.pdf | more
or search URLs:
    python3 ../file path/pdf-parser.py suspicious.pdf -s "/URI"

Example:
obj 1 0
 Type: /Catalog
 Referencing: 2 0 R
 /OpenAction 8 0 R

obj 2 0
 Type: /Pages
 Referencing: 3 0 R

obj 3 0
 Type: /Page
 Referencing: 4 0 R, 5 0 R

obj 4 0
 Type: /JS
 /JavaScript 7 0 R

obj 5 0
 Type: /EmbeddedFile
 Contains stream
 Filter: FlateDecode
 Stream length: 10240 bytes

obj 6 0
 Contains stream
 Filter: FlateDecode
 Stream length: 290 bytes
---- Stream ----
(this.exportDataObject({ cName: "update.doc", nLaunch: 2 }))

Extract embedded file (example: object 5):
    python3 ../file path/pdf-parser.py suspicious.pdf --object 5 --filter --raw > update.doc
    python3 ../file path/oledump.py update.doc
    python3 ../file path/oledump.py update.doc -s <stream number> --vbadecompresscorrupt
                                                
-----------------------------------------------
JUST FOR WINDOWS 
-----------------------------------------------
Safest way to extract attachments is using emldump.py.
Do NOT double-click attachments from Outlook/Thunderbird.

POWERSHELL:
    Get-FileHash .\<filename>
    Get-FileHash -Algorithm MD5 .\<filename>
    Get-FileHash -Algorithm SHA1 .\<filename>

-----------------------------------------------
TOOLS for SANDBOX 
-----------------------------------------------
- JoeSandbox
- Any.run https://app.any.run
- Hybrid Analysis https://hybrid-analysis.com/

JUST FOR FUN
https://github.com/rf-peixoto/phishing_pot
https://bazaar.abuse.ch/
```
