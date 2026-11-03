## Digital certificate 

- COSE' : 
  È un file crittografico che funge da documento di identità digitale, generato e firmato digitalmente da un'autorità terza fidata denominata Certificate Authority (CA).
- Scopi Primari:  
  a) Authentication (Autenticazione): Accerta l'identità del server o del client che partecipa alla sessione crittografica, proteggendo da tentativi di spoofing o attacchi Man-in-the-Middle.
  b) Encryption and decryption (Cifratura e Decifratura): Fornisce la chiave pubblica necessaria per negoziare chiavi di sessione simmetriche, rendendo inintelligibile il traffico durante il transito (ad esempio in connessioni HTTPS, SSL-VPN o tunnel IPsec con
  certificati).
  c) Integrity (Integrità): Garantisce che le informazioni scambiate non siano state manipolate, alterate o corrotte durante il percorso tra sorgente e destinatario.
- Come FortiGate identifica Dispositivi e Persone:
  FortiOS adotta lo standard X.509 versione 3 (X.509v3). Per associare il certificato all'identità specifica di una persona, di un'interfaccia, di un dominio o di un apparato, FortiGate ispeziona due campi chiave:
  a) Subject: Il nome distintivo principale (DN) dell'entità a cui il certificato è stato rilasciato. Nella schermata a destra è valorizzato con training.fortinet.com.
  b) Subject Alternative Name (SAN): Un'estensione essenziale di X.509v3 che consente di specificare host o domini aggiuntivi/alternativi (es. DNS Name = training.fortinet.com), indirizzi IP o email.
     I browser e i sistemi moderni usano prioritariamente il SAN per validare l'identità del server.
- Anatomia dei Campi Evidenziati:
  Serial number: L'identificativo esadecimale univoco generato dalla CA emittente per tracciare e revocare quel singolo certificato (es. all'interno di una CRL o tramite OCSP).
  Valid from / Valid to: La finestra temporale di validità. Se la data di sistema del FortiGate (gestita via NTP) si trova prima di Valid from o dopo Valid to, il certificato viene considerato non valido e la negoziazione SSL fallisce.
  Subject e Subject Alternative Name: I valori che FortiOS confronta con il nome host o il dominio a cui si connette l'utente o il servizio.

- Raggruppamento per pilastri per uso dei certificati:
  1. Inspection (Ispezione del Traffico Cifrato):
    SSL/SSH and HTTPS traffic inspection
    Inbound or outbound traffic through FortiGate
    Traffic to and from FortiGate
  2. Privacy (Riservatezza e Protezione dei Dati in Transito)
  3. Authentication (Autenticazione e Controllo degli Accessi)
     User authentication for network access
     User authentication for VPN connection
     As second-factor authentication for FortiGate administrator

- Come il fortigate verifica la firma digitale
  Il Principio di Base
  Una firma digitale non è altro che un valore hash crittografato con la chiave privata dell'emittente (la Certificate Authority - CA).
  Per convalidare il certificato, FortiGate deve ricalcolare l'hash in locale e confrontarlo con l'hash originale decifrato.
  Le 3 Fasi della Verifica (Passo per Passo):
  
  [ FASE 1: Calcolo Hash Locale ]
  Certificato del Server ──► Funzione Hash (stesso algoritmo) ──► Nuovo Hash (10101)
  Fase 1: 
  Calcolo del nuovo Hash da parte del FortiGateIl server presenta il proprio certificato digitale a FortiGate.   
  FortiGate prende il contenuto del certificato e lo passa all'interno della stessa identica funzione hash (es. SHA-256) utilizzata dal firmatario.   
  Il risultato è un Fresh hash result (nell'esempio: 10101), calcolato in tempo reale dal firewall sul file appena ricevuto. 

  [ FASE 2: Estrazione Hash Originale ]
  Firma Digitale (Sigillo) ──► Decifratura con Chiave Pubblica CA ──► Hash Originale (10101)
  Fase 2: 
  Decifratura dell'Hash originale con la Chiave Pubblica della CAIl certificato contiene la firma digitale della CA (il "sigillo di cera" rosso).
  FortiGate possiede già nel proprio archivio fidato il certificato della CA emittente (e quindi la sua chiave pubblica).
  Utilizzando la chiave pubblica della CA, FortiGate decifra la firma digitale.
  L'operazione rivela il risultato hash originariamente generato da chi ha emesso il certificato (Original hash result produced by the signer, nell'esempio: 10101).
   
  [ FASE 3: Confronto ]
  Nuovo Hash (10101)  ==  Hash Originale (10101)  ──►  Verifica Riuscita! (Match)
  Fase 3: 
  FortiGate confronta i due valori:
  Fresh hash result {?}{=} Original hash result
  i due valori coincidono (10101 == 10101):
  
  Autenticità garantita: Solo la CA in possesso della corrispondente chiave privata poteva aver creato quella firma decifrabile con la sua chiave pubblica.   
  Integrità garantita: Nessun dato all'interno del certificato è stato manomesso o alterato in transito (anche la minima modifica cambierebbe completamente l'output della funzione hash).  
  
  Se i due valori non coincidono: Il certificato viene considerato corrotto, contraffatto o non attendibile, e la connessione sicura viene interrotta.

