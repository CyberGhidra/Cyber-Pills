# Guida Tecnica FortiOS: Antivirus Scanning Engine, Architetture di Ispezione e Protocol Options

---

## 1. Funzionamento del Motore Antivirus (AV Engine)

Il modulo **AntiVirus** di FortiOS analizza il flusso dati in transito (Layer 7) alla ricerca di pattern malevoli, trojan, ransomware, virus e payload sconosciuti.

```
                         ┌─────────────────────────────────────────┐
                         │   FortiGuard Antivirus Cloud Service    │
                         └────────────────────┬────────────────────┘
                                              │ Aggiornamenti Firme
                                              │ e Query Real-Time
                                              ▼
[ Traffico Inbound/Outbound ] ──► [ FortiGate AV Engine ] ──► [ Traffico Ispezionato ]
```

### Componenti Fondamentali
1. **Database delle Firme FortiGuard**:
   - Firme costantemente aggiornate tramite il cloud **FortiGuard Antivirus Service**.
   - Disponibile in diverse varianti (*Extended Database*, *Extreme Database* a seconda del modello di apparato e dei vincoli hardware).
2. **Aggiornamenti in Tempo Reale**:
   - I database vengono sincronizzati via rete **FDN (FortiGuard Distribution Network)** via push/pull su canale cifrato TLS (porta TCP 443).

---

## 2. Ordine Sequenziale di Scansione (Order of Scan)

Quando un file o un payload attraversa il motore AV, FortiOS esegue l'ispezione seguendo una pipeline a 3 fasi sequenziali ordinate dalla più veloce ed economica in termini computazionali alla più avanzata:

```
                  ┌──────────────────────┐
                  │ File / Stream Packet │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │  1. Antivirus Scan   │ ──► [Match Firma Noto?] ──► BLOCCO IMMEDIATO
                  └──────────┬───────────┘
                             │ No
                             ▼
                  ┌──────────────────────┐
                  │   2. Grayware Scan   │ ──► [Match PUP / PUA?]  ──► BLOCCO / MONITOR
                  └──────────┬───────────┘
                             │ No
                             ▼
                  ┌──────────────────────┐
                  │     3. AI Scan       │ ──► [Anomalia / Zero-Day?] ──► BLOCCO / SANDBOX
                  └──────────┬───────────┘
                             │ File Pulito
                             ▼
                  [ Inoltro al Destinatario ]
```

### Le 3 Fasi Dettagliate
1. **Antivirus Scan (Firme Tradizionali)**:
   - Primo livello di controllo statico ad altissima velocità.
   - Confronta l'impronta (hash/pattern) con il database delle firme note. Se il file è malevolo, la sessione viene interrotta immediatamente.
2. **Grayware Scan (PUP / PUA)**:
   - Secondo livello mirato a intercettare *Potentially Unwanted Programs* (adware, cryptominer nascosti, spyware, remote control tool non autorizzati).
   - Rileva software che non distrugge direttamente il sistema ma viola la privacy aziendale o consuma risorse di rete/calcolo.
3. **AI Scan (Scansione Machine Learning / Euristica)**:
   - Terzo livello ad intelligenza artificiale integrato nel kernel e nel motore IPS/AV.
   - Analizza la struttura statica del codice (struttura PE, sezioni di memoria, stringhe sospette) tramite modelli statistici e predittivi addestrati dai FortiGuard Labs, rilevando varianti polimorfiche o minacce **Zero-Day** prima ancora del rilascio formale di una firma specifica.

---

## 3. Meccanismi di Protezione e Feed Dinamici

FortiOS estende le capacità difensive oltre le firme statiche integrando sorgenti dinamiche locali ed esterne:

| Meccanismo | Descrizione Tecnica |
| :--- | :--- |
| **Signature-based detection** | Metodo classico di identificazione basato sui pattern del database FortiGuard. |
| **Virus Outbreak Prevention (VOS)** | Servizio euristico in cloud che copre il gap temporale tra la scoperta di un malware globale e il rilascio del pacchetto firme ufficiale, bloccando le epidemie in corso. |
| **External Malware Blocklist** | Liste dinamiche di hash malevoli (MD5, SHA-1, SHA-256) importate periodicamente da feed esterni o Threat Intelligence di terze parti via HTTP/HTTPS. |
| **EMS Threat Feed** | Integrazione Fabric diretta con **FortiClient EMS**: riceve hash malevoli rilevati sugli endpoint aziendali per applicare blocchi perimetrali immediati e coordinati su tutti i firewall. |

