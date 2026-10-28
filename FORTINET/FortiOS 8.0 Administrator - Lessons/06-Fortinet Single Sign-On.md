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
