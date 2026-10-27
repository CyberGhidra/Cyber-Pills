## Firewall authentication

- Identificazione di utenti e gruppi di utenti:
    - include l'autenticazione formale di singoli utenti o gruppi.
    - Maggiore affidabilità:
      autenticare gli utenti tramite credenziali valide offre un livello di sicurezza superiore rispetto al semplice riconoscimento basato sull'indirizzo IP o sulla tipologia di dispositivo (entrambi facilmente falsificabili o condivisi).
    - Flusso operativo:
      L'utente tenta di accedere a una risorsa di rete e il FortiGate richiede l'autenticazione presentando una schermata di login (Captive Portal / Authentication Required).
      L'utente inserisce le credenziali valide (Username e Password).
      Una volta accertata l'identità dell'utente o del dispositivo, FortiGate applica le relative Firewall Policy e i Security Profile per autorizzare o negare l'accesso alla specifica risorsa richiesta.
      
- Metodi di Autenticazione Firewall su FortiGateLa :
    - A. Local Password Authentication (Autenticazione Locale)
      Nome utente e password sono memorizzati e gestiti direttamente all'interno del database locale del FortiGate.
      Indicata per contesti con un numero ridotto di account o per utenze di emergenza.
    - B. Server-based Password Authentication (Autenticazione Remota)
      Chiamata anche autenticazione remota, delega la verifica delle credenziali a un server di directory o di autenticazione centralizzato.
      I protocolli/server supportati includono:
      LDAP (es. Microsoft Active Directory)
      RADIUS
      TACACS+
      POP3
    - C. Two-Factor Authentication (2FA / Autenticazione a Due Fattori)
      Viene attivata come livello aggiuntivo sopra uno dei metodi preesistenti (locale o basato su server).
      Richiede la combinazione di due fattori di sicurezza:
      Something you know (qualcosa che si conosce): password o PIN.
      Something you have (qualcosa che si possiede): token hardware/software (es. FortiToken Mobile) o certificato digitale.  

- Per vedere le utenze create User authentication > user definition
- Il Processo di Server-Based Password Authentication (da A a Z)
  
  [ FASE PREPARATORIA: Configurazione ]
  Server Remoto preconfigurato (LDAP/RADIUS)
           │
           ├───► Metodo 1: Creazione singolo utente locale associato a Server
           │         └─► Inserito in un User Group
           │
           └───► Metodo 2: Associazione dell'intero Server a un User Group
                     └─► Tutti gli utenti del server sono inclusi
                                   │
                                   ▼
                       [ Inserito in Firewall Policy ][cite: 1, 11]

-------------------------------------------------------------------------

[ FASE OPERATIVA: Flusso di Connessione Utente ]
  1. Client invia traffico ────► FortiGate presenta Captive Portal
  2. Utente inserisce credenziali (User/PW) ────► FortiGate riceve
  3. FortiGate inoltra credenziali ────► Remote Server (LDAP/RADIUS)
  4. Server verifica e risponde "OK" ────► FortiGate valida identità
  5. FortiGate applica Firewall Policy + Profili di Sicurezza[cite: 1]


- Come si fa :
  a) Prerequisito fondamentale: il server di autenticazione esterno (RADIUS, LDAP, TACACS+) deve essere già preventivamente censito e configurato su FortiGate
  b) Da qui, l'amministratore può scegliere tra due strade operative:
  
  Opzione 1 - (Creazione Utente per Utente):
  Percorso: User & Authentication > User Definition.
  Si seleziona il tipo di utente remoto (Remote RADIUS User, Remote TACACS+ User o Remote LDAP User).
  È obbligatorio specificare manualmente il campo Username esatto censito sul server esterno e puntare al server preconfigurato.
  L'utente creato viene infine aggiunto a un gruppo utenti di FortiGate (User Group).
  Vantaggio: controllo granulare sui singoli profili utente definiti sul firewall.
  
  Opzione 2 (Server aggiunto direttamente al Gruppo):
  Percorso: User & Authentication > User Groups.
  Invece di creare i singoli account a mano sul firewall, si seleziona l'intero server remoto (o specifici gruppi remoti del server, es. un gruppo Active Directory) e lo si assegna direttamente al gruppo locale di FortiGate.
  Vantaggio: tutti gli utenti presenti su quel server diventano automaticamente membri del gruppo. Elimina l'onere amministrativo di creare o aggiornare le utenze su FortiOS quando nuovi utenti vengono assunti o censiti sulla directory aziendale.
  
  Collegamento alle Policy: il gruppo utenti (ottenuto tramite Opzione 1 o Opzione 2) viene inserito nel campo Source (sezione User/Group) della Firewall Policy per abilitare il controllo d'accesso condizionato.


- LDAP Overview (Panoramica e Funzionamento)
  LDAP (Lightweight Directory Access Protocol) è un protocollo di livello applicativo impiegato per interrogare,
  autenticare e manutenere servizi di directory distribuiti (come Microsoft Active Directory, OpenLDAP, Novell eDirectory)
  Componenti della comunicazione:
  - LDAP Client (FortiGate): il firewall agisce come client LDAP che inoltra le richieste di verifica credenziali ricevute
    dagli utenti della rete.
  - DSA (Directory System Agent): è il server LDAP/Active Directory remoto che detiene il database utenti.
  Porte di rete e crittografia:
  - Porta TCP 389: utilizzata per il traffico LDAP in chiaro oppure protetto tramite estensione crittografica STARTTLS
    (aggiornamento dinamico della sessione a TLS).
  - Porta TCP 636: utilizzata per LDAPS (LDAP over SSL/TLS), in cui l'intero canale TCP è cifrato fin dal primo handshake.

