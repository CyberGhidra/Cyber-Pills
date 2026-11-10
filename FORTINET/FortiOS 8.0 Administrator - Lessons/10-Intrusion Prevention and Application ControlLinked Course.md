# Intrusion Prevention System (IPS) & Application Control

---

## 1. Architettura del Motore IPS (IPS Engine)

Il modulo **Intrusion Prevention System (IPS)** di FortiOS è progettato per rilevare e bloccare in tempo reale exploit su vulnerabilità note, violazioni degli standard di protocollo (RFC) e attacchi applicativi. L'analisi opera in modalità **Flow-Based Detection**, scansionando i pacchetti in streaming al passaggio attraverso le interfacce senza trattenere arbitrariamente l'intero payload prima dell'elaborazione.

```
                  ┌─────────────────────────────────────────────────────────────┐
                  │                    MOTORE IPS (FortiOS)                     │
                  │                                                             │
                  │  ┌──────────────────────┐         ┌──────────────────────┐  │
                  │  │  Protocol Decoders   │         │    IPS Signatures    │  │
                  │  │ (HTTP, DNS, SMB...)  │         │   Database (FDN)     │  │
                  │  └──────────┬───────────┘         └──────────┬───────────┘  │
                  │             │                                │              │
                  │             └───────────────►◄───────────────┘              │
                  │                             │                               │
                  │                  ┌──────────▼──────────┐                    │
                  │                  │     IPS Engine      │                    │
                  │                  │(Matching Real-Time) │                    │
                  │                  └──────────┬──────────┘                    │
                  └─────────────────────────────┼───────────────────────────────┘
                                                │
       [ PACCHETTO DI RETE ] ───────────────────┴───────► [ AZIONE: ALLOW / BLOCK / RST ]
```

### I Tre Componenti Cardine
1. **IPS Signature Database (Database delle firme):**
   * Collezione costantemente aggiornata di signature binarie e pattern euristiche gestite dai **FortiGuard Labs**.
   * Identifica exploit verso vulnerabilità software note (CVE), malware di rete e codice arbitrario remoto.
2. **Protocol Decoders (Decodificatori di Protocollo):**
   * Moduli software specializzati nell'analisi sintattica dei singoli protocolli (HTTP, DNS, SMB, FTP, DCE-RPC, TLS, ecc.).
   * Verificano che il traffico rispetti rigorosamente gli standard definiti dalle RFC.
   * "Scartano" i layer di incapsulamento ed estraggono i singoli campi del pacchetto (es. URI, Header HTTP, record DNS) per sottoporli al motore di confronto.
3. **IPS Engine:**
   * Il processo software/hardware centrale orchestratore dell'ispezione.
   * Esegue la correlazione logica in tempo reale tra i campi decodificati e i pattern del database, applicando le policy di mitigazione a latenza ultrabassa.

---

## 2. Flusso Operativo del Traffico (Life of Packet IPS)

```
[ Attaccante Esterno ] ──► (1. Vettore Exploit) ──► [ FORTIGATE (IPS Attivo) ]
                                                            │
                     ┌──────────────────────────────────────┴──────────────────────────────────────┐
                     ▼                                                                             ▼
          [ Match Firma / Anomalia ]                                                     [ Traffico Legittimo ]
                     │                                                                             │
                     ▼                                                                             ▼
           (3. Drop / TCP RST)                                                            (4. Inoltro Pulito)
                     │                                                                             │
            Attaccante Bloccato                                                       Destinazioni Protette:
           (Connessione Troncata)                                                    • Sedi e Uffici LAN
                                                                                     • Server & Data Center
```

1. **Tentativo di Intrusione:** Un host remoto invia pacchetti contenenti buffer overflow, injection SQL o anomalie di protocollo diretti alle risorse aziendali.
2. **Intercettazione:** Il traffico transita attraverso la Firewall Policy su cui è agganciato il sensore IPS. I decodificatori estraggono i dati e il motore li confronta con il database.
3. **Mitigazione:** Al rilevamento del pattern di attacco, FortiGate scarta i pacchetti malevoli (**Drop**) o interrompe la sessione inviando un reset (**TCP RST**).
4. **Protezione:** Solo il traffico conforme e privo di pattern di minaccia raggiunge le destinazioni interne protette.

