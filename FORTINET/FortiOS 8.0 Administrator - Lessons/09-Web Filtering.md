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

- SSL Certificate Inspection
  Security Profiles > SSL/SSH Inspection
  Analisi dei Parametri
  # Enable SSL inspection of: Multiple Clients Connecting to Multiple Servers   
  Identifica lo scenario Outbound / Forward Proxy: protegge i client della rete interna che navigano verso molteplici server web esterni su Internet.   
  
  # Inspection method: SSL Certificate Inspection   
  Selezionato al posto di Full SSL Inspection. 
  In questa modalità il firewall non decifra il contenuto dei dati, ma analizza unicamente l'handshake TLS iniziale (SNI e certificato server) per estrarre il dominio di destinazione.
  Non richiede l'installazione di una CA aziendale sui computer degli utenti.   
  
  # Server certificate SNI check (con opzioni: Enable, Strict, Disable)   
  A cosa serve (Action if the SNI does not match the CN or SAN fields): 
  Verifica che il nome di dominio richiesto dal client nel campo SNI corrisponda a quello dichiarato nel certificato inviato dal server (campi Common Name o Subject Alternative Name).
     
  Vincolo architetturale: La nota evidenzia che questa verifica opera esclusivamente quando la policy è configurata in modalità Proxy-based (only in proxy-based inspection). 
  In modalità Flow-based questo controllo specifico non interviene.  
  
  # Protocol Port Mapping (HTTPS: 443,10443)   
  Indica al firewall su quali porte TCP intercettare il protocollo HTTPS per estrarre i certificati.   
  Come evidenziato dal callout azzurro (You can specify more than one port number, separated by comma), è possibile specificare più porte separate da una virgola (nell'esempio la porta standard 443 e la porta alternativa 10443).
  
  # Gestione degli errori nei certificati:
  
  Malicious certificates (Block): 
  Blocca il traffico se il certificato appartiene a siti identificati come malevoli dal database FortiGuard.   
  
  Untrusted SSL certificates (Block): 
  Interrompe la connessione se il certificato del server è firmato da una CA non riconosciuta o non valida.   
  
  ncrypted Client Hello (Block): 
  Opzione in basso per decidere se bloccare o consentire l'accesso se il browser utilizza ECH (Encrypted Client Hello), tecnologia che nasconderebbe l'SNI alla vista del firewall.

Come creare un profilo web filtering in base a Flow-based vs Proxy-based.
Security Profiles > Web Filter
I componenti principali configurabili sono in Flow-Based:   
- Feature set: Flow-based (selezionato di default).   
- FortiGuard Category Based Filter: Il motore primario di classificazione cloud di Fortinet per consentire, monitorare, avvisare o bloccare intere categorie di siti web.   
- Search Engines (Safe Search): Consente di forzare la modalità di ricerca sicura (Safe Search) sui motori di ricerca supportati (Google, Yahoo!, Bing, Yandex) per escludere contenuti per adulti dai risultati.   
- Static URL Filter: Permette di definire liste esplicite di URL/pattern (tramite semplici stringhe, wildcard o espressioni regolari) da autorizzare o bloccare a prescindere dalla categoria FortiGuard.   
- Rating Options: Definisce il comportamento del firewall quando i server FortiGuard sono temporaneamente irraggiungibili (Allow all websites o Block all websites) e abilita la classificazione sia per dominio che per indirizzo IP.

I componenti principali configurabili sono in Proxy-Based :
Requisito Hardware Obbligatorio:Supported on only FortiGate models with more than 2 GB RAM   
L'ispezione proxy per il Web Filter richiede la ritenzione e l'analisi dei pacchetti in memoria RAM; pertanto, FortiOS disabilita o non supporta questa modalità sui modelli entry-level dotati di 2 GB di RAM o meno.   
Funzionalità Esclusive (Feature available only in proxy based):
- Il passaggio a proxy sblocca opzioni avanzate visibili con l'icona del divieto/attenzione rossa, non disponibili in flow-mode:  
  1) Log all search keywords: Registra nei log di sistema le esatte parole chiave digitate dagli utenti nei motori di ricerca.
  2) Restrict Google account usage to specific domains: Inserisce header HTTP personalizzati per consentire l'accesso solo agli account Google aziendali (es. @tuaazienda.com), bloccando gli account Gmail personali.
  3) Rimozione di componenti attivi (Proxy Options):
     Remove Java Applets   
     Remove ActiveX   
     Remove Cookies
  Questi elementi vengono eliminati direttamente dal codice HTML prima che la pagina web venga consegnata al browser del client.


Categorie Web filtering (Security Profiles > Web Filter):
Allow (Consenti):
Il traffico verso i siti appartenenti a quella categoria viene autorizzato a passare senza restrizioni o notifiche. Di default non genera log di blocco (riduce il volume dei log).   

Monitor (Monitora):
Il traffico viene consentito esattamente come con Allow, ma viene registrato esplicitamente nei log di sistema (log event). È l'azione ideale durante le fasi di collaudo o audit di una rete per scoprire quali categorie visitano gli utenti prima di decidere se bloccarle.  

Block (Blocca):
La connessione viene immediatamente interrotta e al browser dell'utente viene inviata la pagina di blocco del FortiGate (Web Filter Block Page), indicando il motivo e la categoria incriminata.   

Warning (Avviso):
Mostra all'utente una schermata web di avviso che segnala che il sito appartiene a una categoria sconsigliata o a rischio. L'utente ha la facoltà di premere un pulsante (es. Proceed / Continua) per aggirare temporaneamente l'avviso e accedere comunque al sito per un intervallo di tempo prestabilito (utile per policy aziendali di tipo "a tuo rischio").   

Authenticate (Autentica):
La categoria è bloccata per la navigazione standard, ma presenta un modulo di autenticazione (Captive Portal / override). Se l'utente fa parte di un gruppo autorizzato (es. dirigenti, team IT) e inserisce credenziali valide, l'accesso a quella specifica categoria viene sbloccato temporaneamente (funzione di User Override).
