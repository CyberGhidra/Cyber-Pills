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

```
# Open-Source Intelligence (OSINT) Fundamentals
## Hunting Breached Passwords — Part 2: Piattaforme, Motori e API

Nella seconda parte dedicata all'investigazione su credenziali compromesse, l'attenzione si sposta sui database di dump centralizzati, sui motori di aggregazione di violazioni (*breach data engines*) e sull'automazione delle ricerche tramite API e sintassi avanzata (come Lucene).

---

### 1. Panoramica delle Piattaforme Citate

* **[WeLeakInfo](https://weleakinfo.to/v2/)**
  * Storico aggregatore di leak (la versione originale fu sequestrata dalle forze dell'ordine nel 2020; varie re-indicizzazioni e cloni operano su TLD alternativi).
  * Consente di cercare correlazioni tra e-mail, username, hash e password in chiaro esposte in vecchi dump.

* **[LeakCheck](https://leakcheck.io/)**
  * Motore di threat intelligence e monitoraggio breach con database costantemente aggiornato.
  * Offre sia un'interfaccia web sia API dedicate per analisti di sicurezza per verificare se credenziali aziendali o personali sono finite in raccolte recenti di stealer log o dump pubblici.

* **[SnusBase](https://snusbase.com/)**
  * Database avanzato indicizzato per indagini OSINT e red teaming.
  * Permette ricerche incrociate partendo non solo dall'indirizzo e-mail, ma anche da IP, hash di password, username, nomi completi o numeri di telefono.

* **[HaveIBeenPwned](https://haveibeenpwned.com/)**
  * Il servizio pubblico di riferimento globale gestito da Troy Hunt.
  * **Caratteristica etica fondamentale:** Conferma *se* un account è stato compromesso e *in quale violazione*, ma **non mostra mai la password in chiaro** né l'hash, limitandosi a indicare la fonte del breach e i tipi di dati esposti (es. "Passwords, IP addresses, Email addresses").

---

### 2. Focus su Scylla.sh: Ricerche Avanzate e API

Dalla schermata si osserva l'interfaccia di **Scylla.sh**, un database pubblico open-source di credenziali esposte:

#### A. Cosa restituisce la piattaforma (Password e Campi)
Come mostrato nel risultato a video, la ricerca per prefisso restituisce direttamente:
* **`email`**: L'account associato (es. `shark@tesla.com`, `shark@mail.ru`).
* **`domain`**: La collezione / dump da cui proviene il dato (es. `Collections`).
* **`password`**: La password direttamente leggibile / estratta dal leak (nel caso mostrato: `907DaDE814`).

#### B. Sintassi di Ricerca Lucene (`Queries`)
Scylla utilizza la sintassi di interrogazione Apache Lucene, che supporta wildcard e filtri mirati sui campi:
* **Ricerca per prefisso / campo specifico:**  
  * `email:shark*` — cerca tutte le email che iniziano per "shark".
  * `password:ff*` — cerca qualsiasi record la cui password inizi per "ff".
* **Wildcard avanzate su più campi:**  
  * `name:da?e password:*d*` — individua nomi utente come *dave, dale, dane* combinati a password contenenti il carattere "d".

#### C. Interfaccia API per Scripting e Automazione
Sotto la sezione **API** della schermata, la piattaforma espone la modalità di interrogazione programmatica[cite: 6]:
* **Chiamata HTTP GET:**  
  È possibile effettuare richieste GET impostando l'header `Accept: application/json`[cite: 6].
* **Esempio di endpoint mostrato:**
  ```text
  GET [https://scylla.sh/search?q=your_lucene_query&size=100&start=200](https://scylla.sh/search?q=your_lucene_query&size=100&start=200)
