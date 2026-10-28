## Fortinet Single Sign-On

- Definizione di SSO (Single Sign-On)
  Cos'è: è un meccanismo di autenticazione che permette a un utente già identificato di accedere a molteplici applicazioni o risorse di rete senza doversi autenticare nuovamente a ogni richiesta.
  Vantaggio: elimina la necessità di reinserire continuamente username e password.

- Autenticazione Trasparente e Funzionamento di FSSO
  A differenza dell'autenticazione attiva tramite Captive Portal (dove il firewall blocca la sessione e mostra una pagina di login), FSSO opera in maniera completamente trasparente o passiva:
  Nessun prompt per l'utente: 
  gli utenti già autenticati (ad esempio al momento del login su Windows all'avvio della propria postazione di lavoro) accedono alle applicazioni e alla rete senza visualizzare alcuna richiesta di credenziali.
  Cosa identifica il software FSSO: intercetta e associa tra loro tre informazioni chiave dell'utente:
  - Username: nome dell'account utente.
  - IP address: indirizzo IP assegnato alla postazione/dispositivo.
  - Group membership: gruppi di appartenenza all'interno del dominio/directory.

- Ruolo del FortiGate e Mappatura dei Gruppi
  Controllo d'accesso basato sui gruppi: FortiGate concede o nega l'accesso alle risorse in base all'appartenenza a specifici gruppi FSSO configurati a bordo del firewall.
  Flessibilità di associazione: i gruppi FSSO sul FortiGate possono essere mappati direttamente su:
  - Singoli utenti (individual users)
  - Gruppi di utenti (user groups)
  - Unità Organizzative (Organizational Units - OUs)
  - Una combinazione di questi elementi

- Integrazione con Directory Services
  FSSO lavora tipicamente integrandosi con servizi di directory aziendali centralizzati, tra cui:
  - Microsoft Active Directory (AD) (lo scenario più diffuso, dove FSSO legge i log di logon dei Domain Controller).
  - Novell eDirectory.
 
- Tabella comparativa: FSSO vs. ZTNA
  
| Parametro | FSSO (Fortinet Single Sign-On) | ZTNA (Zero Trust Network Access) |
| :--- | :--- | :--- |
| **Identity source** | Active Directory / Eventi di login LDAP | FortiClient EMS o FortiClient EMS Cloud |
| **How it works** | Mappa Indirizzo IP + Utente/Gruppo | Mutual TLS (mTLS) esplicito + Tag di security posture |
| **Agent** | DC Agent, Collector Agent o SSOMA | FortiClient (obbligatorio su endpoint) |
| **FortiGate receives** | Username AD + Appartenenza ai gruppi | Certificato del dispositivo, identità utente, tag |
| **Policy enforcement** | Corrispondenza gruppi utente nelle Firewall Policy | ZTNA Firewall Policy o ZTNA Proxy Policy |
| **Device posture** | Non valutato (*Not evaluated*) | Controllato continuamente (antivirus, patch, OS, ecc.) |
| **Location scope** | Principalmente utenti on-net (rete locale) | Sia on-net che off-net (remoto e locale) |

- Quando usare FSSO
  FSSO è la scelta migliore quando l'obiettivo principale è la comodità d'uso all'interno del perimetro di rete aziendale:
  Nessun impatto sull'endpoint:
    - Non è necessario installare né gestire software o agenti sui computer o dispositivi personali degli utenti (no agent on every device).
  Autenticazione trasparente:
    - L'utente effettua il login al proprio PC Windows (o su directory LDAP) all'inizio della giornata lavorativa e accede alle risorse di rete senza visualizzare finestre o portali di login (Captive Portal).
  Flessibilità BYOD:
    - È indicato in ambienti in cui sono presenti postazioni non gestite, dispositivi personali (BYOD) o sistemi operativi non standard che non possono ospitare un agente aziendale.
  Firewall Policy on-net:
    - Si applica quando il firewall deve semplicemente associare l'indirizzo IP locale dell'utente ai gruppi di sicurezza di Active Directory per le normali regole di transito.
 
- Quando usare ZTNA
ZTNA è indispensabile quando l'azienda adotta un'architettura Zero Trust ("Never trust, always verify"):
Controllo continuo dello stato di salute (Device Posture): 
  - L'accesso è subordinato non solo a chi è l'utente, ma anche a come sta il dispositivo (antivirus attivo e aggiornato, patch di sicurezza applicate, crittografia del disco abilitata). 
    Se il computer contrae un malware o non rispetta le conformità, l'accesso viene revocato istantaneamente.
Sostituzione delle dial-up VPN (Smart Working sicuro): 
  - Anziché connettere l'utente remoto con una VPN classica (che apre l'accesso all'intera subnet aziendale), ZTNA crea un tunnel mTLS criptato applicazione per applicazione (per-application access), impedendo il movimento laterale di possibili attaccanti.
Sicurezza costante ovunque:
  - Garantisce la medesima postura e gli stessi controlli sia che il dipendente si trovi alla scrivania in ufficio (on-net), sia che operi da remoto da casa o in mobilità (off-net).
 

- Che cos'è FortiIdentity Cloud?
  Definizione: è un servizio IAM (Identity and Access Management) completo erogato interamente via Cloud (cloud-delivered).
  Scopo principale: fornisce meccanismi avanzati di MFA (Multi-Factor Authentication) e SSO (Single Sign-On) sia per l'ecosistema di prodotti Fortinet che per applicazioni e servizi di terze parti.
  Zero-Footprint Deployment: la sua implementazione è estremamente snella e non richiede l'acquisto di hardware aggiuntivo, l'installazione di software server dedicato in sede o modifiche complesse alle liste di controllo degli accessi di rete (no additional hardware,   software, or ACL changes).

- Funzionalità e Architettura (I Tre Pilastri)
  A. Metodi MFA (MFA Methods)
     Offre molteplici opzioni per soddisfare standard di sicurezza e preferenze degli utenti:
     - FortiToken Mobile OTP: generazione di codici temporizzati (TOTP) o notifiche push tramite l'applicazione mobile per smartphone.
     - Hardware Token (FTK): supporto per token fisici portatili Fortinet (es. serie FortiToken-200).
     - Email / SMS OTP: invio di codici One-Time Password via posta elettronica o messaggistica SMS.
     - FIDO2 Passkeys: autenticazione moderna, passwordless e resistente agli attacchi di phishing conforme allo standard FIDO2 (come chiavi di sicurezza fisiche USB o biometria di sistema).

  B. Funzionalità IAM (IAM Features)
     Local Identity Provider (IdP): può agire direttamente come autorità centrale di autenticazione e provider di identità per gli account aziendali.
     IdP Proxy capability: può fare da intermediario (proxy) tra le applicazioni e altri Identity Provider esterni o federati preesistenti (ad esempio Microsoft Entra ID / Azure AD, Okta o Google Workspace).
     Adaptive Authentication: applica regole di autenticazione dinamiche e contestuali basate su fattori di rischio (posizione geografica, dispositivo utilizzato, reputazione IP o comportamento anomalo).
     Admin and End-User Portals: include portali web dedicati per consentire agli amministratori di configurare le policy e agli utenti finali di accedere alle proprie applicazioni abilitate via SSO e gestire i propri token di sicurezza.

  C. Integrazioni (Integrations)
     Prodotti Fortinet nativi: integrazione immediata con FortiGate, FortiAuthenticator, FortiPAM (Privileged Access Management), FortiSandbox e FortiADC.   
     Applicazioni basate su standard aperti: supporto per qualsiasi piattaforma SaaS o on-premise che utilizzi i protocolli standard SAML, OIDC (OpenID Connect) e OAuth 2.0.
     Importazione e sincronizzazione utenti: supporto per il caricamento massivo tramite file CSV, chiamate API e provisioning automatico in tempo reale tramite standard SCIM (System for Cross-domain Identity Management).

- Modello di Licenza (License Types)
  La piattaforma adotta un modello commerciale flessibile basato su cloud:   
  Subscription-based: servizio fruibile in modalità abbonamento.   
  Per-user: il canone viene calcolato in base al numero di identità/utenti attivi censiti sulla piattaforma.   
  Time-based: licenze con durata temporale definita (es. contratti annuali o pluriennali).   
  Stackable with co-term renewals: le licenze sono cumulabili (stackable) e supportano l'allineamento delle scadenze (co-terming), consentendo di aggiungere nuovi pacchetti utente in qualsiasi momento sincronizzando un'unica data di rinnovo annuale

- Piattaforme Supportate da FortiOS
  FortiOS supporta l'integrazione SSO con diversi sistemi e piattaforme aziendali:   
  - Microsoft Windows Active Directory (AD) (lo scenario più diffuso)
  - Novell eDirectory
  - Citrix
  - VMware Horizon
  - Microsoft Exchange

- Modalità di Deployment per Microsoft Active Directory (AD)
  Per intercettare le sessioni di login degli utenti su Windows AD e mappare l'associazione IP <-> Utente <-> Gruppo, FortiOS offre tre architetture principali:

  A. Domain Controller (DC) Agent Mode
  Come funziona: 
  un piccolo componente software (l'agente DC) viene installato direttamente a bordo di ogni Domain Controller della foresta AD.
  Caratteristiche:
  Il DC Agent intercetta i pacchetti di autenticazione Kerberos/NTLM in tempo reale non appena l'utente effettua il logon alla postazione.
  Invia immediatamente l'evento a un Collector Agent centrale, che a sua volta aggiorna la tabella FSSO sul FortiGate. 
  È la soluzione più reattiva ed evita ritardi di propagazione, ma richiede l'installazione di software di terze parti sui Domain Controller aziendali. 

  B. Polling Mode
  In questa modalità non si installa alcun software sui Domain Controller. 
  Fortinet interroga periodicamente i registri eventi di sicurezza (Security Event Log) di Windows per individuare i codici di evento legati ai logon con successo (es. Event ID 4624). 
  Si divide in due varianti:   
 -  Collector agent-based (Collector Agent Polling):
    Il software Fortinet FSSO Collector Agent viene installato su una macchina Windows dedicata all'interno del dominio (server membro o VM di servizio).   
    Il Collector interroga via rete (WMI o WinAPI) i Domain Controller a intervalli regolari, estrae i log di login e trasmette le associazioni al FortiGate.   
  - gentless (Polling diretto dal FortiGate):
    Non viene installato alcun agente esterno né software server.   
    È il FortiGate stesso a collegarsi direttamente via WMI/SMB ai Domain Controller tramite credenziali amministrative per leggere i registri di autenticazione.
    Pro: architettura a impatto zero (zero-footprint).
    Contro: genera un carico aggiuntivo sulla CPU del firewall ed è consigliabile per ambienti di piccole dimensioni.
  C. Terminal Server (TS) Agent
     Il problema da risolvere: 
     in ambienti desktop condivisi (come Citrix XenApp/Virtual Apps o Microsoft Remote Desktop Session Host / Terminal Server), decine di utenti diversi lavorano contemporaneamente condividendo lo stesso indirizzo IP del server. 
     Con il normale FSSO, il firewall vedrebbe solo l'ultimo utente connesso sovrascrivere l'IP.
     - Come funziona il TS Agent:
       Viene installato a bordo del server Citrix/Terminal Server.
       Potenzia le capacità del Collector Agent o di FortiAuthenticator assegnando dinamicamente a ciascun utente loggato un intervallo univoco di porte TCP/UDP sorgente (source port range).
       In questo modo, FortiGate distingue e applica policy differenti a utenti multipli attestati sul medesimo IP.

FSSO Configuration — Agentless Polling Mode
Come configurare l'autenticazione trasparente FSSO senza installare alcun software aggiuntivo su server o domain controller.
Percorso e Configurazione GUI
Percorso: Security Fabric > External Connectors.   
Selezione connettore: 
Dalla categoria Endpoint/Identity, si seleziona l'icona con il logo Windows Poll Active Directory Server.   
Parametri principali (Connector Settings):
Server IP/Name: Indirizzo IP o FQDN del Domain Controller da monitorare.   
User / Password: Credenziali di un account di servizio con privilegi sufficienti per leggere i log di sicurezza (Security Event Log) del controller di dominio.   
LDAP server: Associazione al server LDAP precedentemente configurato su FortiOS (necessario per risolvere i gruppi di appartenenza dell'utente).   
Enable polling: Levetta attiva per abilitare il controllo periodico dei registri di logon da parte del firewall.

FSSO Configuration — Collector Agent-Based Polling or DC Agent Mode
Mostra la configurazione quando nella rete è presente l'agente software dedicato Fortinet FSSO Collector Agent (sia che operi leggendo i log via polling dai DC, sia che riceva i dati dai DC Agent installati a bordo dei controller)
Funzionalità dell'Agente FSSOIl Collector Agent 
Monitora in modo centralizzato i dati di login degli utenti provenienti da molteplici fonti: Active Directory (AD), server di posta Exchange, sessioni Terminal Server / RDS, ambienti virtuali Citrix e server Novell eDirectory.   
Una volta consolidate le informazioni (Username + IP + Gruppi), il Collector Agent le invia al FortiGate. 
Percorso e Configurazione GUI
Percorso: Security Fabric > External Connectors.   
Selezione connettore: Si sceglie l'icona rossa FSSO Agent on Windows AD.   
Parametri principali (Connector Settings):
Name: Nome identificativo del connettore sul firewall.   
Primary FSSO agent: Indirizzo IP/nome della macchina Windows su cui è in esecuzione il servizio Collector Agent e la relativa password di autenticazione configurata sull'agente.   
User group source (Scelta fondamentale):Collector Agent: I gruppi e le appartenenze degli utenti vengono risolti e filtrati direttamente dal Collector Agent sulla macchina Windows remota prima di inviarli al firewall.   
Local: Cliccando su Local, si indica al FortiGate di utilizzare una connessione diretta a un proprio LDAP server interno per recuperare in autonomia i gruppi della directory (Proactively retrieve from LDAP server), riducendo la dipendenza dal Collector per la risoluzione dei metadati. 


SSO Agent Installation (Download del software)
Percorso di download sul portale:Accesso al sito di supporto Fortinet ([https://support.fortinet.com](https://support.fortinet.com)).   
Navigazione in Support > Firmware Download.   
Selezione del prodotto FortiGate, quindi clic su Download.   
Selezione della versione firmware di riferimento (nell'esempio: v8.00 > 8.0 > 8.0.0). 
Pacchetti di installazione disponibili (Available agents):
DC agent (DCAgent_Setup): Il componente da installare direttamente a bordo dei singoli Domain Controller.   
Collector Agent per Microsoft servers (FSSO_Setup): L'agente collettore principale da installare su un server membro Windows.   
Collector Agent per Novell (FSSO_Setup_edirectory): Pacchetto specifico per l'integrazione con Novell eDirectory.   
TS Agent (TSAgent_Setup): Il Terminal Server Agent dedicato ad ambienti Citrix o Microsoft Remote Desktop (RDS) per tracciare le porte per singolo utente.   
