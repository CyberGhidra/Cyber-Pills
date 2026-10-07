# 01. The Python Package Manager (pip)

`pip` è il gestore di pacchetti ufficiale e predefinito per Python. Consente di cercare, installare, aggiornare, rimuovere e verificare le librerie di terze parti ospitate su PyPI (Python Package Index) o su repository esterni, oltre a gestire le relative dipendenze all'interno del sistema o di ambienti virtuali.

---

### Flusso Comandi ed Esempi dal Terminale

```bash
# 1. Verifica assenza del modulo in Python
$ python3
Python 3.9.1+ (default, Feb  5 2021, 13:46:56) 
[GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from pwn import *
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'pwn'
>>> exit()

# 2. Installazione dell'ultima versione di un pacchetto
$ pip install pwntools
Installing collected packages: pwntools
WARNING: The scripts asm, checksec, common, constgrep, cyclic, debug, disablenx, disasm, elfdiff, elfpatch, errno, hex, main, phd, pwn, pwnstrip, scramble, shellcraft, template, unhex, update and version are installed in '/home/notroot/.local/bin' which is not on PATH.
Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed pwntools-4.5.1

# 3. Verifica importazione dopo l'installazione
$ python3
>>> from pwn import *
>>> exit()

# 4. Disinstallazione di un pacchetto
$ pip uninstall pwntools

# 5. Installazione di una versione specifica (pinning)
$ pip install pwntools==4.5.0
Installing collected packages: pwntools
Successfully installed pwntools-4.5.0

# 6. Elenco dei pacchetti installati
$ pip list | less

# 7. Esportazione delle dipendenze esatte con pip freeze
$ pip freeze | less
```
# 02. Python Virtual Environments (Ambienti Virtuali)

Un ambiente virtuale in Python è una directory autonoma e isolata che contiene una copia (o symlink) dell'interprete Python, un set dedicato di binari (`pip`, `python`) e una cartella `site-packages` indipendente. Serve a evitare conflitti di versione tra dipendenze di progetti diversi e a non sporcare l'ambiente Python globale di sistema[cite: 10, 12].

---

### Trascrizione del Flusso Operativo dai Terminali

Nelle schermate sono visibili due terminali affiancati:
* **Terminale Sinistro:** Creazione, attivazione e utilizzo del virtual environment.
* **Terminale Destro:** Sistema host Kali Linux (Python globale) usato per confronto.

