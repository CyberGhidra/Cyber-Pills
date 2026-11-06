# FortiOS: Architettura del Web Filtering & SSL/SSH Inspection

---

## 1. Posizionamento nel Ciclo di Vita del Pacchetto (Packet Lifecycle)

Il **Web Filtering** opera a **Livello 7 (Application Layer)** dello stack OSI. Non interviene all'inizio della sessione: il traffico deve prima essere instradato e autorizzato esplicitamente a livello Layer 3/Layer 4 dalla tabella delle policy.

```text
[ ARRIVO FRAME/PACCHETTO ]
           │
           ▼
[ 1. ROUTE LOOKUP (FIB) ] ────────► Determina l'interfaccia di uscita (egress)
           │
           ▼
[ 2. FIREWALL POLICY CHECK ] ─────► Ricerca sequenziale top-to-bottom
           │                        - Se MATCH (Action: ACCEPT) ──► Session Table creata
           │                        - Se MATCH (Action: DENY)   ──► Drop immediato
           ▼
[ 3. SSL INSPECTION ENGINE ] ─────► Lettura SNI/SAN (Certificate Inspection)
           │                        oppure Decifratura in RAM (Deep Inspection)
           ▼
[ 4. WEB FILTERING ENGINE (L7) ] ─► Controllo Categorie FortiGuard / URL Statici
           │
           ├─► Azione BLOCK/DENY  ──► Connessione troncata (RST/Block Page)
           └─► Azione ALLOW/PASS  ──► Inoltro verso la destinazione
```

### Regola d'Oro d'Esame:
Affinché il Web Filtering possa ispezionare il traffico HTTPS (che costituisce ormai oltre il 95% del traffico web mondiale), la Firewall Policy deve associare congiuntamente:
1. Un profilo **Web Filter**.
2. Un profilo **SSL/SSH Inspection** (`certificate-inspection` oppure `custom-deep-inspection`).

---

## 2. Identificazione del Dominio in HTTPS (SSL Certificate Inspection)

Nel traffico crittografato TLS/SSL, la risorsa puntuale (Path e URI, ad esempio `/login.php?user=admin`) è cifrata e inaccessibile senza Full Inspection. Tuttavia, il FortiGate estrae l'**FQDN (Fully Qualified Domain Name)** analizzando i metadati scambiati in chiaro durante l'handshake crittografico iniziale.

```text
Client Browser                                             FortiGate (In-Line)                                        Web Server
      │                                                           │                                                       │
      │ ──── 1. Client Hello (SNI: www.fortinet.com) ───────────► │ ──── [ESTRAZIONE 1: Campo SNI in chiaro] ──────────► │
      │                                                           │                                                       │
      │ ◄─── 2. Server Hello + Certificate (X.509) ────────────── │ ◄─── [ESTRAZIONE 2 (Fallback): SAN o CN] ───────────── │
      │                                                           │                                                       │
      │                                                           ├─► Query Locale / FortiGuard Cloud Rating              │
      │                                                           ├─► Match Categoria / Blacklist / Whitelist             │
      │                                                           │                                                       │
      │ ◄─── [Se Categoria Bloccata: Invio TCP RST o Block Page] ─┤                                                       │
```

### I Due Passaggi di Ispezione dell'Handshake:
1. **Passo Primario — Server Name Indication (SNI):**
   * Nel pacchetto `Client Hello`, il client include l'estensione TLS **SNI**, che contiene in chiaro il nome di dominio richiesto (es. `Server Name: www.fortinet.com`).
   * Il FortiGate estrae il dato istantaneamente, prima ancora che venga scambiata qualsiasi chiave o certificato.
2. **Passo Secondario (Fallback) — Certificato Server X.509:**
   * Se il client è obsoleto o non include l'SNI, FortiOS attende il pacchetto `Server Hello` proveniente dal server web.
   * Il firewall legge i campi pubblici del certificato: prioritariamente il **SAN** (*Subject Alternative Name*) e successivamente il **CN** (*Common Name* nel *Subject*), ricavando il dominio (es. `CN = *.fortinet.com`).

---

## 3. Matrice Architetturale: Flow-Based vs. Proxy-Based

La scelta della modalità di ispezione definisce come la CPU e la memoria allocano le risorse e gestiscono i socket TCP.

```text
MODALITÀ FLOW-BASED (Streaming in-line):
[ Client ] ────────────── [ FortiGate (IPS Engine / Direct Flow) ] ────────────── [ Server ]
                     (1 Singola Connessione TCP End-to-End passante)

MODALITÀ PROXY-BASED (Dual Socket / WAD Daemon):
[ Client ] ◄── Connessione TCP 1 ──► [ FortiGate / WAD Proxy ] ◄── Connessione TCP 2 ──► [ Server ]
```

