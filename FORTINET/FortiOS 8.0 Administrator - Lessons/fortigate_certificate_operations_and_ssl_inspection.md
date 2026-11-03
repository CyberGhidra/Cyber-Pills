# FortiGate Certificate Operations & SSL Inspection

I certificati digitali e l'infrastruttura a chiave pubblica (PKI) costituiscono la base crittografica su cui FortiOS fonda l'autenticazione sicura dei dispositivi/utenti, la riservatezza delle comunicazioni e la visibilità applicativa (Deep Packet Inspection) necessaria a proteggere la rete dalle minacce veicolate nei flussi cifrati.

---

## 1. Fondamenti sui Certificati Digitali

### 1.1 Che cos'è un Certificato Digitale?
* **Identità Digitale Firmata:** È un file standardizzato emesso e firmato digitalmente da un'autorità terza fidata denominata **Certificate Authority (CA)**.
* **Analogia del Passaporto:** Funziona esattamente come un passaporto o una patente: un'autorità riconosciuta e garante (lo Stato / la CA) convalida che determinati dati identificativi appartengano a una persona o a un dispositivo specifico.
* **Standard:** FortiGate adotta lo standard industriale **X.509 versione 3 (X.509v3)**.

### 1.2 Scopi Primari (Primary Purposes)
1. **Authentication (Autenticazione):** Accerta l'identità del server o del client che partecipa alla sessione crittografica, proteggendo da attacchi di tipo spoofing e Man-in-the-Middle (MitM).
2. **Encryption and Decryption (Cifratura e Decifratura):** Fornisce la chiave pubblica necessaria a negoziare in modo sicuro le chiavi simmetriche temporanee di sessione (ad esempio per HTTPS, SSL-VPN o tunnel IPsec).
3. **Integrity (Integrità):** Garantisce che le informazioni scambiate non siano state alterate, contraffatte o corrotte durante il percorso tra sorgente e destinatario.

### 1.3 Anatomia dei Campi X.509v3 Rilevanti per FortiGate
* **Serial Number:** Valore esadecimale univoco generato dalla CA per catalogare, tracciare e revocare quel singolo certificato (ad esempio tramite liste CRL o protocollo OCSP).
* **Valid from / Valid to:** Finestra temporale di validità. Se l'orario di sistema del FortiGate (sincronizzato via NTP) si trova prima di *Valid from* o dopo *Valid to*, il certificato è considerato non valido.
* **Subject (DN / Common Name):** Identificativo principale dell'entità titolare del certificato (es. `CN=training.fortinet.com`).
* **Subject Alternative Name (SAN):** Estensione essenziale di X.509v3 usata dai moderni browser/sistemi operativi per elencare ulteriori domini (es. `DNS Name=training.fortinet.com`), indirizzi IP o indirizzi email associati al certificato.

---

## 2. Come FortiGate Verifica una Firma Digitale

La firma digitale è un **hash crittografato con la chiave privata dell'emittente (la Certificate Authority - CA)**. Per convalidare il certificato, FortiGate esegue una verifica crittografica asimmetrica a tre fasi.

### Schema di Verifica della Firma

```text
[ FASE 1: Calcolo Locale ]
Certificato del Server ──► Funzione Hash (stesso algoritmo) ──► Nuovo Hash (10101)

[ FASE 2: Decifratura della Firma ]
Firma Digitale (Sigillo) ──► Decifratura con Chiave Pubblica CA ──► Hash Originale (10101)

[ FASE 3: Match / Confronto ]
Nuovo Hash (10101)  ==  Hash Originale (10101)  ──►  Certificato Valido e Integro
```

1. **Calcolo Locale (Fresh Hash):** FortiGate riceve il certificato dal server remoto ed estrae il payload del file, calcolandone l'impronta hash tramite il medesimo algoritmo impiegato dal firmatario (es. SHA-256).
2. **Estrazione dell'Hash Originale:** FortiGate preleva dal proprio archivio fidato (*Remote CA Certificate Store*) la **chiave pubblica della CA** emittente e con essa decifra la firma digitale apposta sul certificato.
3. **Confronto:** Se i due valori hash risultano perfettamente identici:
   * **Autenticità garantita:** Solo la CA in possesso della propria chiave privata segreta poteva aver generato quella firma decifrabile con la sua chiave pubblica.
   * **Integrità garantita:** Nessun byte del certificato è stato manomesso o alterato in transito.

