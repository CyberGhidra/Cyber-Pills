# Fortinet Single Sign-On (FSSO), ZTNA e FortiIdentity Cloud

Guida tecnica e appunti di studio per la preparazione e gestione delle identità in ambiente **FortiOS**.

---

## 1. Panoramica FSSO (Fortinet Single Sign-On)

### 1.1 Definizione di SSO
* **Cos'è:** Meccanismo di autenticazione che consente a un utente già identificato di accedere a molteplici applicazioni e risorse di rete senza doversi riautenticare ad ogni richiesta.
* **Vantaggio principale:** Elimina la necessità di reinserire continuamente username e password, aumentando l'usabilità e riducendo il carico di supporto IT.

### 1.2 Autenticazione Trasparente e Meccanismo FSSO
A differenza dell'autenticazione attiva tramite **Captive Portal** (in cui il firewall intercetta la sessione HTTP/HTTPS e presenta un form di login a video), FSSO opera in maniera **completamente trasparente o passiva**:

* **Nessun prompt per l'utente:** L'utente effettua il login standard al proprio PC Windows (o su directory LDAP) all'avvio della sessione lavorativa e accede alle risorse di rete senza interruzioni.
* **Dati identificativi intercettati da FSSO:** Il software FSSO associa e traccia tre parametri cardine:
  1. **Username:** Nome dell'account dell'utente.
  2. **IP address:** Indirizzo IP assegnato alla postazione/workstation.
  3. **Group membership:** Gruppi di sicurezza o directory a cui l'utente appartiene.

### 1.3 Ruolo del FortiGate e Mappatura dei Gruppi
* **Controllo d'accesso basato su gruppi:** FortiGate autorizza o nega il traffico in transito verificando l'appartenenza a specifici gruppi FSSO richiamati all'interno delle Firewall Policy.
* **Flessibilità di associazione:** I gruppi FSSO sul FortiGate possono essere mappati direttamente su:
  * Singoli utenti (*individual users*).
  * Gruppi di utenti (*user groups*).
  * Unità Organizzative (*Organizational Units - OUs*).
  * Una combinazione eterogenea di questi elementi.

### 1.4 Integrazione con Directory Services
FSSO si integra nativamente con servizi di directory centralizzati:
* **Microsoft Active Directory (AD):** Scenario primario e più diffuso; FSSO monitora i Domain Controller per analizzare i log di autenticazione Kerberos/NTLM.
* **Novell eDirectory.**

---

## 2. FSSO vs. ZTNA: Confronto e Scenari Operativi

### 2.1 Tabella Comparativa

| Parametro | FSSO (Fortinet Single Sign-On) | ZTNA (Zero Trust Network Access) |
| :--- | :--- | :--- |
| **Identity source** | Active Directory / Eventi di login LDAP | FortiClient EMS o FortiClient EMS Cloud |
| **How it works** | Mappa `Indirizzo IP + Utente/Gruppo` in modo trasparente | Mutual TLS (mTLS) esplicito + Tag di security posture |
| **Agent** | DC Agent, Collector Agent o SSOMA (nessun agente su endpoint) | FortiClient (obbligatorio su ogni endpoint) |
| **FortiGate receives** | Username AD + Appartenenza ai gruppi | Certificato del dispositivo, identità utente, security posture tag |
| **Policy enforcement** | Corrispondenza gruppi utente nelle Firewall Policy | ZTNA Firewall Policy o ZTNA Proxy Policy |
| **Device posture** | Non valutato (*Not evaluated*) | Controllato continuamente (antivirus, patch OS, crittografia, ecc.) |
| **Location scope** | Principalmente utenti on-net (rete locale) | Sia on-net che off-net (remoto e locale) |

### 2.2 Quando usare FSSO
FSSO è la scelta ideale quando l'obiettivo principale è la **comodità d'uso all'interno del perimetro di rete aziendale**:
* **Nessun impatto sull'endpoint:** Non richiede l'installazione né la manutenzione di software/agenti sulle singole macchine degli utenti (*no agent on every device*).
* **Autenticazione trasparente:** L'utente accede al PC la mattina e naviga senza interruzioni di Captive Portal.
* **Flessibilità BYOD:** Ideale in ambienti con postazioni non gestite, dispositivi personali (BYOD) o sistemi operativi non standard che non possono ospitare un agente aziendale.
* **Firewall Policy on-net:** Adatto quando il firewall deve semplicemente associare l'indirizzo IP locale ai gruppi di sicurezza di Active Directory per le regole di transito interne.