```bash
# -------------------------------------------------------------
# 1. Installazione globale del tool virtualenv (se non integrato)
# -------------------------------------------------------------
(notroot㉿kali)-[~]
$ pip install virtualenv
Requirement already satisfied: six<2, >=1.9.0 in /usr/lib/python3/dist-packages (from virtualenv) (1.15.0)
Requirement already satisfied: appdirs<2, >=1.4.3 in /usr/lib/python3/dist-packages (from virtualenv) (1.4.4)
Installing collected packages: filelock, distlib, virtualenv
WARNING: The script virtualenv is installed in '/home/notroot/.local/bin' which is not on PATH.
Successfully installed distlib-0.3.2 filelock-3.0.12 virtualenv-20.4.7

# -------------------------------------------------------------
# 2. Creazione della directory di progetto e dell'ambiente virtuale
# -------------------------------------------------------------
(notroot㉿kali)-[~]
$ mkdir virutal-demo
$ cd virutal-demo

# Creazione del venv usando il modulo integrato 'venv' (chiamato 'env')
(notroot㉿kali)-[~/virutal-demo]
$ python3 -m venv env

# -------------------------------------------------------------
# 3. Attivazione dell'ambiente virtuale
# -------------------------------------------------------------
(notroot㉿kali)-[~/virutal-demo]
$ source env/bin/activate

# Notare il prefisso '(env)' comparso nel prompt:
(env)(notroot㉿kali)-[~/virutal-demo]
$

# -------------------------------------------------------------
# 4. Verifica dell'isolamento iniziale (assenza di librerie terze)
# -------------------------------------------------------------
(env)(notroot㉿kali)-[~/virutal-demo]
$ python3
Python 3.9.2 (default, Feb 28 2021, 17:03:44) 
[GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from pwn import *
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'pwn'
>>> exit()

# -------------------------------------------------------------
# 5. Ispezione della struttura interna di 'env'
# -------------------------------------------------------------
(env)(notroot㉿kali)-[~/virutal-demo]
$ ls -la
total 12
drwxr-xr-x  3 notroot notroot 4096 Jun 13 00:38 .
drwxr-xr-x 17 notroot notroot 4096 Jun 13 00:41 ..
drwxr-xr-x  6 notroot notroot 4096 Jun 13 00:38 env

(env)(notroot㉿kali)-[~/virutal-demo]
$ ls -laR | less

# -------------------------------------------------------------
# 6. Confronto dei percorsi dei binari (which python3 / which pip)
# -------------------------------------------------------------
# Dentro il venv (terminale sinistro):
(env)(notroot㉿kali)-[~/virutal-demo]
$ which python3
/home/notroot/virutal-demo/env/bin/python3

(env)(notroot㉿kali)-[~/virutal-demo]
$ which pip
/home/notroot/virutal-demo/env/bin/pip

# Nel sistema globale (terminale destro, per confronto):
(notroot㉿kali)-[~]
$ which python
/usr/bin/python

(notroot㉿kali)-[~]
$ which python3
/usr/bin/python3

# -------------------------------------------------------------
# 7. Confronto pacchetti installati (pip freeze)
# -------------------------------------------------------------
# Nel sistema globale (terminale destro): contiene dozzine di pacchetti di sistema
(notroot㉿kali)-[~]
$ pip freeze | less
gitdb==4.0.5
GitPython==3.1.12
gpg===1.14.0-unknown
graphene==2.1.7
...

# Dentro il venv (terminale sinistro): completamente pulito, lista vuota!
(env)(notroot㉿kali)-[~/virutal-demo]
$ pip freeze | less

# -------------------------------------------------------------
# 8. Installazione pacchetto isolato ed esecuzione con successo
# -------------------------------------------------------------
(env)(notroot㉿kali)-[~/virutal-demo]
$ pip install pwntools

# Verifica della presenza nel venv:
(env)(notroot㉿kali)-[~/virutal-demo]
$ pip freeze | less
# Ora compaiono: pwntools==4.5.1, capstone, unicorn, ropgadget, ecc.

(env)(notroot㉿kali)-[~/virutal-demo]
$ python3
>>> from pwn import *
>>> exit()
# Importazione riuscita senza errori!

# -------------------------------------------------------------
# 9. Disattivazione dell'ambiente (deactivate)
# -------------------------------------------------------------
(env)(notroot㉿kali)-[~/virutal-demo]
$ deactivate

# Il prompt perde il prefisso '(env)' e si torna all'interprete di sistema:
(notroot㉿kali)-[~/virutal-demo]
$ which python3
/usr/bin/python3
```
# Recap: sys, requests, pwntools

Panoramica rapida delle funzionalità principali, dei casi d'uso e dei comandi più comuni per `sys`, `requests` e `pwntools`.

---

## 1. `sys` (Libreria Standard di Sistema)

Modulo integrato in Python (non necessita di installazione con `pip`) che consente l'interazione diretta con l'interprete e l'ambiente operativo dell'host.

### Funzionalità Principali
* **Argomenti da riga di comando (`sys.argv`):**
  * `sys.argv[0]`: percorso/nome dello script eseguito.
  * `sys.argv[1:]`: lista dei parametri passati da terminale.
* **Controllo del flusso di esecuzione (`sys.exit()`):**
  * Interrompe lo script restituendo un exit code al sistema operativo (`sys.exit(0)` per successo, codici diversi da 0 o messaggi per errore).
* **Canali di I/O standard a basso livello:**
  * `sys.stdin`: canale di input standard.
  * `sys.stdout`: canale di output standard.
  * `sys.stderr`: canale per messaggi di errore e diagnostica.
* **Risoluzione dei percorsi dei moduli (`sys.path`):**
  * Lista di directory in cui l'interprete cerca i file e i pacchetti da importare (modificabile dinamicamente a runtime).

