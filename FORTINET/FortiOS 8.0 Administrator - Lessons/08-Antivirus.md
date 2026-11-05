## Antivirus Scanning Engine
- Funzionamento del Motore Antivirus
  Database delle firme: Il motore utilizza database di firme antivirus per rilevare e neutralizzare codici malevoli (malware, virus, trojan) nei file e nel traffico in transito.
  Aggiornamenti in tempo reale: I database delle firme vengono mantenuti costantemente aggiornati in real time tramite il servizio in cloud FortiGuard Antivirus Service.

- Ordine di Scansione (Order of Scan)
  Quando un file o un payload attraversa il profilo Antivirus, FortiOS esegue l'ispezione seguendo rigorosamente tre fasi sequenziali (dalla più veloce ed economica in termini di risorse a quella più avanzata):
  a) Antivirus Scan : 
     Primo livello di analisi basato sulle firme tradizionali di malware noto. Se il file corrisponde a una minaccia conosciuta nel database FortiGuard, viene bloccato immediatamente.
  b) Grayware Scan :
     Secondo passaggio mirato a rilevare software potenzialmente indesiderato (PUP/PUA), come adware, spyware o tool che non sono virus distruttivi ma presentano rischi per la privacy o le prestazioni.
  c) AI Scan (Scansione basata su Intelligenza Artificiale) :
     Ultimo passaggio che sfrutta modelli euristiche di intelligenza artificiale/machine learning per identificare varianti sconosciute o minacce zero-day prima di consentire l'inoltro del pacchetto verso i server o client interni.
  
  - Meccanismi di Protezione Antivirus
    FortiGate adotta diversi meccanismi standard di settore per rilevare e bloccare il malware:
    a) Signature-based detection (Rilevamento basato su firme):
       è il metodo tradizionale di identificazione delle minacce note tramite il confronto dei file con il database delle firme costantemente aggiornato da FortiGuard.
    b) Virus outbreak prevention (Prevenzione epidemie di virus): 
       interviene a protezione delle nuove minacce appena diffuse su scala globale prima ancora del rilascio formale del pacchetto completo di firme, bloccando gli outbreak emergenti in tempo reale.
    c) External malware blocklist: 
       consente di integrare liste esterne di blocco (hash malevoli o indicatori di compromissione - IoC) importate da repository o feed di sicurezza di terze parti.
    d) EMS threat feed: 
       Riceve direttamente da FortiClient EMS elenchi dinamici e telemetrie su file o endpoint malevoli rilevati all'interno della rete locale per applicare blocchi coordinati sul firewall.

  - Configurazione nel Profilo Antivirus (GUI)
    Nella finestra mostrata a destra (Security Profiles > AntiVirus):
    Name: default (nome dell'oggetto profilo).
    AntiVirus scan: interruttore attivo impostato sull'azione Block (il firewall interrompe il transito dei file malevoli anziché limitarsi a registrare l'evento con Monitor).
    Feature set: impostato sulla modalità Flow-based (in alternativa a Proxy-based).
    Sezione Virus Outbreak Prevention ( ON / OFF ):
    Use FortiGuard outbreak prevention database (attivo su Block).   
    Use 0-day malware stream scanning (scansione in streaming per minacce zero-day).   
    Use external malware block list (utilizzo di liste hash malevoli esterne).   
    Use EMS external feed (connessione ai feed di minaccia generati da FortiClient EMS).
    
- Componenti Antivirus Avanzati
  Content Disarm and Reconstruction (CDR): 
  Tecnica di sanificazione dei file che rimuove gli elementi attivi potenzialmente malevoli lasciando inalterato il contenuto legittimo del documento.
  Behavior-based detection (Rilevamento basato sul comportamento):
  Analizza le azioni dinamiche e le anomalie comportamentali di un file o processo per intercettare minacce che tentano di eludere i database di firme noti.
  CIFS scanning:
  Scansione del traffico relativo alla condivisione file di rete (protocollo SMB/CIFS), essenziale per prevenire la propagazione di ransomware e malware tra share aziendali interne.
  AI/ML, behavioral, and human analysis:
  Integrazione multi-livello che combina modelli di Machine Learning, analisi euristico-comportamentale e analisi manuale/forense fornita dai FortiGuard Labs o tramite appliance dedicate come FortiSandbox e FortiNDR.  


- Antivirus Integration With Multiple FortiSandbox Destinations)
  On-premises FortiSandbox: 
  appliance fisica o macchina virtuale (VM) installata localmente all'interno dell'infrastruttura di rete aziendale (icona del server fisico con la "bomba").
  FortiSandbox Cloud:
  servizio gestito in cloud erogato direttamente da Fortinet (icona della nuvola con la "bomba")

  Configurazione:
  Abilitazione Simultanea: 
  È possibile abilitare contemporaneamente entrambe le destinazioni all'interno della configurazione (Both destinations can be enabled simultaneously).   
  Comportamento in Modalità Automatica (auto):
  Quando la destinazione FortiSandbox è impostata sul valore automatico (auto), FortiOS seleziona e inoltra i campioni verso il dispositivo contrassegnato come predefinito (default).  

















