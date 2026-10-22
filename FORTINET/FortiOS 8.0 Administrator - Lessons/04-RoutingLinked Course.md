<img width="670" height="364" alt="image" src="https://github.com/user-attachments/assets/85ac754d-f426-4022-9829-6c238ceffd28" />## ROUTING 
- FortiGate in Modalità NAT (Routed Mode)
  Quando il FortiGate è configurato nella modalità operativa predefinita (NAT Mode, detta anche Routed Mode), opera come un router IP a tutti gli effetti.
  Inoltro tra reti diverse
  Supporto IPv4 e IPv6
- Firewall Traffic vs Local-Out Traffic
  1) Firewall Traffic = È il traffico generato da endpoint, server o client esterni che attraversa il firewall
                        Prima ancora di valutare le regole di sicurezza (Firewall Policy), il firewall deve effettuare un Routing Lookup per individuare l'interfaccia di uscita. Se non esiste una rotta valida verso la destinazione, il pacchetto viene scartato
                         immediatamente per unreachable route senza nemmeno consultare le policy
  
  2) Local-Out Traffic = È il traffico che ha come sorgente il FortiGate stesso. Rientrano in questa categoria le connessioni verso i server FortiGuard, l'invio dei log verso FortiAnalyzer o Syslog etc
                         Anche per questo traffico FortiOS consulta la tabella di routing per sapere da quale interfaccia uscire e quale indirizzo IP locale adottare come sorgente

- Come funziona la decisione di Routing e il Next Hop
  Analizza l'indirizzo IP di destinazione contenuto nell'header del pacchetto e consulta la tabella di inoltro per determinare due elementi chiave:
    1) Outgoing Interface (Interfaccia di uscita): La porta fisica o virtuale da cui il pacchetto deve essere trasmesso verso il segmento successivo
    2) Gateway / Next Hop (Salto successivo): L'indirizzo a cui consegnare il pacchetto.
       La slide specifica che il next hop può trovarsi in due condizioni:
       a) Un altro router lungo il percorso: Tipico scenario di uscita verso Internet o verso una WAN aziendale. Il FortiGate non conosce l'infrastruttura successiva, quindi invia il frame Ethernet al MAC address del router del provider (il default gateway 0.0.0.0/0)
       b) La destinazione finale stessa: Accade quando la destinazione appartiene a una sottorete direttamente connessa a una delle interfacce del FortiGate (rotta di tipo Connected). In questo caso il next hop è l'host finale e il FortiGate risolve direttamente il suo              indirizzo MAC tramite richiesta ARP.

- La tabella di Routing
  La tabella di routing contiene l'insieme delle rotte apprese dal firewall (connesse, statiche o dinamiche come OSPF e BGP), associando a ogni rete di destinazione il relativo Next Hop e l'interfaccia di uscita.
  Quando un pacchetto deve essere instradato, FortiOS esegue il Route Lookup (ricerca del miglior percorso) basandosi su una gerarchia di criteri molto rigorosa:
  a) Best Route: Longest Prefix Match (La rotta più specifica) ---> vince sempre quella con la subnet mask più lunga/stretta
     esempio:
     sempio pratico: Il client vuole raggiungere l'IP 10.50.1.10. Nel FortiGate sono presenti tre rotte:

     0.0.0.0/0 (Default route - prefisso /0)
     10.50.0.0/16 (Prefisso /16)
     10.50.1.0/24 (Prefisso /24) ---> vince questa

  b) Duplicate Routes: Criteri di spareggio
     Cosa succede se esistono due o più rotte verso la stessa identica subnet (es. due rotte per 10.50.1.0/24) provenienti da fonti diverse o configurate su gateway differenti?
     +------------------------------------------------+
               |  1. Longest Prefix Match (Subnet più stretta)  |
               +-----------------------+------------------------+
                                       | (Se il prefisso è identico)
                                       v
               +------------------------------------------------+
               |     2. Distance (Administrative Distance)      |
               +-----------------------+------------------------+
                                       | (Se la Distance è identica)
                                       v
               +------------------------------------------------+
               |            3. Metric (Costo rotta)             |
               +-----------------------+------------------------+
                                       | (Se la Metric è identica)
                                       v
               +------------------------------------------------+
               |      4. Priority (Specifico di FortiOS)        |
               +-----------------------+------------------------+
                                       | (Se tutto è identico)
                                       v
               +------------------------------------------------+
               |           ECMP (Bilanciamento attivo)          |
               +------------------------------------------------+