| Criterio Tecnico | Flow-Based Inspection | Proxy-Based Inspection |
| :--- | :--- | :--- |
| **Architettura TCP** | **Singola sessione TCP diretta** passante tra client e server. | **Due sessioni TCP distinte**: Client $\leftrightarrow$ FortiGate e FortiGate $\leftrightarrow$ Server. |
| **Motore di Elaborazione** | Gestito principalmente dall'**IPS Engine** (e chip ASIC CP9). | Gestito dal demone proxy di sistema **WAD** (*Web Application Daemon*). |
| **Buffer dei Dati** | Analisi in streaming al volo (overhead di memoria minimo). | Buffer completo del payload in memoria RAM prima dell'analisi. |
| **Requisito Hardware** | Supportato su **tutti i modelli** FortiGate. | Vincolo: **Modelli con più di 2 GB di RAM** (disabilitato sui modelli entry-level). |
| **Latenza & Throughput** | Latenza quasi nulla, velocità vicina al wire-speed. | Latenza superiore introdotta dal riassemblaggio TCP e buffering. |
| **Funzionalità Esclusive** | Classificazione FortiGuard, SafeSearch, Static URL. | Log keyword di ricerca, Google Tenant Restriction, Stripping Header/ActiveX/Java/Cookie, Server SNI Check rigoroso. |

---

## 4. Configurazione del Profilo SSL/SSH Inspection

Percorso: `Security Profiles > SSL/SSH Inspection`.

### Parametri Chiave:
* **Enable SSL inspection of:** `Multiple Clients Connecting to Multiple Servers`  
  Identifica lo scenario **Forward Proxy / Outbound**: protegge i client della rete interna che accedono a risorse eterogenee su Internet.
* **Inspection Method:** `SSL Certificate Inspection`  
  Non decifra il payload applicativo. Non necessita dell'installazione del certificato di root CA (*Fortinet_CA_SSL*) sui dispositivi client.
* **Protocol Port Mapping:**  
  Specifica le porte Layer 4 sulle quali FortiOS intercetta il traffico HTTPS per analizzare i certificati (Default: `443, 10443`). È possibile indicare porte multiple separate da virgola.
* **Server certificate SNI check:**  
  Verifica la corrispondenza tra l'SNI dichiarato nel `Client Hello` e il CN/SAN presente nel certificato del server per prevenire attacchi di domain fronting/spoofing.  
  *Vincolo Tecnico:* Funzione attiva **esclusivamente su policy in modalità Proxy-based**.

### Gestione degli Errori e Meccanismi di Protezione:
* **Untrusted SSL certificates (Block):** Tronca la connessione se il certificato del server non è firmato da una CA valida presente nel trusted store di FortiOS.
* **Malicious certificates (Block):** Blocca i certificati associati a siti noti per malware o frodi secondo l'intelligence FortiGuard.
* **Encrypted Client Hello - ECH (Block):** Blocca la sessione se il browser utilizza standard ECH/ESNI. Poiché l'ECH cifra l'estensione SNI verso il server DNS/web, impedirebbe al firewall di visualizzare il dominio; l'azione di blocco forza il browser a degradare a TLS standard o impedisce bypass non controllati.

---

## 5. Azioni sulle Categorie FortiGuard

All'interno di `Security Profiles > Web Filter`, FortiGuard Labs suddivide i siti in macro-categorie e sotto-categorie. Per ciascuna di esse è possibile definire una delle seguenti 5 azioni:

```text
[ Query Categoria Sito ]
           │
           ├─► ALLOW        ──► Inoltro trasparente (Nessun log generato di default)
           ├─► MONITOR      ──► Inoltro del traffico + Generazione evento nei log di sicurezza
           ├─► BLOCK        ──► Sessione terminata + Invio "Web Filter Block Page"
           ├─► WARNING      ──► Pagina di avviso: l'utente può cliccare "Proceed"
           └─► AUTHENTICATE ──► Captive Portal: sblocco subordinato a credenziali di gruppo
```

1. **Allow (Consenti):**
   * Il traffico è consentito senza interferenze. Per preservare lo spazio su disco e non saturare l'I/O del database di logging, di default questa azione non genera voci nei log UTM.
2. **Monitor (Monitora):**
   * Il traffico è autorizzato esattamente come con *Allow*, ma FortiOS genera una voce esplicita nei **Security Logs** (`subtype="webfilter"`). Essenziale durante i test pilota per definire la baseline di navigazione aziendale.
3. **Block (Blocca):**
   * La connessione viene troncata. Al browser dell'utente viene inviata una pagina HTML personalizzabile di sistema (*Replacement Message*) che riporta l'URL, la categoria FortiGuard incriminata e il motivo del drop.
4. **Warning (Avviso):**
   * Presenta una schermata intermedia di avvertimento. Se l'utente clicca sul pulsante di conferma (*Proceed*), FortiOS crea un bypass temporaneo per quell'IP/utente e consente l'accesso per un lasso di tempo configurabile (es. 15 minuti).