---

## 3. Perché FortiGate Utilizza i Certificati Digitali

FortiOS integra i certificati in tre ambiti operativi distinti:

### 3.1 Inspection (Ispezione del Traffico Cifrato)
* **SSL/SSH and HTTPS traffic inspection:** Apre e analizza i flussi cifrati per applicare motori UTM (AntiVirus, IPS, Web Filter, DLP).
* **Inbound Inspection:** Protegge i server web locali dell'azienda agendo da reverse proxy crittografico (utilizzando il certificato e la chiave privata del server interno).
* **Outbound Inspection:** Analizza la navigazione degli utenti interni verso Internet, generando certificati dinamici firmati da una CA interna del firewall.

### 3.2 Privacy (Riservatezza dei Dati di Sistema)
* Protegge le comunicazioni dirette tra il FortiGate e l'ecosistema esterno (query in tempo reale verso il cloud di **FortiGuard**, sincronizzazione telemetrica protetta con **FortiAnalyzer** tramite protocollo **OFTPS**).

### 3.3 Authentication (Controllo degli Accessi PKI)
* Autenticazione degli utenti per l'accesso alla rete aziendale (802.1X / PKI).
* Connessioni VPN client-to-site (SSL-VPN) o site-to-site (IPsec basate su certificati RSA/ECDSA).
* Secondo fattore di autenticazione (2FA) o autenticazione esclusiva basata su certificato per gli amministratori della GUI HTTPS e della CLI.

---

## 4. Modalità di Ispezione SSL su FortiOS

Nel traffico Internet privo di ispezione (*Encrypted Traffic With No SSL Inspection*), il tunnel crittografico end-to-end nasconde interamente il payload: virus, spyware ed exploit transitano indisturbati attraverso il firewall senza poter essere intercettati dai motori AntiVirus o DLP.

Per gestire questo traffico, FortiOS mette a disposizione due modalità di ispezione:

### 4.1 SSL Certificate Inspection
* **Funzionamento:** FortiGate **non decifra il traffico** (*does not decrypt the traffic*). La sessione TLS/SSL tra client e server rimane diretta e ininterrotta.
* **Come identifica il sito:** Estrae l'FQDN in chiaro scambiato durante l'handshake iniziale da:
  * L'estensione **Server Name Indication (SNI)** nel pacchetto `Client Hello`.
  * I campi **Subject** o **Subject Alternative Name (SAN)** del certificato inviato dal web server.
* **Cosa verifica:** Controlla che il nome richiesto dal client (SNI) coincida con il certificato inviato dal server (*checks for match*).
* **Profili compatibili:** Web Filtering basato su categorie/domini, Application Control di base.
* **Limiti:** Non supporta scansioni antivirus su file scaricati via HTTPS, non legge i percorsi URL completi (vede solo il dominio, es. `dominio.com`, non `dominio.com/path/file.exe`).
* **Vantaggi:** Nessun impatto sui dispositivi client (zero avvisi di certificato) e carico CPU ridotto.

### 4.2 Full SSL Inspection (Deep Packet Inspection)
* **Funzionamento:** FortiGate agisce come un proxy **Man-in-the-Middle autorizzato**, spezzando la connessione in **due sessioni SSL distinte e separate**:
  1. **Client-to-FortiGate:** Tra il computer dell'utente e il firewall.
  2. **FortiGate-to-server:** Tra il firewall e il web server remoto di destinazione.
* **Flusso crittografico:**
  1. Riceve i dati dal server esterno e li decifra nella memoria RAM.
  2. Scansiona il payload in chiaro tramite i profili UTM (AntiVirus, IPS, File Filter, DLP, Web Filter avanzato).
  3. Se il contenuto è pulito, FortiGate genera "al volo" un certificato per quel dominio firmato dalla propria CA locale (es. `Fortinet_CA_SSL`), ri-cifra il pacchetto e lo consegna al client.
* **Protocolli supportati:** HTTPS, SMTPS, POP3S, IMAPS, FTPS.
* **Requisito fondamentale:** Il certificato Root della CA di FortiGate deve essere preinstallato nel *Certificate Store* delle macchine client per evitare avvisi di sicurezza nel browser.