- Route Look up
  Per ogni nuova connessione, il FortiGate non calcola il percorso una volta sola, ma interroga la tabella di routing esattamente due volte:
  
  [ PC Client ]               [ FortiGate ]               [ Web Server ]
      │                           │                           │
      │ ─── 1. SYN ─────────────► │                           │
      │                  [1° ROUTE LOOKUP]                    │
      │                  [POLICY LOOKUP]                      │
      │                           │ ─── 1. SYN ─────────────► │
      │                           │                           │
      │                           │ ◄── 2. SYN/ACK ────────── │
      │                  [2° ROUTE LOOKUP]                    │
      │                  [SCRITTURA SESSIONE]                 │
      │ ◄── 2. SYN/ACK ────────── │                           │
      │                           │                           │
      │ ─── 3. ACK (Dati...) ───► │                           │
      │                    (FAST PATH / NPU)                  │
      │                   [NESSUN LOOKUP!]                    │
      │                           │ ─── 3. ACK (Dati...) ───► │

- Momento 1: Arriva il primo pacchetto (SYN) dall'andataIl
  tuo PC spara il pacchetto TCP SYN verso Google ed entra nella porta LAN del FortiGate.
  Il FortiGate lo riceve e si ferma un istante: esegue il 1° Route Lookup.
  Controllo: «Google ha IP 142.250.180.206. Da dove lo faccio uscire?»
  Risposta della tabella: «Usa la porta WAN1 verso il gateway dell'operatore».
  Con questa informazione in mano, controlla la Firewall Policy, applica il NAT e fa uscire il SYN verso Google
  
- Momento 2: Arriva il primo pacchetto di risposta (SYN/ACK) dal ritorno
  Google riceve il SYN e risponde con il pacchetto TCP SYN/ACK.
  Il pacchetto bussa alla porta WAN1 del FortiGate.
  Il FortiGate lo riceve e fa il controllo che ti ha incuriosito: esegue il 2° Route Lookup!
  Perché lo fa se sa già da dove era partito?
  Per motivi di sicurezza e anti-spoofing (chiamato RPF - Reverse Path Forwarding). Il firewall verifica: «Per raggiungere il PC 10.0.1.10, la mia tabella di routing dice davvero che devo passare dalla porta LAN interna?»
  Se la risposta è Sì: il percorso è coerente, valida la sessione e consegna il SYN/ACK al PC.
  Se ci fosse un'incongruenza (es. routing asimmetrico errato o IP contraffatto): scarterebbe il pacchetto per sicurezza.

- Momento 3: Cosa succede subito dopo?
  Il FortiGate scrive una riga indelebile nella Session Table (RAM):"Sessione ID 1234: L'andata esce da WAN1; il ritorno esce da LAN"
  Il PC risponde con il 3° pacchetto (ACK) e poi partono tutti i dati veri e propri (la richiesta HTTP/HTTPS della pagina web).
  Per tutti questi milioni di pacchetti successivi, il FortiGate non apre più la tabella di routing!
  Guarda direttamente la voce già pronta nella Session Table (o fa gestire tutto direttamente al chip hardware NPU/FastPath a velocità di linea).
  
  una volta che il pacchetto torna non si limita a andare a ritroso ma fa un altro confronto con la routing table

- RIB (Routing Information Base) e FIB (Forwarding Information Base)
  La RIB è la classica tabella di instradamento di sistema, gestita dal demone di routing di FortiOS
  Cosa contiene:
  - Rotte connesse (Connected) alle interfacce fisiche o logiche.   
  - Rotte statiche (Static).   
  - Rotte apprese tramite protocolli dinamici (OSPF, BGP, RIP)
  
- La FIB (Forwarding Information Base) — Il Piano di Inoltro del Kernel
  La FIB è la tabella di instradamento vista direttamente dal Kernel di FortiOS ed è la struttura dati ottimizzata per la commutazione hardware e software in tempo reale.
  A cosa serve:
  - È la tabella che il FortiGate interroga concretamente quando esegue i Route Lookups per ogni nuova sessione
  Da cosa è composta:
  - Da quasi tutte le voci presenti nella RIB.
  - In più, contiene voci specifiche di sistema/kernel (system-specific entries), come rotte interne di gestione, indirizzi broadcast locali

