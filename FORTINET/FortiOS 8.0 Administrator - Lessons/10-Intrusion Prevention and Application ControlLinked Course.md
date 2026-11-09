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
  Block: La connessione viene troncata e il traffico scartato non appena si riscontra il pattern malevolo.
  Pass: Il pacchetto viene autorizzato a transitare (usato tipicamente per firme informative o a basso rischio).
  CVE-ID: Il codice standard internazionale di vulnerabilità (ad esempio CVE-2005-0277, CVE-2014-9263) per una rapida consultazione e tracciabilità con i bollettini di sicurezza.  