### Tabella Comparativa

| Caratteristica | SSL Certificate Inspection | Full SSL Inspection |
| :--- | :--- | :--- |
| **Decifratura del traffico** | Nessuna (payload intatto) | Completa (in memoria RAM) |
| **Numero di sessioni SSL** | 1 sessione diretta Client-Server | 2 sessioni distinte (Client-FGT e FGT-Server) |
| **Visibilità URL** | Solo Host / Dominio FQDN | URL completo, query string, file e form |
| **Supporto AntiVirus / DLP** | Non supportato | Pienamente supportato |
| **Distribuzione Certificato CA sui PC** | Non richiesta | Obbligatoria (per evitare schermate di blocco) |

---

## 5. Configurazione del Profilo SSL/SSH Inspection

Il profilo si crea e si gestisce da **Security Profiles > SSL/SSH Inspection**.

### 5.1 Opzioni Principali
* **Enable SSL Inspection of:**
  * **Multiple Clients Connecting to Multiple Servers:** Scenario *Outbound / Forward Proxy* (decifratura del traffico degli utenti interni verso Internet).
  * **Protecting SSL Server:** Scenario *Inbound / Reverse Proxy* (protezione di server web aziendali interni esposti verso Internet).
* **Inspection Method:** Scelta tra *SSL Certificate Inspection* e *Full SSL Inspection*.
* **CA Certificate:** Seleziona l'autorità utilizzata per firmare i certificati al volo (di default `Fortinet_CA_SSL`).

### 5.2 Gestione Certificati Non Validi (Invalid & Untrusted Certificates)
FortiGate può rilevare anomalie nei certificati remoti (scaduti, revocati, firma non verificabile o timeout) e permette di personalizzare le azioni:
* **Allow / Keep Untrusted & Allow:** Inoltra al client un certificato temporaneo marcato come *non fidato*, permettendo al browser dell'utente di mostrare il nativo avviso di sicurezza.
* **Block:** Abbassa la connessione e blocca istantaneamente l'accesso ai siti con certificati non validi.
* **Ignore:** FortiGate sostituisce comunque il certificato malevolo con uno valido firmato dalla propria CA fidata, mascherando di fatto il problema all'utente finale (sconsigliato per motivi di sicurezza).
* **Custom:** Permette di differenziare l'azione (*Keep Untrusted & Allow*, *Block*, *Trust & Allow*) per ciascuna singola casistica (*Expired*, *Revoked*, *Validation timed-out*, *Validation failed*).

### 5.3 Esenzioni dall'Ispezione SSL (SSL Exemptions)
Non tutto il traffico deve o può essere decifrato. Le esenzioni sono necessarie per tre motivi:
1. **Problemi applicativi:** Applicazioni con meccanismi di *Certificate Pinning* (es. Dropbox, client bancari dedicati) che rifiutano categoricamente certificati intermedi generati da proxy.
2. **Requisiti legali e di conformità:** Normative su privacy/GDPR che impediscono ai datori di lavoro di decifrare dati sensibili su portali bancari o sanitari.
3. **Ottimizzazione risorse:** Sgravare la CPU del firewall decifrando solo ciò che è necessario.

#### Metodi di Esenzione Configurabili:
* **Reputable websites:** Toggle rapido per escludere automaticamente i siti a elevata reputazione classificati come sicuri da FortiGuard.
* **Web categories:** Esclusione per intere macro-categorie (es. *Finance and Banking*, *Health and Wellness*, *Personal Privacy*).
* **Addresses:** Esenzione puntuale per singoli host, FQDN o classi di rete (es. oggetti indirizzo per Microsoft 365, Apple, Google Drive, Dropbox).

### 5.4 Applicazione alla Firewall Policy
* Il profilo configurato viene agganciato nella sezione **Security Profiles > SSL inspection** della specifica regola in **Policy & Objects > Firewall Policy**.
* Se la policy viene impostata su **`no-inspection`**, il firewall non analizza né le intestazioni né il payload del traffico SSL/SSH, rendendo inefficaci le scansioni di AntiVirus, File Filter e DLP sul traffico cifrato.
* **Decrypted traffic mirror:** Opzione per duplicare e inviare una copia del traffico decifrato verso porte fisiche collegate a sonde terze (sistemi IDS/IPS o SIEM).

