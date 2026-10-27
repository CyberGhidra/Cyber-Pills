# FortiGate Firewall Authentication

L'autenticazione a livello di firewall consente di identificare in modo univoco singoli utenti o gruppi prima di autorizzare l'accesso alle risorse di rete. Rispetto al controllo basato esclusivamente su indirizzo IP o MAC address (facilmente soggetti a spoofing, NAT o rotazione DHCP), l'autenticazione dell'identità garantisce un livello di sicurezza superiore e permette l'applicazione di policy granulari orientate all'utente (**Identity-Based Policies**).

---

## 1. Architettura e Metodi di Autenticazione

### Flusso Operativo Generale
1. **Richiesta di connessione:** Il client genera traffico verso una destinazione protetta.
2. **Intercettazione (Captive Portal):** FortiGate intercetta la sessione HTTP/HTTPS e presenta un portale web di autenticazione (*Authentication Required*).
3. **Verifica delle credenziali:** L'utente inserisce username e password, che vengono validate localmente o inoltrate a un server esterno.
4. **Enforcement della Policy:** Accertata l'identità, FortiGate associa l'indirizzo IP della sessione all'utente autenticato e applica le specifiche **Firewall Policy** e i relativi **Security Profiles** (Antivirus, Web Filter, IPS).

### Metodi di Autenticazione Supportati
* **Autenticazione Locale (Local Password Authentication):**
  * Credenziali salvate direttamente nel database interno di FortiOS.
  * *Uso consigliato:* Reti isolate, contesti con pochissimi utenti o account amministrativi di emergenza (*break-glass*).
* **Autenticazione Remota (Server-Based Authentication):**
  * Delega la validazione a un server di directory aziendale.
  * *Protocolli supportati:* LDAP / LDAPS (Microsoft Active Directory, OpenLDAP), RADIUS, TACACS+, POP3.
  * *Uso consigliato:* Ambienti aziendali scalabili con gestione centralizzata delle identità.
* **Autenticazione a Due Fattori (2FA / MFA):**
  * Si sovrappone all'autenticazione locale o remota combinando:
    * **Qualcosa che conosci:** Password o PIN.
    * **Qualcosa che possiedi:** Token hardware/software (es. FortiToken Mobile) o certificato digitale.

---

## 2. Server-Based Authentication: Configurazione e Flussi

### Flusso Logico (End-to-End)

```
[ FASE PREPARATORIA: Configurazione ]
Server Remoto Censito (LDAP / RADIUS)
          │
          ├───► Metodo 1: Creazione singolo utente remoto su FortiGate
          │              └─► Inserimento in un User Group locale
          │
          └───► Metodo 2: Mappatura dell'intero Server o Gruppo AD in uno User Group
                         └─► Appartenenza dinamica automatica
                                     │
                                     ▼
                     [ Applicazione nella Firewall Policy ]
```

```
[ FASE OPERATIVA: Flusso di Connessione ]
1. Client invia traffico           ────► FortiGate mostra il Captive Portal
2. Utente inserisce credenziali    ────► FortiGate raccoglie User/Password
3. FortiGate interroga il server   ────► Query verso Remote Server (LDAP/RADIUS)
4. Server valida e risponde        ────► Risposta di autorizzazione (OK / Access-Accept)
5. Sessione aperta                 ────► FortiGate applica Policy e Security Profiles
```

### Strategie di Mappatura Utenti su FortiOS

| Parametro | Opzione 1: Utente Singolo (*Remote User*) | Opzione 2: Server / Gruppo Mappato (*Group Match*) |
| :--- | :--- | :--- |
| **Percorso GUI** | `User & Authentication > User Definition` | `User & Authentication > User Groups` |
| **Configurazione** | Si crea un utente remoto specificando l'esatto username presente sul server esterno. | Si inserisce direttamente il server remoto o uno specifico gruppo del directory server nello User Group locale. |
| **Manutenzione** | Alta: ogni nuovo assunto richiede la creazione dell'oggetto su FortiGate. | Bassa: automatica in base alle modifiche apportate sul server centrale (es. Active Directory). |
| **Vantaggio** | Massimo controllo granulare sui singoli utenti. | Elevata scalabilità ed eliminazione del sovraccarico amministrativo. |

