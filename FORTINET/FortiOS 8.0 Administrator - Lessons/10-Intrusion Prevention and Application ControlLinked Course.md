## Intrusion Prevention and Application ControlLinked Course
- I tre elementi cardine che costituiscono il motore IPS:
  a) IPS signature databases (Database delle firme IPS):
     raccolta di pattern e firme costantemente aggiornate (tramite i servizi di intelligence FortiGuard) per riconoscere attacchi noti, CVE ed exploit diffusi.
  b) Protocol decoders (Decodificatori di protocollo): analizzano i singoli protocolli di rete (es. HTTP, DNS, SMB, FTP) verificando che il traffico rispetti gli standard (RFC) e decodificando il flusso per consentire all'engine di esaminare i campi specifici del
     pacchetto.
  c) IPS engine: il motore di elaborazione che orchestra l'ispezione, confrontando i pacchetti decodificati con le firme e applicando gli algoritmi di rilevamento in tempo reale.

- Flusso Operativo Nello Schema GraficoMinaccia esterna (Attaccante / Hacker): 
  1) tenta di inviare traffico malevolo, vettori di attacco o exploit verso l'infrastruttura.
  2) FortiGate con IPS attivo: il traffico attraversa le interfacce e viene intercettato dal motore IPS. I decoder di protocollo e il database delle firme scansionano il flusso di dati.
  3) Blocco e Protezione: il traffico contenente l'exploit o l'anomalia viene interrotto e scartato (icona dell'attaccante barrata).
  4) Destinazioni Protette: solo il traffico legittimo e pulito raggiunge le sedi aziendali (edifici industriali/uffici) o i server e i datacenter interni.
 
- Percorso e Gestione dei Sensori IPS
  Percorso GUI: Security Profiles > Intrusion Prevention.
  IPS Sensors: All'interno di questa sezione è possibile creare nuovi sensori personalizzati (Create new) oppure visualizzare e modificare i sensori predefiniti forniti da FortiOS (come il profilo default mostrato a sinistra).
  Pulsante View IPS Signatures: Cliccando sul link/pulsante, si apre la vista dettagliata dell'intero database delle firme IPS caricate sul firewall.

  Metriche e Classificazione del Database Firme (Grafici a ciambella)
  Severity (Gravità dell'impatto): suddivide le firme in base al livello di rischio dell'exploit:   
  Critical - High - Medium - Low  - Information   
  Target (Bersaglio dell'attacco): distingue se la vulnerabilità sfrutta componenti lato Server (servizi in ascolto esposti) oppure lato Client (browser, software locali, client mail).
  OS (Sistema Operativo): cataloga le firme in base all'ambiente target dell'exploit (Windows, Linux, macOS, Solaris, ecc.).

  Anatomia della Tabella Firme
  Name: Il nome univoco identificativo della vulnerabilità o dell'exploit (ad es. 3Com.3CDaemon.FTPServer.Buffer.Overflow).
  Severity: Il livello di gravità associato alla specifica minaccia.
  Target: L'obiettivo della firma (Server o Client).
  OS: La piattaforma software a cui si applica la vulnerabilità (Windows, Linux, ecc.).
  Action (Default action): L'azione predefinita associata dal team di sicurezza FortiGuard a quella singola firma:
    1) Block: La connessione viene troncata e il traffico scartato non appena si riscontra il pattern malevolo.
    2) Pass: Il pacchetto viene autorizzato a transitare (usato tipicamente per firme informative o a basso rischio).
  CVE-ID: Il codice standard internazionale di vulnerabilità (ad esempio CVE-2005-0277, CVE-2014-9263) per una rapida consultazione e tracciabilità con i bollettini di sicurezza.  

- Percorso e Configurazione Base del Sensore
  Percorso GUI: Security Profiles > Intrusion Prevention.
  New IPS Sensor: creazione di un nuovo profilo/sensore personalizzato (nell'esempio denominato IPS_profile).
  Pulsante + Create New: all'interno della tabella IPS Signatures and Filters, premendo il pulsante si apre la finestra di dialogo per inserire le regole. È possibile scegliere tra due tipologie operative tramite il selettore Type (Filter o Signature).

  Metodo 1: Add Individual Signatures (Aggiunta di Singole Firme)
  Finestra in alto (Type: Signature):
    - selezione puntuale: Permette di cercare e selezionare manualmente specifiche firme dal database (tramite nome, CVE o barra di ricerca).
    - Metodo 1 --- Parametri configurabili sulla singola firma:
        - Action: stabilisce il comportamento (Default, oppure forzato su Pass o Block).
        - Packet logging: abilita o disabilita il salvataggio dei pacchetti che hanno innescato l'evento per successive analisi forensi / pcap (Enable / Disable).
        - Status: stato operativo della firma (Enable, Disable, Default).
        - Rate-based settings: impostazioni di soglia per attacchi volumetrici o brute-force.
        - Exempt IPs: definizione di eccezioni per escludere specifici indirizzi IP dall'ispezione della regola
     
  - Metodo 2 --- Add Groups of Signatures Using Filters (Filtri Dinamici)
        - Creazione di gruppi dinamici: Anziché selezionare le firme una a una, si impostano filtri basati su criteri di classificazione (es. severità, protocollo, sistema operativo, target).
        - Manutenzione automatica: Tutte le nuove firme rilasciate e scaricate tramite i servizi FortiGuard che corrispondono a questi criteri vengono incluse automaticamente nel sensore senza alcun intervento manuale.


- Rate-Based Signatures
  - A differenza delle normali firme IPS che intervengono al primo pacchetto corrispondente al pattern malevolo, le rate-based signatures permettono di monitorare e bloccare il traffico solo quando viene superata una soglia prestabilita entro una determinata finestra       temporale (Add rate-based signatures to block traffic when the threshold is exceeded during a time period).
  - Parametri di Configurazione
    1) Sotto la voce Rate-based settings, selezionando Specify, si sbloccano i parametri di soglia che si applicano alle firme selezionate nella tabella in basso (These parameters are applicable to the signatures selected at the bottom):
       a) Threshold (Soglia): Definisce il numero massimo di tentativi o corrispondenze ammesse prima di far scattare la contromisura (nell'esempio il campo è editabile).
       b) Duration (seconds) (Intervallo di tempo): Stabilisce l'arco temporale (in secondi, impostato a 60 nell'immagine) entro cui viene calcolato il superamento della soglia.
    2) Criterio di Tracciamento (Track By)
       a) Source IP: Conta le occorrenze generate dallo stesso indirizzo IP sorgente. È la scelta ideale contro tentativi di attacco brute force o port scan provenienti da un singolo attaccante.
       b) Destination IP: Conta le occorrenze dirette allo stesso server/IP di destinazione, utile per proteggere una specifica risorsa da attacchi distribuiti a bassa intensità.   
       c) Any: Considera entrambi i criteri o qualsiasi combinazione di traffico corrispondente alla firma.  
       
