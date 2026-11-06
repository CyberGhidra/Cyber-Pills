## Web Filtering 
- Il Web Filtering non agisce all'inizio assoluto della connessione (quello è il compito del Routing e del controllo delle Firewall Policy), ma si attiva subito dopo che la policy ha autorizzato il traffico (azione ACCEPT) e (se configurato) dopo che il pacchetto è stato    decifrato.
  1. ARRIVO PACCHETTO ──► 2. ROUTING & POLICY MATCH (ACCEPT) ──► 3. SSL INSPECTION (Se attiva) ──► 4. WEB FILTERING (Layer 7)
     Attivazione del Web Filter: 
     Web Filtering associato a quella specifica Firewall Policy. Il motore legge l'URL, estrae il nome del dominio (o interroga il database cloud FortiGuard) e verifica se la categoria di quel sito (es. News and Media, Social Networking, Gambling) è autorizzata o bloccata.

- Come fa il FortiGate a identificare il sito web di destinazione quando è attiva la modalità SSL Certificate Inspection (senza decifratura del payload) ?.
  Nel traffico HTTPS, il contenuto della richiesta (l'URL esteso come /index.php?login=true) è protetto da cifratura. Tuttavia, durante l'avvio della sessione TLS, il firewall intercetta e legge in chiaro l'handshake iniziale per estrarre l'FQDN (Fully Qualified Domain
  Name) del server:

  1) Passo Principale — Estrazione tramite SNI (Server Name Indication):
     Nel primo passaggio (1. Client Hello), il browser dell'utente invia la versione del protocollo, le suite crittografiche supportate e l'estensione Server Name Indication (SNI).
     Il campo SNI contiene il nome host in chiaro:
     Server Name: [www.fortinet.com](https://www.fortinet.com)
     FortiGate legge questo campo prima ancora che la cifratura sia stabilita e identifica a quale sito il client vuole accedere.
  2) Passo Secondario (Fallback) — Lettura del Certificato del Server:
     Se il client non include l'estensione SNI nel Client Hello ("If SNI is not present..."), FortiOS attende la risposta del server (2. Server Hello).
     Il server remoto invia il proprio certificato digitale X.509.
     FortiGate esamina i campi del certificato, leggendo il valore del Subject / Common Name (CN) (es. CN = *.fortinet.com) o dei campi SAN (Subject Alternative Name) per risalire al dominio.


- Modalita di ispezione:
  Flow-based Inspection (Ispezione basata sul flusso):
  Modalità predefinita (Default inspection mode): È l'impostazione standard consigliata su FortiOS.
  Come gestisce il traffico (Traffic inspected while flowing through FortiGate): Il firewall non spezza la sessione TCP; il flusso transita direttamente attraverso il dispositivo mentre i pacchetti vengono analizzati in streaming.
  Prestazioni:
  - Requires fewer processing resources: Richiede un minor consumo di memoria RAM e cicli CPU.   
  - Faster scanning: L'elaborazione è più veloce e la latenza introdotta è minima.   

  Proxy-based Inspection (Ispezione basata su proxy):
  - Il FortiGate termina fisicamente la sessione TCP proveniente dal client e ne apre una seconda verso il server remoto, interponendosi come proxy.
  - Ispezione approfondita (More thorough inspection): Il traffico viene intercettato, bufferizzato ed esaminato nel dettaglio dal demone proxy di sistema.
  - Funzionalità aggiuntive (Provides additional options): Sblocca controlli avanzati non disponibili in flow-mode, come la manipolazione degli header HTTP, la rimozione sicura di script o l'integrazione di filtri più complessi.
  - Impatto hardware (More resource intensive): Comporta un maggiore utilizzo di RAM (per il buffering completo dei payload) e un consumo di CPU più elevato.   