---

## 6. Full SSL Inspection su Traffico Inbound (Protecting SSL Server)

Quando il firewall deve proteggere un server applicativo interno pubblicato su Internet:
1. In **Security Profiles > SSL/SSH Inspection**, si seleziona l'opzione **Protecting SSL Server**.
2. **Requisiti crittografici su FortiGate:** A bordo del firewall devono essere caricati:
   * Il **certificato pubblico reale** del server.
   * La **relativa chiave privata**.
   * L'intera catena di certificati intermedi della CA.
3. **Funzionamento:** FortiGate funge da terminatore crittografico (*SSL Offloading / Reverse Proxy*). Presenta il certificato valido ai client esterni su Internet, decifra il flusso, scansiona il payload alla ricerca di attacchi (IPS/WAF/AV) e rigenera la sessione verso il server web interno.

### Gestione Multi-Server tramite SNI
* Un unico profilo di ispezione inbound può ospitare più certificati per proteggere server web differenti che condividono lo stesso indirizzo IP pubblico esterno (*Multiple certificates defined in one profile*).
* FortiGate esamina l'estensione **SNI** inviata nel `Client Hello` esterno, la confronta con i campi CN/SAN dei certificati installati e serve il certificato corrispondente (es. `www.aaa.com`, `www.bbb.com`, `www.ccc.com`).
* **Fallback:** Se il client invia un SNI privo di corrispondenza, FortiGate utilizza automaticamente il primo certificato della lista.

---

## 7. Requisiti e Gestione dei Certificati CA

### 7.1 Requisiti della CA per la Deep Inspection
Per consentire a FortiGate di emettere e firmare certificati al volo durante la Full SSL Inspection, il certificato impiegato deve possedere obbligatoriamente due estensioni standard X.509:
* **`cA=True`** (Basic Constraints): specifica che il certificato è un'autorità di certificazione e non un'entità finale (*End Entity*).
* **`keyUsage=keyCertSign`**: autorizza esplicitamente l'apparato a firmare altri certificati digitali.

### 7.2 Tipologie di CA Utilizzabili
1. **`Fortinet_CA_SSL` (Preinstallato di Fabbrica):** Certificato autofirmato (*self-signed*). È immediatamente funzionante, ma non essendo emesso da una CA pubblica riconosciuta a livello globale, genera avvisi di sicurezza nel browser di ciascun utente (*Software is Preventing Browser From Safely Connecting*) finché non viene importato nei keystore dei client.
2. **Subordinate CA di Dominio (Consigliato in Ambienti Enterprise):** Certificato rilasciato dall'autorità PKI interna dell'azienda (es. Microsoft Active Directory Certificate Services). Essendo la CA radice aziendale già fidata su tutti i computer del dominio, non richiede alcuna installazione aggiuntiva sui PC.

---

## 8. Guida Operativa: Il FortiGate Certificate Store

L'intero ciclo di vita dei certificati si amministra da **System > Certificates**.

### 8.1 Struttura del Certificate Store