### 2.3 Quando usare ZTNA
ZTNA è indispensabile quando l'organizzazione adotta un modello **Zero Trust** (*"Never trust, always verify"*):
* **Controllo continuo dello stato di salute (Device Posture):** L'accesso è vincolato sia all'identità dell'utente sia alla conformità dell'endpoint (antivirus aggiornato, patch applicate, crittografia disco attiva). Se il PC contrae un malware o perde i requisiti di sicurezza, l'accesso viene revocato istantaneamente.
* **Sostituzione delle dial-up VPN (Smart Working sicuro):** Anziché aprire l'accesso all'intera subnet aziendale con una VPN tradizionale, ZTNA instaura tunnel mTLS cifrati applicazione per applicazione (*per-application access*), impedendo il movimento laterale di possibili attaccanti.
* **Sicurezza costante ovunque:** Applica la medesima postura e gli stessi controlli sia che il dipendente operi dalla sede centrale (*on-net*), sia che si trovi in smart working o mobilità (*off-net*).

---

## 3. FortiIdentity Cloud (FIC)

### 3.1 Definizione e Caratteristiche
* **Cos'è:** Servizio IAM (*Identity and Access Management*) completo erogato interamente in modalità Cloud (*cloud-delivered*).
* **Scopo principale:** Fornisce meccanismi avanzati di MFA (*Multi-Factor Authentication*) e SSO (*Single Sign-On*) sia per i dispositivi Fortinet che per applicazioni terze (SaaS o on-premise).
* **Zero-Footprint Deployment:** Non richiede hardware aggiuntivo, server dedicati on-premise o modifiche complesse alle liste di controllo accessi (ACL) della rete perimetrale.

### 3.2 I Tre Pilastri Funzionali

#### A. Metodi MFA (MFA Methods)
* **FortiToken Mobile OTP:** Codici temporizzati (TOTP) o notifiche push tramite app per smartphone.
* **Hardware Token (FTK):** Token fisici portatili Fortinet (es. serie FortiToken-200).
* **Email / SMS OTP:** Codici monouso recapitati via email o SMS.
* **FIDO2 Passkeys:** Autenticazione passwordless e resistente al phishing conforme allo standard FIDO2 (chiavi hardware USB o biometria di sistema).

#### B. Funzionalità IAM (IAM Features)
* **Local Identity Provider (IdP):** Autorità centrale di autenticazione e archivio utenti cloud.
* **IdP Proxy capability:** Intermediario tra le applicazioni e Identity Provider esterni/federati (Microsoft Entra ID, Okta, Google Workspace).
* **Adaptive Authentication:** Valutazione dinamica del rischio contestuale (geolocalizzazione, tipo di dispositivo, reputazione IP o anomalie comportamentali).
* **Admin & End-User Portals:** Portali web dedicati per la configurazione delle policy e per il self-service degli utenti (accesso SSO alle app e gestione dei token).

#### C. Integrazioni (Integrations)
* **Prodotti Fortinet nativi:** FortiGate, FortiAuthenticator, FortiPAM, FortiSandbox e FortiADC.
* **Protocolli Standard Aperti:** Supporto SAML, OIDC (*OpenID Connect*) e OAuth 2.0.
* **Provisioning Utenti:** Importazione da file CSV, chiamate API e provisioning automatico in tempo reale tramite standard SCIM (*System for Cross-domain Identity Management*).

### 3.3 Modello di Licenza
* **Subscription-based:** Modello su abbonamento cloud.
* **Per-user:** Costo calcolato sul numero di utenti/identità attive censite.
* **Time-based:** Contratti annuali o pluriennali.
* **Stackable with co-term renewals:** Licenze cumulabili nel tempo con allineamento delle scadenze su un'unica data annuale di rinnovo (*co-terming*).

---

## 4. Modalità di Deployment FSSO per Active Directory

FortiOS supporta integrazioni con Microsoft Windows AD, Novell eDirectory, Citrix, VMware Horizon e Microsoft Exchange. Per ambienti Microsoft AD, sono disponibili tre architetture:

