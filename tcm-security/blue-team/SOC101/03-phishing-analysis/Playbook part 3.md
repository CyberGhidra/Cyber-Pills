#   Email Attachment Analysis
```
-----------------------------------------------
6. ATTACHMENT EXTRACTION AND IOC EXTRACTION
-----------------------------------------------

-----------------------------------------------
6.1 emldump.py (extract attachments from .eml) https://github.com/DidierStevens/DidierStevensSuite/blob/master/emldump.py
-----------------------------------------------
Python command:
    python3 ../file path/emldump.py <email.eml>
OUTPUT LIKE
    1: M multipart/mixed > container 
    2: M multipart/related > sotto-container 
    3: 1528 Text/HTML
    4: 114688 Application/octet-strem (nomefile.iso) example
Extract number 4 
    python3 ../file path/emldump.py <email.eml> -s 4 -d > <nome file>
    --> inside the same folder we can see the sha256, sha1 or md5
    --> command sha256 <name file>
    --> OUTPUT like = 75gf85js96sjs78vd...
    --> take and use virustotal to see if is good 
-----------------------------------------------
6.2 eioc.py (MalwareCube Email IOC Extractor)
-----------------------------------------------
Python command:
    python3 eioc.py email.eml

Extracts:
• URLs  
• Hashes (MD5, SHA1, SHA256)  
• Attachments  
• IP indicators

-----------------------------------------------
6.3 oledump.py – identify macro streams
-----------------------------------------------
List all OLE/VBA streams:
    python3 ../file path/oledump.py <suspicious.doc> just Microsoft files or VBA ecc

EXTRACT LIKE 
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

1 column = n stream
3 column = stream Wheigth
2 columns = M > active macro
            m = noautoactive macro

Extract specific stream (example stream 7):
    python3 oledump.py <suspicious.doc> -s 7 -S

Decompress corrupted macro stream:
    python3 oledump.py suspicious.doc -s 7 --vbadecompresscorrupt

Used to:
• Reveal VBA macros  
• Find AutoOpen()  
• Detect PowerShell commands  
• Identify downloaders / dropper logic  

JUST FOR WINDOWS 

there isn't a safe mode to download a file from EML.
But we can analys in the same way 
POWERSHELL > Get-filehash .\<file name>
or
> Get-filehash -algorithm md5 .\<file name>
> Get-filehash -algorithm sha1 .\<file name>

TOOLS for SANDBOX 
- JoeSandbox
- Any.run https://app.any.run
- Hybrid Analysis https://hybrid-analysis.com/



```
