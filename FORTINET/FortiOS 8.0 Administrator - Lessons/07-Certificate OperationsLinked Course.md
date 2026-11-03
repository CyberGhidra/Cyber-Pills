<img width="634" height="297" alt="image" src="https://github.com/user-attachments/assets/d67d435a-386b-4b8d-aeea-8e1c04ef8e3b" />## Digital certificate 

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