```text
                           +----------------------------+
                           |  Modalità Deployment FSSO  |
                           +--------------+-------------+
                                          |
        +---------------------------------+---------------------------------+
        |                                 |                                 |
        v                                 v                                 v
+----------------+               +-----------------+               +-----------------+
| DC Agent Mode  |               |  Polling Mode   |               | TS Agent Mode   |
+----------------+               +--------+--------+               +-----------------+
| Agente su ogni |                        |                        | Citrix/RDS con  |
| Domain Control.|         +--------------+-------------+          | porte TCP/UDP   |
| Real-time push |         |                            |          | per ogni utente |
+----------------+         v                            v          +-----------------+
                 +-------------------+        +--------------------+
                 | Collector Polling |        | Agentless Polling  |
                 | (Server membro)   |        | (FortiGate diretto)|
                 +-------------------+        +--------------------+
```

### A. DC Agent Mode
* **Come funziona:** Un componente software (`DCAgent_Setup`) viene installato direttamente a bordo di ciascun Domain Controller della foresta.
* **Caratteristiche:**
  * Intercetta i pacchetti di autenticazione Kerberos/NTLM in tempo reale non appena l'utente effettua il logon.
  * Invia immediatamente l'evento al Collector Agent centrale (porta UDP 8002).
  * Massima reattività e scalabilità (nessun ritardo di propagazione); richiede però l'installazione di software sui Domain Controller.

### B. Polling Mode
Non installa alcun software sui Domain Controller; interroga periodicamente i registri eventi di sicurezza (*Security Event Log*, es. Event ID 4624).
1. **Collector Agent-Based Polling:**
   * Il software `FSSO_Setup` è installato su un server membro Windows dedicato all'interno del dominio.
   * Il Collector interroga da remoto i Domain Controller a intervalli regolari (WMI o WinAPI) e invia le associazioni al FortiGate.
2. **Agentless (Polling diretto dal FortiGate):**
   * Nessun agente o server intermedio (*zero-footprint*).
   * È il FortiGate stesso a collegarsi direttamente via WMI/SMB ai Domain Controller con credenziali amministrative.
   * *Pro:* Nessuna installazione esterna.
   * *Contro:* Genera carico aggiuntivo sulla CPU del firewall; consigliato per ambienti di piccole dimensioni.

### C. Terminal Server (TS) Agent
* **Il problema:** In ambienti desktop condivisi (Citrix XenApp o Microsoft Terminal Server / RDS), decine di utenti lavorano contemporaneamente condividendo l'indirizzo IP del server. Il normale FSSO sovrascriverebbe l'IP ad ogni nuovo login.
* **Come funziona:** L'agente `TSAgent_Setup` viene installato sul Terminal Server. Assegna dinamicamente a ciascun utente un intervallo univoco di porte TCP/UDP sorgente (*source port range*). FortiGate è così in grado di distinguere gli utenti e applicare policy differenziate pur condividendo lo stesso IP.

---

## 5. Configurazione FSSO su FortiOS (Fabric Connectors)

### 5.1 FSSO Agentless Polling Mode
* **Percorso GUI:** `Security Fabric > External Connectors`
* **Selezione:** Categoria *Endpoint/Identity* -> icona **Poll Active Directory Server** (logo Windows).
* **Parametri principali:**
  * `Server IP/Name`: Indirizzo IP o FQDN del Domain Controller da monitorare.
  * `User / Password`: Credenziali di un account con privilegi di lettura sui log di sicurezza del DC.
  * `LDAP server`: Associazione al server LDAP preconfigurato su FortiOS (necessario per risolvere i gruppi).
  * `Enable polling`: Abilitare il selettore per attivare il controllo periodico dei registri.

### 5.2 FSSO Collector Agent Mode (o DC Agent Mode)
* **Percorso GUI:** `Security Fabric > External Connectors`
* **Selezione:** Icona rossa **FSSO Agent on Windows AD**.
* **Parametri principali:**
  * `Name`: Nome identificativo del connettore.
  * `Primary FSSO agent`: Indirizzo IP/nome della macchina Windows su cui gira il servizio Collector Agent e relativa password di autenticazione.
  * `User group source`:
    * **Collector Agent:** I gruppi e le appartenenze vengono risolti e filtrati dal Collector remoto prima dell'invio al firewall.
    * **Local:** FortiGate usa la propria connessione LDAP locale (*Proactively retrieve from LDAP server*) per risolvere autonomamente i gruppi, riducendo il carico sul Collector.

---

## 6. Procedura di Installazione e Setup Dettagliato

