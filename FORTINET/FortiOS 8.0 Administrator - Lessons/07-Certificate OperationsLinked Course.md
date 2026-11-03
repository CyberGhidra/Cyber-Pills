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