> **Associazione alla Policy:** In entrambi i casi, lo **User Group** risultante deve essere inserito nel campo **Source** (sezione *User / Group*) della Firewall Policy.

---

## 3. Protocollo LDAP / LDAPS

LDAP (*Lightweight Directory Access Protocol*) è un protocollo applicativo per interrogare e gestire directory distribuite.

* **Ruoli:**
  * **LDAP Client:** FortiGate (inoltra le richieste di bind/search).
  * **DSA (Directory System Agent):** Il server remoto (es. Domain Controller Active Directory).
* **Porte e Sicurezza:**
  * **TCP 389:** LDAP standard in chiaro; supporta l'upgrade a sessione cifrata tramite **STARTTLS**.
  * **TCP 636:** **LDAPS** (LDAP over SSL/TLS), crittografato nativamente fin dall'handshake iniziale.

### Parametri di Configurazione (`User & Authentication > LDAP Servers`)

* **Name:** Identificativo univoco dell'oggetto su FortiOS (es. `AD_Corp_DC01`).
* **Server IP/Name & Port:** Indirizzo IP/FQDN del server e porta (`389` per STARTTLS o `636` per LDAPS).
* **Common Name Identifier:** Attributo di schema che rappresenta il nome di login:
  * OpenLDAP / Linux: solitamente `uid` o `cn`.
  * Active Directory: solitamente `sAMAccountName` (formato *username*) o `userPrincipalName` (formato *user@domain.local*).
* **Distinguished Name (Base DN):** Il ramo dell'albero da cui iniziare la ricerca (es. `dc=azienda,dc=local`).
* **Bind Type (Regular):** Consente al firewall di autenticarsi con un account di servizio per navigare la struttura directory:
  * **Username:** DN completo dell'account amministrativo/servizio (es. `cn=svc_fortigate,ou=ServiceAccounts,dc=azienda,dc=local`).
  * **Password:** Password associata all'account di servizio.
* **Test Tools:**
  * *Test Connectivity:* Verifica raggiungibilità IP e apertura della porta.
  * *Test User Credentials:* Verifica in tempo reale il corretto binding e la validità delle credenziali di un utente reale.

---

## 4. Protocollo RADIUS

RADIUS (*Remote Authentication Dial-In User Service*) opera su architettura client/server fornendo funzionalità complete di **AAA**:
* **Authentication:** Verifica chi è l'utente.
* **Authorization:** Determina a quali risorse o gruppi ha accesso.
* **Accounting:** Registra statistiche di utilizzo, inizio/fine sessione e volume di traffico.

### Pacchetti di Comunicazione RADIUS
1. **Access-Request:** FortiGate pacchettizza le credenziali utente e le inoltra al server RADIUS.
2. **Access-Accept:** Credenziali corrette e privilegi validati; FortiGate sblocca il traffico.
3. **Access-Reject:** Credenziali errate o utente non autorizzato; accesso negato.
4. **Access-Challenge:** Il server richiede passaggi aggiuntivi prima di autorizzare la sessione (es. richiesta OTP per 2FA o cambio password scaduta).

### Parametri di Configurazione (`User & Authentication > RADIUS Servers`)
* **Primary Server IP/Name:** Indirizzo del server RADIUS (es. FortiAuthenticator, Cisco ISE, Microsoft NPS).
* **Secret (Shared Secret):** Stringa segreta condivisa tra FortiGate e RADIUS. È usata per calcolare l'hash di protezione delle credenziali e autenticare il NAS (*Network Access Server*). Deve coincidere rigorosamente su entrambi i nodi.
* **Authentication Method:** Metodo di autenticazione supportato (`Default`, oppure specificato manualmente tra `PAP`, `CHAP`, `MS-CHAPv2`).
* **NAS IP:** Indirizzo IP sorgente con cui FortiGate si presenta al server RADIUS (utile se il server applica filtri sui client autorizzati).

