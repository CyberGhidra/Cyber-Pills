# Open-Source Intelligence (OSINT) Fundamentals
## Hunting Breached Passwords — Part 1: Analisi Operativa

Questa sessione illustra come gli analisti OSINT e di threat intelligence individuano credenziali compromesse (*breached credentials*) e password collegate a target specifici attraverso database di leak pubblici e strumenti di decifrazione hash.

---

### 1. Interrogazione di Credenziali Compromesse su DeHashed

Nella prima e seconda schermata viene impiegato il motore di ricerca specializzato **[DeHashed](https://dehashed.com/)**:

* **Che cos'è DeHashed:** Un motore di ricerca di leak e data breach progettato per analisti di sicurezza, investigatori e team di prevenzione frodi. Raccoglie miliardi di record esposti provenienti da violazioni note, pastebin e dump pubblici.
* **Tipi di Ricerca Eseguiti:**
  1. **Ricerca per Identità Specifica (`bob@gmail.com bobrocks123`):**  
     Verifica mirata per accertare se un indirizzo e-mail specifico compaia nei database insieme a una precisa stringa o password nota.
  2. **Domain Scan (`@tesla.com`):**  
     Inserendo `@dominio.com` (oppure usando l'operatore/filtro di scansione dominio), il tool estrae tutti gli account aziendali di quel perimetro compromessi in incidenti di sicurezza.
* **Informazioni Estratte nei Risultati:**
  * **Fonte del breach:** Ad esempio la dicitura *"Sourced from ShareThis data"* indica esattamente quale fornitore o servizio ha subito l'esfiltrazione.
  * **Metadati dell'account:** L'anteprima del record mostra `Name`, `Email` (es. `george@tesla.com`), e il campo `Username` o identificativo univoco. Spesso nei record sono visibili anche gli **hash delle password** associate a quell'utenza.

---

### 2. Risoluzione degli Hash delle Password su Hashes.org

Nella terza schermata il flusso operativo passa alla fase di cracking/lookup dell'impronta crittografica:

* **Contesto:** Quando i database compromessi non espongono la password in testo chiaro ma conservano l'hash (o stringhe codificate/saltate come `bztdOA...`), l'analista preleva la stringa cifrata.
* **Operazione eseguita su Hashes.org:**
  * La stringa hash viene incollata nel modulo **Search Hashes**.
  * Si seleziona l'opzione di verifica (inclusa la risoluzione del CAPTCHA) per interrogare un database massivo di corrispondenze precalcolate (*reverse lookup / rainbow lookup*).
  * L'obiettivo è verificare se l'algoritmo (MD5, SHA-1, SHA-256, NTLM, ecc.) e la relativa password in chiaro (*plain text*) siano già stati risolti dalla community globale senza dover eseguire calcoli computazionali in locale.

---

### Flusso Metodologico Riassunto

```text
[ Target: Dominio o E-mail ]
            │
            ▼
┌───────────────────────────────┐
│     Ricerca su DeHashed       │ ──> Individuazione dell'account, della fonte del leak
└───────────────────────────────┘     e dell'eventuale hash associato
            │
            ▼
┌───────────────────────────────┐
│     Lookup su Hashes.org      │ ──> Ricerca dell'hash nel database precalcolato
└───────────────────────────────┘
            │
            ▼
[ Password in Chiaro Identificata ]