5. **Authenticate (Autentica):**
   * Blocca la normale navigazione mostrando un modulo di autenticazione (*Override*). Solo gli utenti appartenenti a specifici gruppi locali o Active Directory/LDAP autorizzati possono superare il blocco inserendo le proprie credenziali.

---

## 6. Funzionalità Avanzate Esclusive: Proxy-Based Web Filtering

Nel momento in cui la Firewall Policy e il profilo Web Filter operano con **Feature set: Proxy-based** (sui modelli con RAM > 2 GB), si sbloccano funzionalità avanzate a livello applicativo:

### 1. Log All Search Keywords
Intercetta e registra nei log di sistema le query testuali digitate dagli utenti nei motori di ricerca supportati (Google, Bing, Yahoo!, Yandex).

### 2. Restrict Google Account Usage to Specific Domains
Previene l'accesso ad account personali (es. `@gmail.com`) da reti aziendali forzando l'uso della Google Workspace aziendale:
* Il FortiGate inietta un header HTTP aggiuntivo nelle richieste verso i server Google:
  ```http
  X-GoogApps-Allowed-Domains: azienda.com, partner.com
  ```

### 3. Rimozione Componenti Attivi (Proxy Options)
Il motore WAD analizza il codice HTML/JS prima di consegnarlo al client ed elimina fisicamente le inclusioni di codice potenzialmente insicure:
* **Remove Java Applets:** Neutralizza i tag `<applet>` e gli archivi `.jar`.
* **Remove ActiveX:** Neutralizza gli oggetti ActiveX (`<object>`, `<embed>`) tipici di ambienti legacy.
* **Remove Cookies:** Rimuove gli header `Set-Cookie` per impedire il tracciamento della sessione.

---

## 7. Struttura del Log di Web Filtering (Analisi Campo per Campo)

Esempio di log reale generato da un blocco FortiGuard:

```text
date=2026-04-29 time=07:29:09 eventtime=1754576949412373904 tz="-0700" logid="0316013056" type="utm" subtype="webfilter" eventtype="ftgd_blk" level="warning" vd="root" policyid=1 poluuid="5476b1d8-8be7-51ef-d3c8-3e0539bb2ec0" policytype="policy" sessionid=35060 srcip=10.0.11.50 srcport=46830 srccountry="Reserved" srcintf="port4" dstip=151.101.67.5 dstport=80 dstcountry="United States" dstintf="port2" proto=6 httpmethod="GET" service="HTTP" hostname="cnn.com" profile="default" action="blocked" reqtype="direct" url="http://cnn.com/" sentbyte=501 rcvdbyte=0 direction="outgoing" msg="URL belongs to a denied category in policy" ratemethod="domain" cat=36 catdesc="News and Media"
```

### Decodifica dei Parametri di Diagnostica:
* `type="utm" subtype="webfilter"`: Identifica l'evento generato dai profili di sicurezza unificati (L7), nello specifico dal motore Web Filter.
* `eventtype="ftgd_blk"`: Indica che la causa scatenante è una regola della categorizzazione cloud FortiGuard (*FortiGuard Block*).
* `level="warning"`: Severità di Livello 4 (evento anomalo o traffico respinto).
* `policyid=1`: ID della Firewall Policy che conteneva il profilo di sicurezza che ha generato il drop.
* `sessionid=35060`: Indice della connessione registrata nella Stateful Session Table in RAM.
* `srcip=10.0.11.50` / `srcport=46830`: Socket di rete sorgente dell'endpoint richiedente (LAN su `port4`).
* `dstip=151.101.67.5` / `dstport=80`: Socket del web server remoto attestato su WAN (`port2`).
* `hostname="cnn.com"` / `url="http://cnn.com/"`: FQDN e risorsa richiesta.
* `cat=36` / `catdesc="News and Media"`: ID numerico e nome della categoria secondo la classificazione FortiGuard.
* `action="blocked"` / `msg="URL belongs to a denied category in policy"`: Esito applicato dal motore di sicurezza (abbattimento del canale TCP).

---

## 8. Diagnostica e Verifica da Riga di Comando (CLI Cheat-Sheet)

```fortios
# 1. Verifica dello stato dei server cloud FortiGuard e del tempo di risposta (RTT)
diagnose test updateinfo
diagnose debug rating

# 2. Test manuale di classificazione di un dominio FQDN dalla CLI
diagnose webfilter fortiguard-rating <fqdn>
# Esempio: diagnose webfilter fortiguard-rating www.cnn.com

# 3. Visualizzazione della cache locale delle categorie (URL Cache)
diagnose webfilter urlcache dump

# 4. Debug in tempo reale del motore WAD (Proxy-based)
diagnose debug reset
diagnose debug application wad 255
diagnose debug enable

# 5. Debug in tempo reale del motore IPS/Flow (Flow-based)
diagnose debug reset
diagnose debug application urlfilter 255
diagnose debug enable
```