```text
┌────────────────────────────────────────────────────────────────────────┐
│                      FORTIGATE CERTIFICATE STORE                       │
├────────────────────────────────────────────────────────────────────────┤
│ 1. CRL (Certificate Revocation Lists)                                  │
│    Liste dei certificati revocati aggiornate via file o URL HTTP/LDAP  │
├────────────────────────────────────────────────────────────────────────┤
│ 2. Local CA Certificate                                                │
│    CA abilitate alla firma al volo per la Deep Inspection              │
│    (Fortinet_CA_SSL predefinito o Subordinate CA aziendale importata) │
├────────────────────────────────────────────────────────────────────────┤
│ 3. Local Certificate                                                   │
│    Certificati con chiave privata residenti su FortiGate:              │
│    • Accesso GUI HTTPS (Fortinet_GUI_Server)                           │
│    • Connessioni SSL-VPN e IPsec VPN                                   │
│    • Richieste CSR in sospeso (Pending CSR)                            │
│    • Certificati dei Web Server interni (Inbound Inspection)           │
├────────────────────────────────────────────────────────────────────────┤
│ 4. Remote CA Certificate                                               │
│    Bundle di CA radice pubbliche e private impiegate per validare      │
│    l'autenticità dei server web remoti e delle firme digitali          │
└────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Come Scaricare e Distribuire la CA (`Fortinet_CA_SSL`)
1. Navigare in **System > Certificates**.
2. Nella sezione **Local CA Certificate**, selezionare `Fortinet_CA_SSL` e cliccare sul pulsante **Download** in alto.
3. Viene salvato il file pubblico `Fortinet_CA_SSL.cer`.
4. **Distribuzione sugli endpoint:**
   * *Aziendale:* Distribuzione automatizzata tramite Criteri di Gruppo Active Directory (GPO), Microsoft Intune o FortiClient EMS.
   * *Manuale (es. Mozilla Firefox):* `Impostazioni > Privacy e Sicurezza > Certificati > Mostra certificati > Autorità > Importa...` e selezionare l'attendibilità per l'identificazione di siti web.

### 8.3 Come Importare Certificati e CA su FortiGate
* **Importare una CA esterna:** `Create/Import > CA Certificate`
  * Supporta il caricamento di file (`.cer`, `.crt`, `.pem`) o la connessione online via protocollo **SCEP** (*Simple Certificate Enrollment Protocol*).
* **Importare un Certificato Locale (con chiave privata):** `Create/Import > Certificate`
  * **Local Certificate:** Completa una CSR precedentemente generata sul FortiGate caricando il file `.cer` firmato dalla CA.
  * **Certificate and associated key file:** Caricamento separato del certificato pubblico (`.cer`) e del file con la chiave privata (`.key` / `.pem`).
  * **PKCS#12 Certificate:** Caricamento di un archivio unico (`.pfx` / `.p12`) protetto da password, contenente certificato, catena di certificazione e chiave privata.
* **Importare Liste di Revoca (CRL):** `Create/Import > CRL`
  * **Online Updating (Raccomandato):** Configura l'aggiornamento automatico e periodico specificando l'URL del server di distribuzione CRL via protocollo **HTTP**, **LDAP** o **SCEP** (es. `http://crl3.digicert.com/...`).

---

## 9. Tecnologie Crittografiche Emergenti per VPN in FortiOS

FortiOS include estensioni crittografiche avanzate per la protezione del trasporto e dell'autenticazione delle connessioni VPN:

### 9.1 TLS 1.3 VPN Key Negotiation Over TCP
* **Contesto:** In presenza di reti o provider che bloccano o degradano il traffico UDP (porte UDP 500/4500 ed ESP protocollo IP 50), FortiOS supporta l'incapsulamento del traffico VPN IPsec all'interno di una sessione TCP protetta da TLS (*IKE over TCP*).
* **TLS 1.3:** Sostituisce l'handshake TLS 1.2 offrendo un tempo di negoziazione ridotto (1-RTT rispetto a 2-RTT) ed escludendo algoritmi crittografici legacy o vulnerabili. Sia la negoziazione IKEv2 che i pacchetti dati ESP viaggiano incapsulati all'interno della connessione TLS.
* **Abilitazione da CLI:**
  ```fortios
  config system settings
      set ike-tcp-service enable
  end
  ```

### 9.2 Post-Quantum Cryptography (PQC) per VPN IPsec (IKEv2)
* **Scopo:** Proteggere la fase di autenticazione delle VPN contro attacchi futuri basati su elaboratori quantistici su larga scala (in grado di compromettere gli algoritmi asimmetrici tradizionali come RSA ed ECDSA).
* **Standard RFC 7427:** Framework flessibile e indipendente dall'algoritmo (*algorithm-agnostic*) per la gestione delle firme digitali in IKEv2.
* **Algoritmi PQC NIST integrati:**
  * **ML-DSA (FIPS 204):** Basato su reticoli algebrici (*Module-Lattice-Based Digital Signature Algorithm*).
  * **SLH-DSA (FIPS 205):** Basato su primitive di hash (*Stateless Hash-Based Digital Signature Algorithm*).
* **Configurazione di Fase 1 da CLI:**
  ```fortios
  config vpn ipsec phase1-interface
      edit "PQC"
          set authmethod signature
          set digital-signature-auth enable
          set signature-hash-alg identity
      next
  end
  ```