- Configuring an LDAP Server on FortiGate (Configurazione GUI)
  - User & Authentication > LDAP Servers
  - I parametri principali evidenziati sono:
    Parametri di Rete di Base:
    Name: etichetta interna dell'oggetto (es. External_Server).
    Server IP/Name: l'indirizzo IP o l'FQDN del controller di dominio / server LDAP (es. 10.0.13.130).  
    Server Port: porta di servizio (nell'esempio: 389).
  - Attributi di Identificazione Utente:
    Common Name Identifier: l'attributo schema che identifica il nome login dell'utente (Directory tree attribute that
    identifies users).
    Nell'esempio è impostato su uid (standard tipico per Linux/OpenLDAP).
    Nota operativa: se il server è Microsoft Active Directory, questo valore viene in genere impostato su sAMAccountName o
    userPrincipalName
  - Credenziali di Binding Amministrativo (Bind Type):
    Bind Type: selezionato su Regular (consente al firewall di presentarsi al server con un account dedicato per poter
    sfogliare la directory).
    Username / Password: le credenziali di un account con permessi di lettura LDAP (Credentials for an LDAP administrator),
    compilate indicando il DN completo (es. uid=adadmin,cn=Users,dc=training,dc=lab).
  - Protocollo di Sicurezza e Test di Connettività:
    Protocol: selettore tra STARTTLS e LDAPS per abilitare la cifratura del traffico ed evitare il passaggio delle
    credenziali in chiaro.
    Test Connectivity: pulsante per validare che l'IP e la porta remota rispondano (esito: Successful).
    Test User Credentials: pulsante per eseguire una prova pratica di autenticazione digitando username e password di un
    utente reale per verificare la conformità di tutto l'albero.

  - RADIUS Overview (Panoramica e Pacchetti)
    Questa slide illustra come funziona il protocollo RADIUS (Remote Authentication Dial-In User Service) quando FortiGate
    agisce come client di autenticazione verso un server centrale.
    Protocollo AAA Standard: RADIUS è un protocollo standard di rete che fornisce servizi di AAA:
    Authentication (Autenticazione): verifica l'identità dell'utente (chi sei).
    Authorization (Autorizzazione): stabilisce cosa l'utente ha il permesso di fare (a quali risorse/gruppi appartiene).        Accounting (Tracciamento/Contabilità): registra cosa fa l'utente, quando accede e quando si disconnette.
    
    Tipi di messaggi RADIUS scambiati:
    Access-Request: FortiGate raccoglie le credenziali dell'utente e le invia al server RADIUS per la validazione.              Risposta del Server: il server RADIUS valuta la richiesta e può rispondere con uno di questi tre pacchetti:
        Access-Accept: credenziali corrette e autorizzazione concessa; FortiGate fa passare il traffico dell'utente.                Access-Reject: credenziali errate o utente non autorizzato; FortiGate nega l'accesso.
        Access-Challenge: il server richiede un passaggio aggiuntivo prima di decidere (ad esempio un codice OTP per                l'autenticazione a due fattori/2FA o un cambio password obbligatorio),

    Slide 2: Configuring a RADIUS Server on FortiGate
         parametri principali evidenziati sono:
         Parametri Generali:
         Name: etichetta interna per identificare l'oggetto sul firewall (es. RADIUS_Server).  
         Authentication method: impostato su Default (oppure Specify per forzare algoritmi specifici come PAP, CHAP, MS
         CHAPv2).
         NAS IP: IP opzionale con cui FortiGate si presenta al server come Network Access Server.
         Include in every user group: se abilitato, include implicitamente questo server in tutti i gruppi utenti del                firewall.
         Parametri del Primary Server:IP/Name: 
         indirizzo IP o FQDN del server RADIUS (nell'esempio: 10.0.13.130).   
         Secret (Shared Secret): la chiave segreta condivisa tra FortiGate e il server RADIUS (must match). 
         Questa password deve essere rigorosamente identica su entrambi i lati; serve a cifrare la password dell'utente nel          pacchetto e a autenticare il firewall presso il server.

  Slide 3: Testing LDAP and RADIUS Query on the CLI (Diagnostica da CLI)
      - A. Test per Server LDAP
        diagnose test authserver ldap <server_name> <username> <password>
      - esempio 
      # diagnose test authserver ldap External_Server aduser1 Training!
        authenticate 'aduser1' against 'External_Server' succeeded!
        Group membership(s) - CN=AD-users,OU=Training,DC=trainingAD,DC=training,DC=lab
      - B. Test per Server RADIUS
        diagnose test authserver radius <server_name> <scheme> <user> <password>
      - esempio
        # diagnose test authserver radius FortiAuth-RADIUS pap student fortinet
          authenticate 'student' against 'pap' succeeded, server=primary
          assigned_rad_session_id=810153440 session timeout=0 secs!
          Group membership(s) - remote-RADIUS-admins
    