---

## 3. Gestione e Classificazione del Database Firme

* **Percorso GUI:** `Security Profiles > Intrusion Prevention`
* **Pulsante `View IPS Signatures`:** Permette di accedere alla vista complessiva delle decine di migliaia di firme attive sull'apparato.

### Metriche di Classificazione (Dashboard a Grafico)
* **Severity (Gravità dell'impatto):** Valuta l'indice di rischio dell'exploit associato:
  $$\text{Critical} \longrightarrow \text{High} \longrightarrow \text{Medium} \longrightarrow \text{Low} \longrightarrow \text{Information}$$
* **Target (Bersaglio):** 
  * `Server`: Attacchi diretti contro servizi in ascolto (porte aperte esposte su Internet o DMZ: HTTP, SQL, SMB).
  * `Client`: Attacchi che sfruttano vulnerabilità software sul lato client (browser, visualizzatori PDF, client di posta) quando l'utente contatta un server malevolo.
* **OS (Sistema Operativo):** Cataloga le firme in base all'ambiente bersaglio (es. Windows, Linux, macOS, Solaris, Cisco IOS).

### Struttura della Tabella Firme (Anatomia di una Firma)
| Campo | Descrizione Operativa |
| :--- | :--- |
| **Name** | Identificativo testuale univoco (es. `3Com.3CDaemon.FTPServer.Buffer.Overflow`, `HTTP.URI.SQL.Injection`). |
| **Severity** | Livello di criticità assegnato dal team di threat intelligence. |
| **Target** | Ambito di applicazione (`Server` o `Client`). |
| **OS** | Piattaforma o architettura software bersaglio dell'attacco. |
| **Action** | Azione predefinita assegnata da FortiGuard: **Block** (scarto immediato) per minacce critiche/alte, **Pass** per regole informative. |
| **CVE-ID** | Codice univoco internazionale dello standard MITRE (es. `CVE-2005-0277`) per il cross-referencing con i bollettini di sicurezza mondiali. |

---

## 4. Configurazione dei Sensori IPS

* **Percorso GUI:** `Security Profiles > Intrusion Prevention` $\rightarrow$ `Create New` (es. sensore denominato `IPS_profile`).
* **Sezione `IPS Signatures and Filters`:** Tramite il pulsante `+ Create New`, si sceglie la modalità di inclusione mediante il selettore `Type`.

```
                    ┌───────────────────────────────────────────────┐
                    │               Tipo di Regola                  │
                    └───────┬───────────────────────────────┬───────┘
                            │                               │
                            ▼                               ▼
               [ Type: Signature (Puntuale) ]   [ Type: Filter (Dinamico) ]
               • Selezione manuale per nome/CVE • Criteri: Severity, OS, Target
               • Eccezioni e tuning mirato      • Include automaticamente le
               • Override dei parametri standard   nuove firme rilasciate da FDN
```

### Metodo 1: Firme Individuali (`Type: Signature`)
Consente di selezionare puntualmente singole firme tramite ricerca per testo o codice CVE.
* **Action:** Sovrascrive l'azione predefinita (`Default`, `Pass`, `Block`, `Reset`).
* **Packet Logging:** Abilita (`Enable` / `Disable`) la cattura e memorizzazione dell'esatto payload di pacchetto che ha innescato l'evento.
* **Status:** Stato operativo (`Enable`, `Disable`, `Default`).
* **Rate-based settings:** Abilita contromisure volumetriche o di soglia temporale.
* **Exempt IPs:** Elenco di indirizzi IP esclusi dall'ispezione della regola (utile per scanner di vulnerabilità interni autorizzati).

### Metodo 2: Gruppi Dinamici tramite Filtri (`Type: Filter`)
Consente di aggregare le firme in base a insiemi di metadati (es. `Target = Server`, `Severity = Medium, High, Critical`, `Protocol = HTTP`, `OS = Windows`).
* **Vantaggio Operativo (Zero-Touch):** Tutte le future firme rilasciate da FortiGuard che soddisfano i criteri configurati vengono aggiunte dinamicamente al sensore senza richiedere l'intervento dell'amministratore.

---

## 5. Rate-Based Signatures (Mitigazione Brute-Force & DoS)

A differenza delle firme tradizionali (che intervengono alla prima corrispondenza binaria), le **Rate-Based Signatures** monitorano la frequenza temporale degli eventi, attivando la contromisura solo al superamento di una soglia prestabilita.

```
       Evento 1        Evento 2             Evento N (Threshold raggiunta)
          │               │                                │
          ▼               ▼                                ▼
───[ Pacchetto ]───►[ Pacchetto ]─── ... ───►[ Pacchetto / Attacco ]───► [ AZIONE DI BLOCCO ]
   └──────────────────────── Finestra Temporale (Duration) ────────────────────────┘
```

### Parametri di Configurazione (`Specify`)
1. **Threshold (Soglia):** Numero massimo di occorrenze consentite prima di applicare l'azione di blocco.
2. **Duration (Secondi):** Intervallo temporale mobile entro cui conteggiare le occorrenze (es. 60 secondi).
3. **Track By (Criterio di Tracciamento):**
   * `Source IP`: Tiene traccia delle occorrenze per singolo indirizzo IP sorgente. Ideale per contrastare port scan e attacchi brute-force.
   * `Destination IP`: Tiene traccia delle occorrenze indirizzate a una specifica destinazione, proteggendo il server bersaglio da attacchi distribuiti a bassa intensità.
   * `Any`: Conteggia qualsiasi occorrenza aggregata corrispondente alla firma.

---

## 6. Ordine di Ispezione del Sensore (Inspection Sequence)

La tabella delle regole all'interno di un sensore IPS opera secondo il principio **Top-to-Bottom**:

```
                       ┌──────────────────────────────────────────────┐
                       │           TRAFFICO IN ENTRATA                │
                       └──────────────────────┬───────────────────────┘
                                              │
                                              ▼
                       ┌──────────────────────────────────────────────┐
                       │ Regola 1: Firma Specifica (Apache Tomcat)    │
                       │           Azione: MONITOR                    │
                       └──────┬────────────────────────────────┬──────┘
                              │                                │
                       [ Corrispondenza ]                 [ No Match ]
                              │                                │
                              ▼                                ▼
                     Azione Eseguita:               ┌──────────────────────────────────────────────┐
                     Registra Log e Passa           │ Regola 2: Filtro Dinamico (Server Windows)   │
                     (Fine Analisi Firma)           │           Azione: DEFAULT (BLOCK)            │
                                                    └──────────────────┬───────────────────────────┘
                                                                       │
                                                                       ▼
                                                            Continua Sequenza...
```

* **Priorità del Matching:** I pacchetti vengono valutati riga per riga partendo dall'alto. Quando un pacchetto corrisponde a una riga, il firewall applica l'azione associata e arresta l'ispezione per quella firma.
* **Inserimento Nuove Regole:** Ogni nuova riga creata viene inserita **automaticamente in fondo alla lista**.
* **Best Practice:** Posizionare sempre le eccezioni e le firme specifiche (es. una firma impostata su `Monitor` per test o tuning) **sopra** i filtri dinamici generici, per evitare che questi ultimi blocchino preventivamente il traffico.

---

## 7. Azioni del Motore IPS & Packet Logging

### Azioni Disponibili
| Azione | Comportamento Operativo del Firewall |
| :--- | :--- |
| **Allow** | Consente il transito del pacchetto malevolo ignorando la corrispondenza (nessun log generato). |
| **Monitor** | Autorizza il transito del pacchetto e genera contestualmente una registrazione nei log di sistema. |
| **Block** | Scarta immediatamente il pacchetto e termina la sessione di rete attiva in memoria. |
| **Reset** | Interrompe la connessione inviando un pacchetto TCP RST (`RST-Client`, `RST-Server` o entrambi), forzando l'immediata chiusura dei socket. |
| **Default** | Applica l'azione raccomandata da FortiGuard Labs (generalmente `Block` per High/Critical, `Pass` per Info). |
| **Quarantine** | Blocca il pacchetto e inserisce l'indirizzo IP della sorgente nella lista di quarantena (ban temporaneo o permanente su ogni flusso del firewall). |

### Funzionalità Packet Logging
* **Scopo:** Se abilitato, FortiOS acquisisce l'esatto flusso binario (header e payload) del pacchetto che ha scatenato la firma.
* **Analisi Forense:** L'amministratore può scaricare il file catturato in formato standard `.pcap` per aprirlo su analizzatori di pacchetti (come Wireshark), verificando se si tratti di un tentativo reale di exploit o di un falso positivo applicativo.

---

## 8. Analisi Forense dei Log IPS

* **Percorso GUI:** `Log & Report > Security Events` $\rightarrow$ Card `Intrusion Prevention`.

### Struttura del Log IPS e Campi di Diagnostica
Analisi dell'evento di sicurezza rilevato dal sistema:

```
date=2026-11-10 time=09:40:12 logid="0419016384" type="utm" subtype="ips" 
level="alert" severity="high" srcip=100.65.0.254 srcport=49152 
dstip=10.0.1.100 dstport=80 proto=6 action="dropped" policyid=3 
poluuid="2b2a647e-8c91-51ef-ba34-118239bb4fc1" attack="HTTP.URI.SQL.Injection" 
attackid=15621 profile="WEBSERVER" direction="incoming" msg="web_misc: HTTP.URI.SQL.Injection" 
ref="https://fortiguard.fortinet.com/encyclopedia/ips/15621"
```

### Decodifica dei Parametri di Contesto
* **`severity="high"` / `Threat Score: 30`:** Grado di rischio dell'attacco secondo i modelli FortiGuard.
* **`proto=6`:** Valore standard IANA per il protocollo di trasporto Layer 4 **TCP** ($6 = \text{TCP}$, $17 = \text{UDP}$, $1 = \text{ICMP}$).
* **`action="dropped"`:** Conferma dell'avvenuta neutralizzazione del pacchetto sul perimetro.
* **`policyid=3` / `poluuid`:** Identifica in modo deterministico la Firewall Policy responsabile del transito.
* **`profile="WEBSERVER"`:** Sensore IPS associato alla regola di firewall.
* **`attack="HTTP.URI.SQL.Injection"` / `attackid=15621`:** Pattern identificato e relativo Attack ID univoco.
* **`direction="incoming"`:** Vettore dell'attacco orientato dall'esterno verso la rete protetta.
* **`ref`:** Collegamento diretto all'Enciclopedia delle Minacce FortiGuard contenente le CVE correlate e la documentazione del payload malevolo.

---

## 9. Application Control: Riconoscimento a Livello 7

Il modulo **Application Control** opera in sinergia strutturale con il motore IPS di FortiOS:

```
                            [ Flusso di Traffico (Layer 7) ]
                                           │
                                           ▼
                     ┌───────────────────────────────────────────┐
                     │          IPS ENGINE (Flow-Based)          │
                     ├─────────────────────┬─────────────────────┤
                     │   Modulo IPS        │ Application Control │
                     │   (Exploit/Firme)   │   (Decodifica L7)   │
                     └─────────────────────┴─────────────────────┘
                                           │
                ┌──────────────────────────┴──────────────────────────┐
                ▼                                                     ▼
     Controllo su Porte Famose                             Controllo Applicativo Reale
    (es. TCP 80, 443, 8080)                               (TikTok, Skype, BitTorrent, P2P)
    Bypassato da porte non standard                       Ispezione comportamentale indipendente dalla porta
```

### Principi Operativi
1. **Condivisione del Motore IPS:** Application Control **utilizza lo stesso IPS engine** in modalità flow-based scan. Questo garantisce scansioni in streaming con impatto minimo su CPU e RAM.
2. **Ispezione a Livello 7 (Application Layer):** Riconosce le applicazioni di rete decodificando il traffico indipendentemente dalle porte TCP/UDP utilizzate (ad esempio, traffico non-web o protocolli remoti mascherati su porta TCP 443).
3. **Gestione dei Protocolli P2P:** Identifica pattern dinamici ed euristici tipici del traffico Peer-to-Peer e dei protocolli distribuiti, aggirando le limitazioni dei firewall Layer 3/4 tradizionali.
4. **Consultazione del Database:**
   * **Percorso GUI:** `Security Profiles > Application Control` $\rightarrow$ Pulsante `View Application Signatures`.
   * Permette di consultare il catalogo completo delle firme applicative (con granularità su singole sotto-funzioni quali download, upload o login) divise per popolarità e classe di rischio.