Traffico Cifrato in Assenza di Ispezione SSL (Encrypted Traffic With No SSL Inspection)
  1. Il Problema Fondamentale (The Blind Spot)
     Cloaked by encryption: Quando il traffico è cifrato end-to-end tra il client e il server remoto, il payload (il contenuto effettivo della comunicazione) è illeggibile per i dispositivi intermedi.
     Viruses can pass through network defenses: Se un utente scarica un file infetto, un trojan o un ransomware tramite un canale cifrato (come [https://example.com](https://example.com)), il malware attraversa il firewall senza essere intercettato, a meno che non
     venga abilitata la full SSL inspection (Deep Packet Inspection)
  2. Analisi dell'Architettura
     
     Client (Bob):
     Bob avvia una sessione HTTPS verso [https://example.com](https://example.com).
     Il client interroga il proprio archivio certificati locale (Certificate store).
     La CA fidata locale valida il certificato presentato dal server remoto.

     FortiGate (al centro):
     L'icona del lucchetto con le frecce (SSL) indica che il firewall vede transitare unicamente un tunnel TLS/SSL opaco.
     Non avendo le chiavi di sessione per decifrare il traffico, FortiGate opera come un semplice router di livello 3/4 o si limita a leggere i metadati in chiaro (SNI / handshake iniziale).
     Conseguenza: I motori UTM e i Security Profiles avanzati (come AntiVirus, File Filter, DLP e scansione approfondita del Web Filter) non possono scansionare i dati in transito e risultano inefficaci contro minacce veicolate dentro la sessione HTTPS.

     Web Server ([https://example.com](https://example.com)):
     Possiede la propria chiave privata e il proprio certificato digitale firmato da una CA pubblica.
     Termina la sessione crittografica direttamente con il computer di Bob. 
     Se il server ospita malware o payload dannosi, questi arrivano direttamente sul PC della vittima all'insaputa del firewall.

     Soluzione:
     Per consentire ai profili di sicurezza (AntiVirus, IPS, File Filter) di bloccare i virus in transito, è obbligatorio configurare una policy con Full SSL Inspection (Deep Packet Inspection), dove FortiGate si interpone terminando la sessione SSL, ispezionando il
     contenuto decifrato e ri-cifrandolo prima di inoltrarlo al client
     
     
SSL Certificate Inspection e Full SSL Inspection
- 1: SSL Certificate Inspection
     Questa modalità esegue un controllo superficiale senza aprire i dati crittografati.
     Principio di funzionamento:
     FortiGate non decifra il trafficoCome identifica la destinazione: Estrae l'FQDN (il nome di dominio del sito) da due elementi scambiati in chiaro durante l'handshake iniziale.
     La sessione TLS/SSL tra client e server rimane diretta e ininterrotta end-to-end.
     Casi d'uso e limiti:
     È utilizzabile per Web Filtering (blocco o autorizzazione di intere categorie o domini) e per Application Control di base. Non consente scansioni a livello di file (AntiVirus, DLP) perché il payload rimane cifrato
- 2: Full SSL Inspection (Deep Packet Inspection)
     Principio di funzionamento: FortiGate opera come un proxy Man-in-the-Middle autorizzato (acts as a man-in-the-middle proxy).
     Due sessioni SSL separate: Il firewall spezza il canale TLS in due connessioni distinte e indipendenti:
     a) Client-to-FortiGate: Tra il browser dell'utente e il firewall.
     b) FortiGate-to-server: Tra il firewall e il server web remoto di destinazione ([https://www.example.com](https://www.example.com)).
     Protocolli: 
     HTTPS: Traffico web cifrato (TCP 443).
     SMTPS: Trasmissione sicura di posta elettronica verso server SMTP (es. TCP 465 o 587).   
     POP3S: Scaricamento sicuro di posta da client (TCP 995).   
     IMAPS: Sincronizzazione avanzata e sicura di caselle di posta (TCP 993).   
     FTPS: Trasferimento file cifrato tramite estensioni SSL/TLS (es. TCP 990 o sessioni TLS esplicite).

      [ Network user ]                                     [ Web server ]
              │                                                    │
              │ ◄────────── HTTPS session 1 ──────────►            │
              │             (Encrypted packet)                     │
              │                                                    │
              ▼                                                    ▼
      ┌─────────────────────────────────────────────────────────────────┐
      │                           FORTIGATE                             │
      │                                                                 │
      │   [Decryption] ──► [Decrypted packet] ──► [Content scanning]   │
      │                             │                                   │
      │                             ▼                                   │
      │                    [Re-encryption]                              │
      │                                                                 │
      └─────────────────────────────────────────────────────────────────┘
                                       │
                                       ◄────── HTTPS session 2 ─────────►
                                              (Encrypted packet)


  TLS 1.3 VPN Key Negotiation Over TCP
  Questa slide presenta l'integrazione di TLS 1.3 all'interno dei meccanismi di incapsulamento VPN su canale TCP.
  Contesto e Funzionamento (IKE / ESP over TCP):
  Perché una VPN su TCP?
  Nativamente, IPsec opera su protocolli e porte UDP (IKE su UDP 500/4500 ed ESP protocollo IP 50). In presenza di firewall intermedi restrittivi, captive portal o reti che bloccano il traffico UDP, FortiOS incapsula l'intero traffico VPN
  all'interno di una sessione TCP protetta da TLS.
  Cosa transita nel tunnel TLS:
  a) IKEv2 (negoziazione chiavi e fase di autenticazione) viene trasportato dentro la connessione TLS.
  b) ESP (Encapsulating Security Payload - i dati effettivi del tunnel) viaggia incapsulato nel medesimo canale TLS (IKEv2 and ESP traffic are transported within the TLS connection).
  Il Passaggio a TLS 1.3 :
  FortiOS introduce l'opzione TLS 1.3 in sostituzione del precedente handshake basato su TLS 1.2.
  Vantaggi:
  Security: Rimozione di algoritmi di cifratura e funzioni di hashing legacy/vulnerabili (es. SHA-1, RC4, modalità CBC).
  Performance: Handshake più rapido (1-RTT rispetto ai 2-RTT richiesti da TLS 1.2 per completare la negoziazione delle chiavi).

  Da cli :

  config system settings
    set ike-tcp-service enable
  end


  Post-Quantum Cryptography (PQC) for VPNs
  Questa affronta la protezione a lungo termine contro la minaccia dei computer quantistici (Post-Quantum Cryptography - PQC) applicata all'autenticazione delle VPN IPsec basate su IKEv2.
  Architettura e Standard Implementati in FortiOS :
  RFC 7427 (IKEv2 Digital Signature Framework): Standard IETF che definisce un'architettura flessibile e algorithm-agnostic (indipendente dallo specifico algoritmo crittografico) per negoziare firme digitali in IKEv2
  Superamento di RSA ed ECDSA: Consente a FortiOS di autenticare le fasi IKEv2 senza vincolarsi esclusivamente agli schemi a chiave pubblica convenzionali.
  Algoritmi PQC standardizzati dal NIST integrati in FortiOS:
  ML-DSA (FIPS 204)
  SLH-DSA (FIPS 205)

  Da cli :

  config vpn ipsec phase1-interface
    edit "PQC"
        set authmethod signature
        set digital-signature-auth enable
        set signature-hash-alg identity
    next
  end


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