---

## 4. Componenti Antivirus Avanzati

### Content Disarm and Reconstruction (CDR)
*Nota architetturale fondamentale*: **Richiede obbligatoriamente un profilo con Feature Set Proxy-based.**
- **Logica Zero-Trust sul file**: Anziché cercare solo codice malevolo noto, CDR presuppone che documenti come PDF e Microsoft Office siano intrinsecamente pericolosi.
- **Sanificazione attiva**: Isola ed elimina elementi attivi incorporati (macro VBA, JavaScript, hyperlink non verificati, oggetti OLE, embedded font).
- **Ricomposizione**: Ricostruisce il documento privo delle parti pericolose e lo consegna all'utente finale.
- **Funzionalità collegate**:
  - *Retain original file*: Archivia una copia originale non sanificata in quarantena/storage per scopi forensi.
  - *Insert cover page*: Inserisce un frontespizio informativo che avvisa l'utente della bonifica applicata.
  - *Allow transmission on error*: Definisce se consegnare il file originale in caso di mancata ricostruzione.

### Behavior-Based Detection
- Monitora il comportamento dinamico di file e processi in transito, rilevando sequenze di azioni anomale tipiche di exploit e packer evasivi.

### CIFS / SMB Scanning
- Ispezione del protocollo di condivisione file Windows (porta TCP 445).
- Fondamentale nelle architetture interne (ISFW - *Internal Segmentation Firewall*) per impedire il movimento laterale di ransomware tra share aziendali e file server.

### AI/ML e Sandbox Analysis
- Coordinamento automatico con sistemi di analisi dinamica avanzata (**FortiSandbox** e **FortiNDR**) per l'esecuzione del codice in ambienti isolati (sandboxing).

---

## 5. Integrazione Multi-Destinazione FortiSandbox

FortiOS supporta la configurazione contemporanea di due ambienti di Sandboxing:

```
                         ┌───────────────────────────────────────┐
                         │         FortiGate Firewall            │
                         └──────────────────┬────────────────────┘
                                            │ Invio Payload Sconosciuti
                   ┌────────────────────────┴────────────────────────┐
                   ▼                                                 ▼
     ┌────────────────────────────┐                    ┌────────────────────────────┐
     │   On-Premises FortiSandbox │                    │     FortiSandbox Cloud     │
     │   (Appliance VM / Server)  │                    │      (Servizio SaaS)       │
     └────────────────────────────┘                    └────────────────────────────┘
```

1. **On-Premises FortiSandbox**: Appliance hardware o virtuale dedicata installata nella LAN/DC aziendale.
2. **FortiSandbox Cloud**: Servizio SaaS scalabile gestito direttamente da Fortinet.
3. **Comportamento della configurazione**:
   - Entrambe le destinazioni possono essere abilitate contemporaneamente (`Both destinations can be enabled simultaneously`).
   - Con il selettore impostato su **auto**, FortiOS seleziona autonomamente il sistema designato come predefinito (*default*).

---

## 6. Architetture a Confronto: Flow-Based vs Proxy-Based

La scelta della modalità di ispezione definisce il modo in cui il traffico viene elaborato dallo stack TCP di FortiOS.

```
MODALITÀ FLOW-BASED (Singola Connessione Passante):
[ Client ] ────────────── SYN ──────────────► [ Server ]
[ Client ] ◄──────────── SYN-ACK ──────────── [ Server ]
[ Client ] ────────────── ACK ──────────────► [ Server ]
              ▲                          ▲
              └─────── [ FortiGate ] ────┘
                   (Ispezione in streaming)

MODALITÀ PROXY-BASED (Due Connessioni TCP Distinte):
[ Client ] ◄── Connessione 1 ──► [ FortiGate Proxy ] ◄── Connessione 2 ──► [ Server ]
                                  (Buffer in RAM)
```

### Tabella Comparativa

| Parametro | Flow-Based Inspection (Default) | Proxy-Based Inspection |
| :--- | :--- | :--- |
| **Sessioni TCP** | **1 sola connessione diretta** tra Client e Server. | **2 connessioni indipendenti** (Client $\leftrightarrow$ FortiGate $\leftrightarrow$ Server). |
| **Processo di Ispezione** | **Streaming**: pacchetto per pacchetto tramite IPS/AV Engine. | **Full Buffering**: il file viene riassemblato interamente in memoria RAM (demone `wad`). |
| **Prestazioni e Latenza** | **Massime prestazioni**, latenza nell'ordine dei microsecondi, minor consumo RAM. | Latenza maggiore, throughput ridotto, elevato consumo di RAM/CPU. |
| **Gestione Ultimo Pacchetto** | Trattiene l'ultimo pacchetto TCP; se infetto invia un **TCP RST** e scarta il frame finale. | Trattiene l'intero payload; blocca la sessione o restituisce una pagina di blocco applicativa HTTP. |
| **Funzionalità Esclusive** | Antivirus, Web Filter, App Control ad altissima velocità. | **CDR**, Video Filter avanzato, manipolazione header HTTP, DLP avanzato. |