### Step 1: Download dei Componenti
* **Portale:** Accesso a [https://support.fortinet.com](https://support.fortinet.com).
* **Percorso:** `Support > Firmware Download > FortiGate > Download > Seleziona Versione (es. v8.00 > 8.0 > 8.0.0)`.
* **Pacchetti disponibili:**
  * `DCAgent_Setup`: Da installare sui singoli Domain Controller.
  * `FSSO_Setup`: Collector Agent per macchine/server Windows.
  * `FSSO_Setup_edirectory`: Collector per Novell eDirectory.
  * `TSAgent_Setup`: Terminal Server Agent per Citrix e Microsoft RDS.

### Step 2: Installazione del Collector Agent
* Eseguire il setup obbligatoriamente come **Administrator**.
* Specificare un account di servizio Windows dedicato nel formato `DomainName\UserName` con permessi amministrativi di consultazione della directory.
* **Install Options:**
  * *Monitor User logon events and send the information to FortiGate*.
  * *Serve NTLM authentication requests coming from FortiGate*.
  * *Access method of Windows Directory*: scelta tra **Standard** (notazione `domain\user`) e **Advanced** (notazione LDAP completa con supporto OU).
* Possibilità di avviare il *DC Agent wizard* al termine della procedura.

### Step 3: Wizard di Installazione del DC Agent
* `IP and port for collector agent`: Inserire l'IP del Collector Agent e la porta di ascolto (default `UDP 8002`).
* `Domains to monitor`: Selezione dei domini Active Directory della foresta da includere.
* `Remove users`: Esclusione preventiva di account specifici (es. utenze di servizio, batch o test).
* `Select domain controllers`: Elenco dei Domain Controller da monitorare.
* `Working Mode`:
  * **DC Agent Mode:** Installa l'agente software sui Domain Controller.
  * **Polling Mode:** Interroga da remoto i registri eventi via API/NetAPI o Security Logs senza installare software sui DC.

### Step 4: Console di Gestione del Collector Agent
* `Monitor user logon events`: Abilita il monitoraggio continuo.
* `Support NTLM authentication`: Gestisce richieste di autenticazione fallback NTLM.
* **Porte di Ascolto Fondamentali:**
  * **Verso FortiGate:** TCP `8000` (in chiaro) | TCP `8001` (con cifratura SSL).
  * **Verso DC Agent:** UDP `8002` (in chiaro) | UDP `8003` (con SSL / Pre-shared key).
* `Enable SSL`: Cifra le comunicazioni di rete tra Collector e FortiGate.
* `Authentication`: Opzione *Require authenticated connection from FortiGate* con password condivisa (deve corrispondere a quella configurata nel connettore su FortiOS).

### Step 5: Filtraggio dei Gruppi (Group Filter)
* **Gestione centralizzata:** Definiti sul Collector Agent per limitare quali informazioni di gruppo inviare a ciascun firewall.
* **Legame con il Serial Number:** I filtri possono essere associati al seriale univoco del FortiGate (*tied to FortiGate serial number*), permettendo a un unico Collector di inviare elenchi di gruppi differenziati a firewall diversi.
* **Flessibilità:** I filtri supportano gruppi specifici, intere Unità Organizzative (OU), singoli utenti o combinazioni.

### Step 6: Lista Utenti Ignorati (Ignored User List)
Evita che account di servizio automatici o task schedulati sovrascrivano continuamente l'IP delle postazioni utente:
* **Manual entry:** Inserimento manuale del nome account (supporta caratteri jolly `*` e `?`).
* **Add Users:** Selezione grafica degli utenti navigando la directory.
* **Add by OU:** Esclusione in blocco di un'intera Unità Organizzativa (es. l'OU degli account di servizio o dei computer).

### Step 7: Configurazione dell'Oggetto Gruppo FSSO su FortiOS
Percorso: `User & Authentication > User Groups > Create New > Fortinet Single Sign-On (FSSO)`

* **In Standard Access Mode:**
  * Mostra l'elenco piatto dei gruppi inviati dal Collector Agent (*Collector Agent Group Filters*).
  * L'amministratore seleziona i gruppi desiderati tramite casella di spunta.
* **In Advanced Access Mode:**
  * Mostra l'intero **albero gerarchico LDAP** del dominio (es. `dc=trainingad,dc=training,dc=lab`).
  * Permette di navigare visivamente tra cartelle e OU, applicare filtri LDAP personalizzati (*Custom LDAP filter*) e selezionare in modo granulare singoli utenti, gruppi o intere OU.