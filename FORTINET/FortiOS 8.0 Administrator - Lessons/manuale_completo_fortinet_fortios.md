# Fortinet FortiOS: Manuale Operativo Completo e Architettura di Sicurezza

Questo documento raccoglie in un unico testo organico i concetti architetturali, i flussi operativi, le configurazioni da interfaccia grafica (GUI) e i comandi CLI analizzati durante il corso Fortinet.

---

## Indice dei Contenuti
1. [Architettura di Piattaforma: SPU, FortiOS e Security Fabric](#1-architettura-di-piattaforma-spu-fortios-e-security-fabric)
2. [Configurazione di Base: Modalità Operative, Interfacce e Routing](#2-configurazione-di-base-modalità-operative-interfacce-e-routing)
3. [Amministrazione, Profili di Accesso (RBAC) e FortiAI](#3-amministrazione-profili-di-accesso-rbac-e-fortiai)
4. [Source NAT (SNAT), Destination NAT (DNAT) e PAT](#4-source-nat-snat-destination-nat-dnat-e-pat)
5. [Internet Services Database (ISDB) e Routing dei Servizi Cloud](#5-internet-services-database-isdb-e-routing-dei-servizi-cloud)
6. [Firewall Policy: Oggetti, Tag Dinamici e Security Profiles](#6-firewall-policy-oggetti-tag-dinamici-e-security-profiles)
7. [Logging e Integrazione FortiAnalyzer](#7-logging-e-integrazione-fortianalyzer)
8. [Autenticazione Firewall: Locale, LDAP, RADIUS e FortiToken (2FA)](#8-autenticazione-firewall-locale-ldap-radius-e-fortitoken-2fa)
9. [FSSO (Fortinet Single Sign-On): Architettura e Funzionamento](#9-fsso-fortinet-single-sign-on-architettura-e-funzionamento)
10. [FSSO vs ZTNA (Zero Trust Network Access)](#10-fsso-vs-ztna-zero-trust-network-access)
11. [FortiIdentity Cloud (FIC) e Flussi SAML](#11-fortidentity-cloud-fic-e-flussi-saml)
12. [FSSO Deployment e Tuning Operativo (Collector, Timers, Filtri)](#12-fsso-deployment-e-tuning-operativo-collector-timers-filtri)

---

## 1. Architettura di Piattaforma: SPU, FortiOS e Security Fabric

L'infrastruttura Fortinet adotta un modello hardware/software integrato per garantire throughput elevato e bassa latenza.

```
+-------------------------------------------------------------------------+
|                  FortiGuard Subscription Services                       |
|     (Threat Intelligence, Firme AV/IPS, GeoIP, Categorie Web/DNS)      |
+------------------------------------+------------------------------------+
                                     │ (Aggiornamenti FDN / Live Queries)
                                     ▼
+-------------------------------------------------------------------------+
|                   Motori di Sicurezza FortiOS (L7)                      |
|       [ NGFW ]   [ AntiVirus ]   [ Web Filter ]   [ IPS ]   [ DLP ]     |
+-------------------------------------------------------------------------+
|                    FortiOS (Sistema Operativo Unificato)                |
|           Routing, Switching, Stateful Firewall, VPN, IPAM              |
+-------------------------------------------------------------------------+
|                   Secure Processing Units (SPU)                         |
|   NP (Network Processor)       CP (Content Processor)      SoC (System) |
+-------------------------------------------------------------------------+
```

### Componenti Fondamentali
1. **SPU (Secure Processing Units):** Chip ASIC proprietari dedicati all'offloading hardware.
   * **NP (Network Processor):** Accelera il routing di rete, le sessioni firewall di base e la terminazione tunnel IPsec.
   * **CP (Content Processor):** Accelera le operazioni crittografiche (SSL/TLS inspection) e la scansione dei contenuti per AntiVirus e IPS.
   * **SoC (System on a Chip):** Combina CPU standard, NP e CP in un singolo silicio (utilizzato nelle serie Entry-Level).
2. **FortiOS:** Il sistema operativo unificato che governa l'intero ecosistema, garantendo coerenza di gestione tra apparati fisici, virtuali (VM) e istanze cloud.
3. **Servizi FortiGuard (FDN):** Rete di threat intelligence globale che fornisce:
   * *Package Updates:* Download schedulato di firme AV e motori IPS su porta TCP 443.
   * *Live Queries:* Interrogazioni in tempo reale per Web Filtering e DNS Filtering (porte UDP 53/8888 o HTTPS 443).
4. **Security Fabric:** Interconnessione tra firewall e moduli esterni:
   * **FortiClient EMS:** Gestione centrale degli endpoint, conformità e telemetria.
   * **FortiSandbox:** Analisi dinamica di malware zero-day in ambienti isolati.
   * **FortiWeb:** Web Application Firewall per la protezione di server HTTP/HTTPS.
   * **FortiMail:** Sicurezza specializzata per il traffico email (anti-spam, anti-phishing).

---

## 2. Configurazione di Base: Modalità Operative, Interfacce e Routing

### 2.1 Modalità Operative del Firewall
* **NAT / Route Mode (Default):** Il firewall opera a Livello 3 (Network). Esegue routing tra subnet distinte, assegna indirizzi IP alle interfacce, gestisce server DHCP e applica traslazioni NAT.
* **Transparent Mode:** Il firewall opera a Livello 2 (Data Link). Viene inserito come un bridge trasparente all'interno della medesima subnet IP senza richiedere modifiche all'indirizzamento esistente.

### 2.2 Primo Accesso Hardware
* **Modelli Low/Mid Level (es. FortiGate 40F, 60F):** Le porte interne (es. `port1`) sono impostate di fabbrica come client DHCP per ricevere un IP dalla rete a monte.
* **Modelli Mid/High Level (es. FortiGate 100F e superiori):** Dispongono di una porta di gestione dedicata (`MGMT`) con indirizzo statico predefinito **`192.168.1.99/24`**, accesso HTTPS/SSH abilitato, utente `admin` e password vuota.

### 2.3 Configurazione Interfacce (`Network > Interfaces`)
* **Role:** Adatta graficamente la visualizzazione delle opzioni disponibili (`LAN`, `WAN`, `DMZ`, `Undefined`). Se impostato su *LAN*, espone i parametri per il server DHCP locale; se impostato su *WAN*, nasconde le funzioni DHCP server.
* **Alias:** Etichetta descrittiva per identificare immediatamente il ruolo della porta (es. `port1 = internal_lan`).
* **Administrative Access:** Protocolli di gestione ammessi sull'interfaccia (`HTTPS`, `SSH`, `PING`, `SNMP`, `Security Fabric Connection`, `FMG-Access`).
* **Supporto LLDP:** Consente la ricezione (`Receive LLDP`) e trasmissione (`Transmit LLDP`) di messaggi di adiacenza a Livello 2 per rilevare apparati di rete collegati.

### 2.4 Rotta Statica Predefinita (`Network > Static Routes`)
Per consentire l'uscita su Internet a tutta la rete locale è obbligatorio creare una rotta predefinita:
* **Destination:** `0.0.0.0/0.0.0.0`
* **Gateway:** Indirizzo IP del router di confine o gateway dell'ISP.
* **Interface:** Interfaccia rivolta all'esterno (es. `wan1`).
* **Administrative Distance:** `10` (valore predefinito su FortiOS per le rotte statiche).

---

## 3. Amministrazione, Profili di Accesso (RBAC) e FortiAI

### 3.1 Profili Amministrativi (`System > Admin Profiles`)
I profili stabiliscono i privilegi operativi assegnati a ciascun utente amministratore.
* **Livelli di autorizzazione per modulo:** `None`, `Read`, `Read & Write`, `Custom`.
* **Permessi CLI:** Possibilità di abilitare o disabilitare l'esecuzione di comandi interattivi (`get`, `show`, `diagnose`).
* **Override Idle Timeout:** Definisce il tempo massimo di inattività della sessione prima della disconnessione forzata.
* **Gerarchia dei ruoli tipica:**
  1. *Super Admin:* Controllo globale illimitato su tutti i VDOM e impostazioni di sistema.
  2. *Security / Network Admin:* Accesso in lettura/scrittura a policy, oggetti e configurazioni di routing.
  3. *Help Desk / Operatori L1:* Accesso in sola lettura e monitoraggio sessioni.
  4. *Auditor:* Accesso limitato alla consultazione dei registri di audit e log.

### 3.2 Restrizione degli Accessi Amministrativi
È consigliato limitare le connessioni amministrative a subnet o indirizzi IP specifici (*Trusted Hosts*):

```fortios
config sys admin
    edit "mario_admin"
        set trusthost1 10.0.1.100 255.255.255.255
        set disallowed-login-methods telnet http
    next
end
```

### 3.3 FortiAI
Assistente integrato in FortiOS per supportare gli amministratori nell'analisi delle configurazioni e nella creazione di regole tramite linguaggio naturale:
* Richiede connettività cloud verso i servizi FortiCare o FortiGuard e privilegi amministrativi.
* Può essere integrato con API OpenAI esterne.
* Raccomandazione di sicurezza: evitare di trasmettere password, chiavi pre-condivise o dati sensibili all'interno dei prompt.

---

## 4. Source NAT (SNAT), Destination NAT (DNAT) e PAT

```
[ Client Locale ]                         [ FortiGate WAN ]                     [ Server Remoto ]
10.0.1.10:1025   ──► (Arrivo su LAN) ──►  70.70.70.70:1025   ──► (Transito) ──►  80.80.80.80:443
10.0.1.20:1025   ──► (Conflitto PAT) ──►  70.70.70.70:30192  ──► (Transito) ──►  80.80.80.80:443
```

### 4.1 Differenza tra SNAT e DNAT
* **Source NAT (SNAT):** Modifica l'indirizzo IP e la porta del mittente. Si attiva all'interno della *Firewall Policy* (`set nat enable`) per consentire la navigazione verso l'esterno mascherando la rete privata.
* **Destination NAT (DNAT):** Modifica l'indirizzo IP e la porta di destinazione. Si configura definendo un oggetto **Virtual IP (VIP)** da utilizzare come *Destination* nella regola di firewall per pubblicare server interni verso l'esterno (*Port Forwarding*).

### 4.2 Modalità Operative per il Source NAT
All'interno della Firewall Policy (`Policy & Objects > Firewall Policy`):
1. **Use Outgoing Interface Address:** Utilizza l'indirizzo IP assegnato alla porta WAN di uscita. Se più host interni generano sessioni concorrenti utilizzando la stessa porta sorgente, FortiOS applica il **PAT (Port Address Translation)**, allocando una nuova porta libera (es. `30192`) per prevenire collisioni.
2. **Use Dynamic IP Pool:** Alloca gli indirizzi pubblici da un insieme preventivamente creato (`Policy & Objects > IP Pools`), supportando configurazioni *Overload*, *One-to-One* o *Fixed Port Range*.

---

## 5. Internet Services Database (ISDB) e Routing dei Servizi Cloud

### 5.1 Definizione di ISDB
L'**ISDB (Internet Service Database)** è un archivio proprietario costantemente allineato dai database FortiGuard che elenca:
* Tutti gli indirizzi IP pubblici, le subnet, i protocolli e le porte Layer 4 utilizzati dai principali servizi globali (Microsoft 365, Amazon AWS, Google, Dropbox, Salesforce).
* Sottoservizi specifici (es. `Amazon-AWS.S3` o `Microsoft-Office365.Teams`).

### 5.2 Routing Statico Basato su ISDB
Consente di instradare il traffico destinato a piattaforme cloud su link dedicati senza dover creare e aggiornare manualmente centinaia di rotte statiche:
* **Configurazione (`Network > Static Routes`):**
  * Selezionare l'opzione **Internet Service** come destinazione.
  * Scegliere il servizio desiderato (es. `Amazon-AWS`).
  * Assegnare interfaccia e gateway dedicato.
* **Funzionamento nel Kernel:** FortiOS inserisce automaticamente nella tabella di inoltro (FIB) tutte le sottoreti associate all'ID del servizio selezionato, aggiornandole dinamicamente al rilascio di nuovi IP da parte del fornitore.

### 5.3 Geographic-Based ISDB
Permette di costruire oggetti ISDB personalizzati filtrando i servizi in base a parametri territoriali:
* **Percorso GUI:** `Policy & Objects > Internet Service Database > Create New`.
* **Parametri:** Impostare il tipo su *Geographic Based*, scegliere il servizio primario e definire `Country`, `Region` e `City`.
* **Caso d'uso:** Vincolare il traffico aziendale a transitare unicamente verso data center situati in territori conformi alle normative sulla sovranità del dato (es. GDPR / Unione Europea).

---

## 6. Firewall Policy: Oggetti, Tag Dinamici e Security Profiles

### 6.1 Struttura della Firewall Policy
* **Source Address (Obbligatorio):** Subnet, Host IP (/32), FQDN, Geographic, Dynamic Fabric Connector o ISDB.
* **User / Group (Opzionale):** Vincola il transito del pacchetto alla preventiva identificazione di un account locale, utente remoto o gruppo FSSO/ZTNA.
* **Destination Address:** Oggetti IP o FQDN di destinazione.
* **Service:** Protocolli e porte ammesse (es. `HTTP`, `HTTPS`, `DNS`).
* **Action:** `ACCEPT` o `DENY`.

### 6.2 Tag Dinamici e Dynamic Tag Address Groups
* **Configurazione Tag (`Policy & Objects > Tags`):** Creazione di tag descrittivi associabili agli oggetti di rete.
* **Gruppi Dinamici:** Aggregano automaticamente gli indirizzi IP che possiedono un determinato tag. Quando un nuovo server o host viene censito con quel tag, entra istantaneamente nel gruppo senza richiedere interventi o modifiche sulle Firewall Policy attive.

### 6.3 Security Profiles (Ispezione Layer 7)
Quando l'azione è impostata su `ACCEPT`, è possibile applicare profili di sicurezza:
* **AntiVirus (AV):** Scansione da malware su protocolli applicativi (HTTP, FTP, SMTP).
* **Web Filter:** Blocco basato su categorie FortiGuard, reputazione e URL filters.
* **DNS Filter:** Intercettazione preventiva delle query DNS prima dell'apertura della sessione IP.
* **Application Control:** Riconoscimento delle applicazioni indipendentemente dalla porta di transito.
* **IPS (Intrusion Prevention System):** Rilevamento e blocco in tempo reale di exploit e vulnerabilità software.
* **SSL Inspection:** Ispezione del traffico crittografato. Può essere *Certificate Inspection* (legge solo l'SNI e i metadati in chiaro) oppure *Deep Packet Inspection* (completa decifratura e ri-cifratura del flusso per consentire ai motori AV, DLP e IPS di analizzare il payload HTTPS).

---

## 7. Logging e Integrazione FortiAnalyzer

### 7.1 Archiviazione dei Log (Disk Logging vs Modelli Hardware)
* **Modelli con suffisso "0" (es. FortiGate 40F, 60F):** Non possiedono disco rigido interno; mantengono i log in memoria RAM temporanea (azzerata in caso di riavvio). Richiedono l'invio dei dati a FortiAnalyzer o server syslog esterni.
* **Modelli con suffisso "1" (es. FortiGate 41F, 61F, 101F):** Dotati di unità SSD/HDD interna per il salvataggio dei log in un database locale indicizzato.
* **Configurazione CLI retention locale:**
```fortios
config log disk setting
    set status enable
    set maximum-log-age 30
end
```

### 7.2 Architettura FortiAnalyzer: Analyzer Mode vs Collector Mode
* **Analyzer Mode (Predefinita):** Riceve i log, li indicizza nel database SQL, supporta query in tempo reale su FortiView e compila report grafici.
* **Collector Mode:** Funziona unicamente come aggregatore e buffer intermedio ad alta velocità per filiali distaccate; non indicizza tabelle SQL e non genera report (`No event management or reporting`). Inoltra i flussi all'Analyzer centrale.

### 7.3 Connessione tra FortiGate e FortiAnalyzer
1. **Sul FortiGate (`Security Fabric > Fabric Connectors > Logging & Analytics`):**
   * Impostare lo stato su *Enabled* e inserire l'indirizzo IP del FortiAnalyzer.
   * Selezionare la frequenza di invio: `Real Time`, `1-Minute` o `5-Minute` (default).
2. **Abilitazione Crittografia OFTPS (TCP 514):**
```fortios
config log fortianalyzer setting
    set status enable
    set server "10.0.13.125"
    set enc-algorithm high-medium
    set reliable enable
    set upload-option realtime
end
```
3. **Sul FortiAnalyzer (`Device Manager`):**
   * Individuare il firewall sotto *Unauthorized Devices*.
   * Selezionare l'apparato e confermare con il pulsante **Authorize**.

---

## 8. Autenticazione Firewall: Locale, LDAP, RADIUS e FortiToken (2FA)

### 8.1 Metodi di Autenticazione
* **Locale:** Credenziali salvate nel database interno del firewall (utenze amministrative e break-glass).
* **Remota:** Delega la convalida a directory centralizzate (LDAP o RADIUS).
* **Multifattore (2FA/MFA):** Combina password/PIN con credenziali monouso (TOTP/Push).

### 8.2 Protocollo LDAP vs RADIUS

| Parametro | LDAP (Directory Protocol) | RADIUS (AAA Protocol) |
| :--- | :--- | :--- |
| **Natura** | Navigazione di database ad albero gerarchico | Controllo accessi alla rete (AAA) |
| **Cosa fa FortiGate** | Esegue bind/search, sfoglia l'albero ed estrae gruppi | Invia `Access-Request` e riceve una decisione (Permesso/Negato) |
| **Porte di rete** | TCP 389 (Clear/STARTTLS), TCP 636 (LDAPS) | UDP 1812 (Auth), UDP 1813 (Acct) |
| **Mappatura Gruppi** | Nativa tramite percorso DN (`OU=...,DC=...`) | Tramite attributi proprietari VSA (*Fortinet-Group-Name*) |
| **Accounting** | Non supportato | Supportato nativamente (durata sessione e byte scambiati) |

### 8.3 Configurazione e Diagnostica CLI per Server Remoti
* **Verifica Autenticazione LDAP:**
```bash
diagnose test authserver ldap <nome_server_ldap> <username> <password>
```
* **Verifica Autenticazione RADIUS:**
```bash
diagnose test authserver radius <nome_server_radius> <pap|chap|mschapv2> <username> <password>
```

### 8.4 Configurazione FortiToken e Modalità di Risposta
* Di fabbrica ogni FortiGate include **2 licenze FortiToken Mobile gratuite**.
* **Modalità Passiva (TOTP):** L'utente apre l'applicazione mobile, legge il codice numerico a 6 cifre e lo digita a video.
* **Modalità Attiva (Push):** Il firewall invia una notifica push sullo smartphone; l'utente convalida con un tocco su *Approve*. (Richiede la porta di servizio FTM abilitata sull'interfaccia WAN).

```fortios
# Registrazione Licenza
config user fortitoken
    edit "<CODICE_ATTIVAZIONE>"
    next
end

# Assegnazione Utente Locale con 2FA
config user local
    edit "mario.rossi"
        set type password
        set passwd "PasswordComplessa123!"
        set two-factor fortitoken
        set fortitoken "<NUMERO_SERIALE_TOKEN>"
        set email-to "mario.rossi@azienda.com"
    next
end
```

---

## 9. FSSO (Fortinet Single Sign-On): Architettura e Funzionamento

**FSSO (Fortinet Single Sign-On)** implementa un'**autenticazione trasparente (passiva)** eliminando i portali di blocco per l'utente finale:
* L'utente effettua il regolare logon al proprio computer collegato al dominio Microsoft Active Directory o Novell eDirectory.
* Il software FSSO intercetta l'evento nei registri di sistema e associa tre parametri:
  1. **Username** (account dell'utente).
  2. **IP Address** (indirizzo della postazione di lavoro).
  3. **Group Membership** (gruppi di sicurezza o OU di appartenenza).
* L'associazione viene trasmessa al FortiGate, che concede l'accesso alle risorse in base ai gruppi FSSO configurati nelle policy di sicurezza.

---

## 10. FSSO vs ZTNA (Zero Trust Network Access)

### 10.1 Tabella di Confronto Dettagliata

| Parametro | FSSO (Fortinet Single Sign-On) | ZTNA (Zero Trust Network Access) |
| :--- | :--- | :--- |
| **Sorgente di Identità** | Eventi di logon Active Directory / LDAP | FortiClient EMS o FortiClient EMS Cloud |
| **Principio Operativo** | Mappa `IP + Utente/Gruppo` in modo trasparente | mTLS esplicito con verifica di Security Posture Tags |
| **Agente Endpoint** | **Nessun agente** sul dispositivo client | **FortiClient** obbligatorio a bordo del computer |
| **Dati ricevuti da FortiGate** | Username AD + appartenenza ai gruppi | Certificato client, identità utente, posture tags |
| **Policy Enforcement** | Regole standard di Firewall Policy per gruppi | ZTNA Firewall Policy o ZTNA Proxy Policy |
| **Device Posture** | **Non valutato** (*Not evaluated*) | **Verificato costantemente** (antivirus, patch, BitLocker) |
| **Ambito di Rete** | Principalmente utenti interni (**on-net**) | Utenti interni (**on-net**) ed esterni/remoti (**off-net**) |

### 10.2 Criteri di Selezione Operativa
* **Adotta FSSO quando:**
  * L'infrastruttura è basata su rete locale aziendale gestita tramite Active Directory.
  * È richiesto accesso trasparente senza impatto software sulle singole postazioni.
  * Sono presenti dispositivi personali (BYOD) o sistemi non-Windows non gestibili centralmente.
* **Adotta ZTNA quando:**
  * L'azienda adotta un paradigma *Zero Trust* ("Non fidarti mai, verifica sempre").
  * Si desidera sostituire le tradizionali VPN remote con connessioni sicure cifrate applicazione per applicazione (*per-application access*).
  * L'accesso deve essere revocato istantaneamente se il dispositivo contrae infezioni o perde i requisiti di conformità aziendali.

---

## 11. FortiIdentity Cloud (FIC) e Flussi SAML

### 11.1 Caratteristiche di FortiIdentity Cloud
* Servizio IAM (Identity and Access Management) cloud-delivered multi-tenant per la gestione di SSO e MFA.
* Implementazione priva di componenti hardware dedicati in sede (*zero-footprint*).
* Supporto a standard aperti (**SAML**, **OIDC**, **OAuth 2.0**) e provisioning dinamico utenti tramite standard **SCIM** o API.
* Modello di licenza in abbonamento per-utente, con quote cumulabili e allineamento date (*co-term*).

### 11.2 Flusso di Autenticazione SAML con FIC IDP Proxy

```text
[ Browser Client ]      [ FortiGate SP ]       [ FIC IDP Proxy ]     [ 3rd Party IDP ]     [ FIC MFA Service ]
        │                      │                       │                     │                      │
  1. Accesso risorsa protetta  │                       │                     │                      │
        ├─────────────────────►│                       │                     │                      │
  2. Redirect verso FIC        │                       │                     │                      │
        │◄─────────────────────┤                       │                     │                      │
        ├─────────────────────────────────────────────►│                     │                      │
  3. Reindirizzamento a IdP Esterno                    │                     │                      │
        │◄─────────────────────────────────────────────┤                     │                      │
        ├───────────────────────────────────────────────────────────────────►│                      │
  4. Inserimento Credenziali (Username/Password)       │                     │                      │
        │───────────────────────────────────────────────────────────────────►│                      │
  5. Rilascio asserzione SAML primaria (Username/Email)│                     │                      │
        │◄───────────────────────────────────────────────────────────────────┤                      │
        ├─────────────────────────────────────────────►│                     │                      │
        │                      │                       ├─ Verifica/Signup    │                      │
        │                      │                       ├─ Richiesta MFA ───────────────────────────►│
  6. Validazione Secondo Fattore (Push/OTP Mobile)     │                     │                      │
        │◄──────────────────────────────────────────────────────────────────────────────────────────┤
        ├─────────────────────────────────────────────►│                     │                      │
  7. Asserzione SAML Definitiva (MFA Passed)           │                     │                      │
        │◄─────────────────────────────────────────────┤                     │                      │
        ├─────────────────────►│                       │                     │                      │
  8. Concessione Accesso Rete  │                       │                     │                      │
        │◄─────────────────────┤                       │                     │                      │
```

---

## 12. FSSO Deployment e Tuning Operativo (Collector, Timers, Filtri)

### 12.1 Modalità di Deployment per Microsoft Active Directory

```text
+------------------------------------------------------------------------------------------------------+
|                                     ARCHITETTURE DI DEPLOYMENT FSSO                                  |
+------------------------------------+------------------------------------+----------------------------+
| 1. DC Agent Mode                   | 2. Polling Mode                    | 3. Terminal Server Agent   |
|    - Agente a bordo di ogni DC     |    a) Collector Agent-Based        |    - Agente a bordo di     |
|    - Intercetta i logon in tempo   |       - Server membro dedicato     |      server RDS/Citrix     |
|      reale (UDP 8002)              |       - Legge log eventi via WMI   |    - Alloca range di porte |
|    - Massima reattività su reti    |    b) Agentless (Direct)           |      TCP/UDP per singolo   |
|      estese                        |       - FortiGate interroga i DC   |      utente sullo stesso IP|
+------------------------------------+------------------------------------+----------------------------+
```

### 12.2 Configurazione Connettori Fabric su FortiOS (`Security Fabric > External Connectors`)
* **Agentless Polling:** Selezionare *Poll Active Directory Server*. Richiede IP del DC, credenziali amministrative con diritti di lettura sui Security Log e associazione a un server LDAP di sistema.
* **Collector Agent Mode:** Selezionare *FSSO Agent on Windows AD*. Richiede IP del Collector Agent e relativa chiave di autenticazione.
  * *User Group Source = Collector Agent:* I filtri e i gruppi vengono risolti dall'agente Windows.
  * *User Group Source = Local:* FortiGate interroga in autonomia il server LDAP locale per estrarre la struttura dei gruppi (*Proactively retrieve from LDAP server*).

### 12.3 Parametri Operativi e Timer del Collector Agent

| Parametro | Default | Descrizione Tecnica |
| :--- | :---: | :--- |
| **Listening Port FortiGate** | **TCP 8000** | Porta di ascolto per la comunicazione con FortiGate (porta **TCP 8001** se cifrata SSL). |
| **Listening Port DC Agent** | **UDP 8002** | Porta di ascolto per i pacchetti inviati dai DC Agent (porta **UDP 8003** se cifrata). |
| **Workstation verify interval** | **5 min** | Intervallo con cui il Collector contatta la postazione client tramite il servizio Windows *Remote Registry* per verificare se l'utente è ancora connesso. (Impostare su `0` per disabilitare). |
| **Dead entry timeout interval** | **480 min (8h)**| Tempo trascorso il quale una voce non confermata (client offline o non raggiungibile) viene rimossa (*purged*) dalla memoria. |
| **IP address change verify** | **60 sec** | Frequenza di verifica dei cambi IP delle postazioni; indispensabile in ambienti con DHCP dinamico. |
| **Cache user group lookup** | **60 min** | Durata di mantenimento in cache dei gruppi utente per evitare query continue verso Active Directory. |

### 12.4 Filtri di Gruppo e Liste di Esclusione
* **Ignored User List:** Impedisce al Collector di monitorare accessi generati da account di servizio, task schedulati o processi batch. L'esclusione può avvenire per nome manuale (supporta wildcard `*` e `?`), per selezione utente o per intera Unità Organizzativa (OU).
* **Group Filters:** Permettono di limitare i gruppi AD inviati a ciascun firewall associando i filtri al numero di serie univoco (**Serial Number**) del FortiGate ricevente.

### 12.5 Modalità AD Access Mode: Standard vs Advanced
Configurabile dal Collector Agent in *Set Directory Access Information*:

| Caratteristica | Standard Access Mode | Advanced Access Mode |
| :--- | :--- | :--- |
| **Notazione Nomi** | Sintassi NetBIOS classica (`Domain\Username` o `Domain\Group`) | Sintassi LDAP distinta (`CN=User, OU=Reparto, DC=Azienda, DC=local`) |
| **Oggetti Supportati** | Applicabile **esclusivamente a gruppi AD** | Applicabile a **singoli utenti, gruppi e intere Unità Organizzative (OU)** |
| **Gruppi Annidati** | **Non supportati** (*nested groups not supported*) | **Supportati** (risolve gruppi annidati ed ereditati) |
| **Interfaccia FortiGate** | Elenco piatto di gruppi inviato dal Collector | Albero gerarchico LDAP navigabile direttamente da GUI |

### 12.6 Fallback per Utenti Non Mappati
Se un utente si autentica regolarmente in dominio ma **non appartiene ad alcuno dei gruppi FSSO configurati sul FortiGate**:
* **Autenticazione passiva pura:** L'utente viene assegnato al gruppo predefinito **`SSO_Guest_Users`**, consentendo la creazione di regole per accessi ristretti o di cortesia.
* **Autenticazione mista:** Se la policy prevede l'integrazione attiva, il firewall presenta una schermata di Captive Portal forzando l'utente a inserire credenziali alternative per proseguire.