---

## 7. Stream-Based Antivirus Scanning in Modalità Proxy

Nei profili Proxy-based moderni, FortiOS imposta di fabbrica la modalità **Stream-Based Proxy Scanning**:
- **Scopo**: Consente al motore proxy di esaminare e decomprimere archivi voluminosi (*large archive files*) "al volo" durante il transito, senza dover allocare prima l'intero volume del file in memoria RAM.
- **Formati Supportati**: `ZIP`, `GZIP`, `BZIP2`, `TAR`, `ISO 9660`.
- **Protocolli Supportati**: HTTP/HTTPS, FTP/FTPS, SFTP/SCP.

### Gestione da CLI (Default vs Legacy)
```fortios
config antivirus profile
    edit "P_Proxy_AV"
        set feature-set proxy
        set scan-mode default    <-- Stream-based proxy attivo (RAM ottimizzata)
        # set scan-mode legacy   <-- Disabilita lo streaming: impone il caricamento completo in RAM
    next
end
```

---

## 8. Applicazione nelle Firewall Policy e Vincoli di Compatibilità

La GUI di FortiOS garantisce la consistenza architetturale tra regole di firewall e profili UTM:

1. **Firewall Policy in modalità Flow-based**:
   - Il menu a discesa della policy mostrerà **esclusivamente** i profili Antivirus configurati con feature set *Flow-based*.
   - I profili *Proxy-based* vengono automaticamente filtrati e nascosti per evitare conflitti di pipeline.
2. **Firewall Policy in modalità Proxy-based**:
   - La policy può agganciare **sia profili Flow-based che profili Proxy-based**.
   - Abilita il modulo esclusivo **Video Filter**.

```fortios
config firewall policy
    edit 1
        set name "LAN_to_WAN"
        set srcintf "port1"
        set dstintf "port2"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set utm-status enable
        set ssl-ssh-profile "certificate-inspection"
        
        # Selezione Modalità di Ispezione
        set inspection-mode flow-based     # oppure proxy
        set av-profile "default"
    next
end
```

---

## 9. Configuring Protocol Options (Mappatura Porte e Opzioni Avanzate)

Il profilo **Protocol Options** (`Policy & Objects > Protocol Options`) governa il comportamento dei protocolli applicativi scansionati dai motori UTM (AntiVirus, DLP, Web Filter).

### Protocol Port Mapping
Permette di specificare le porte su cui intercettare i vari protocolli (anche su porte non standard):

```
HTTP  : 80
SMTP  : 25
POP3  : 110
IMAP  : 143
FTP   : 21, 222, 23    <-- Supporto porte multiple separate da virgola
NNTP  : 119
MAPI  : 135
DNS   : 53
CIFS  : 445
```

> **Regola d'Oro per la Certificazione**: Il reindirizzamento e la personalizzazione delle porte tramite *Protocol Port Mapping* ha valore operativo ed efficacia **esclusivamente quando la Firewall Policy opera in modalità Proxy-based**. In modalità Flow-based, FortiOS intercetta il traffico basandosi sulla classificazione applicativa standard.

### Opzioni Comuni e Web
- **Comfort Clients**: 
  - Risolve il problema del timeout di sessione nei browser durante il download di file di grandi dimensioni.
  - Mentre il firewall sta ancora scansionando il file in memoria, invia al client byte lenti/fittizi a intervalli regolari, evitando che il browser chiuda la connessione per inattività (*Prevents transmission failures*).
- **Log Oversized Files**:
  - Registra nei log di sistema quando un file supera la soglia massima dimensionale di scansione supportata dal motore UTM.
- **Block Oversized File / Email**:
  - Blocca preventivamente i file o le e-mail che superano la memoria di buffer assegnata, impedendo che payload troppo grandi transitino non ispezionati.
- **Chunked Bypass (Web Options)**:
  - Consente o blocca il bypass per il traffico HTTP che utilizza la codifica di trasferimento a blocchi (*Chunked Transfer Encoding*).