### Esempio Pratico
```python
import sys

# Controllo degli argomenti da terminale
if len(sys.argv) < 2:
    sys.stderr.write("Uso: python3 script.py <target_ip>\n")
    sys.exit(1)

target = sys.argv[1]
print(f"[+] Target impostato: {target}")
```
### 2. `requests.md`
```markdown
# requests — HTTP for Humans

Libreria client di terze parti (`pip install requests`) impiegata per effettuare comunicazioni HTTP/HTTPS con server web, REST API e applicazioni in modo dichiarativo e leggibile.

---

### Funzionalità Principali

* **Metodi HTTP Completi:**
  * Implementa chiamate native per tutti i verbi dello standard: `get()`, `post()`, `put()`, `delete()`, `head()`, `patch()`.
* **Invio Parametri e Dati di Payload:**
  * Query parameters per URL: `params={"id": 1}`
  * Dati form/URL-encoded: `data={"username": "admin", "password": "pwd"}`
  * Payload strutturati JSON: `json={"key": "value"}`
* **Configurazione e Sessioni:**
  * Personalizzazione degli header di richiesta (es. `headers={"User-Agent": "MyScanner"}`).
  * Persistenza di cookie, stato e connessioni TCP tramite `requests.Session()`.
  * Supporto a timeout di rete (`timeout=5`) e autenticazione (`auth=('user', 'pass')`).
* **Ispezione Avanzata della Risposta:**
  * `response.status_code`: codice numerico di stato (es. 200, 404, 500).
  * `response.text`: corpo della risposta decodificato in stringa testuale.
  * `response.content`: payload grezzo in byte (ideale per file binari o immagini).
  * `response.json()`: parser automatico da stringa JSON a dizionario Python.

---

### Esempio Pratico

```python
import requests

url = "[https://httpbin.org/post](https://httpbin.org/post)"
headers = {"User-Agent": "SecurityProbe/1.0"}
payload = {"target": "10.10.10.1", "action": "scan"}

response = requests.post(url, json=payload, headers=headers, timeout=5)

if response.status_code == 200:
    risultato = response.json()
    print("[+] Richiesta inviata con successo. Risposta ricevuta:")
    print(risultato)
else:
    print(f"[-] Errore: HTTP {response.status_code}")
```
### 3. `pwntools.md`
```markdown
# pwntools — CTF & Exploit Development Framework

Framework avanzato di terze parti (`pip install pwntools`) espressamente concepito per accelerare lo sviluppo di exploit binari, il reverse engineering e la risoluzione di sfide Capture The Flag (CTF).

---

### Funzionalità Principali

* **Astrazione Unificata dell'I/O (Tubes):**
  * Fornisce gli stessi metodi di I/O (`send()`, `sendline()`, `recv()`, `recvuntil()`, `interactive()`) indistintamente per processi locali (`process('./vuln')`) e socket remoti su rete (`remote('ip', port)`).
* **Packing e Unpacking per Architettura (Endianness):**
  * Converte numeri interi in sequenze di byte e viceversa secondo l'architettura scelta: `p32()`, `p64()` (packing) e `u32()`, `u64()` (unpacking).
* **Pattern Ciclici (De Bruijn Sequences):**
  * Genera stringhe non ripetitive con `cyclic(n)` e calcola istantaneamente l'offset esatto di sovrascrittura di registri come `EIP` o `RIP` con `cyclic_find(val)`.
* **Analisi Binari ELF & Mitigazioni:**
  * Carica ed esamina file ELF (`elf = ELF('./target')`) per estrarre sezioni, tabelle GOT/PLT, simboli e controlli attivi (`checksec`: NX, ASLR/PIE, Canary).
* **Strumenti CLI Inclusi:**
  * Utility da riga di comando pronte all'uso: `checksec`, `cyclic`, `asm`, `disasm`, `ROPgadget`.

---

### Esempio Pratico

```python
from pwn import *

# Impostazione dell'architettura e livello di log
context(os='linux', arch='amd64', log_level='info')

# Target di rete (o processo locale: io = process('./vuln'))
io = remote('127.0.0.1', 1337)

# Calcolo payload per buffer overflow
offset = 40
rip_target = 0x0000000000401156  # Indirizzo di una funzione target (es. win())

payload = b"A" * offset + p64(rip_target)

# Invio e passaggio alla shell interattiva
io.sendlineafter(b"Inserisci input: ", payload)
io.interactive()
```