---

## 5. Diagnostica e Troubleshooting da CLI

La CLI di FortiOS offre comandi dedicati per simulare le query di autenticazione senza dover attendere il traffico reale dei client.

### Test Autenticazione LDAP
```
diagnose test authserver ldap <ldap_server_name> <username> <password>
```

*Esempio ed esito atteso:*
```
# diagnose test authserver ldap External_Server aduser1 Training!
authenticate 'aduser1' against 'External_Server' succeeded!
Group membership(s) - CN=AD-users,OU=Training,DC=trainingAD,DC=training,DC=lab
```

### Test Autenticazione RADIUS
```
diagnose test authserver radius <radius_server_name> <auth_scheme> <username> <password>
```
*Schemi supportati:* `pap`, `chap`, `mschap`, `mschapv2`.

*Esempio ed esito atteso:*
```
# diagnose test authserver radius FortiAuth-RADIUS pap student fortinet
authenticate 'student' against 'pap' succeeded, server=primary
assigned_rad_session_id=810153440 session timeout=0 secs!
Group membership(s) - remote-RADIUS-admins
```

---

## 6. FortiToken e Autenticazione a Due Fattori (2FA)

FortiOS include nativamente il supporto per l'autenticazione multifattore tramite token hardware fisici o token software (**FortiToken Mobile**).
* Ogni unità FortiGate include **2 licenze FortiToken Mobile gratuite** utilizzabili fin da subito.
* Licenze aggiuntive possono essere registrate tramite codice di attivazione in `User & Authentication > FortiTokens`.

### Modalità di Risposta: Passiva vs Attiva

```
[ MODALITÀ PASSIVA (TOTP Manuale) ]
Utente apre l'app ──► Legge il codice a 6 cifre ──► Digita manualmente l'OTP a schermo

[ MODALITÀ ATTIVA (Push Notification) ]
Utente invia User/PW ──► Notifica Push su smartphone ──► Tap su "Approve" o "Deny"
```

* **Modalità Passiva (TOTP):** Genera codici numerici *Time-based One-Time Password* a 6 cifre con validità di 30-60 secondi. Richiede la digitazione manuale dell'OTP nel form del portale o su FortiClient.
* **Modalità Attiva (Push):** Richiede connettività Internet per il dispositivo mobile. Mostra una notifica interattiva: con un tocco su **Approve** l'autenticazione viene completata istantaneamente senza digitare codici.

### Assegnazione via GUI
1. **Per Amministratori di Sistema:**
   * Percorso: `System > Administrators`.
   * Modificare l'account, abilitare **Two-factor Authentication** e selezionare il FortiToken da associare.
2. **Per Utenti di Rete (VPN / Captive Portal):**
   * Percorso: `User & Authentication > User Definition`.
   * Selezionare l'utente locale o remoto, attivare **Two-factor Authentication** e scegliere il token.
3. **Distribuzione:**
   * Configurare i campi *Email* o *SMS* nel profilo utente.
   * Cliccare su **Send Activation Code**: FortiOS invia una notifica con il codice di provisioning o il QR code da inquadrare con l'applicazione mobile.

---

## 7. Configurazione da CLI (Prontuario Rapido)

### Registrazione Licenze Token su FortiGate
```
config user fortitoken
    edit "<CODICE_ATTIVAZIONE_O_SERIALE>"
    next
end
```

### Associazione 2FA a un Amministratore (`admin_sec`)
```
config sys admin
    edit "admin_sec"
        set two-factor enable
        set fortitoken "<NUMERO_SERIALE_TOKEN>"
        set email-to "admin@azienda.local"
    next
end
```

### Creazione Utente Locale con 2FA FortiToken
```
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
