## ROUTING 
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
 
- Come creare una rotta statica 
  Network > Static Routes
  Come creare la defoult route : 
  Destination (Subnet vs Internet Service):
  > Subnet (selezionata nell'immagine): permette di inserire una destinazione IP con relativa maschera di rete.
  > 0.0.0.0/0.0.0.0 (Default Route): rappresenta qualsiasi indirizzo IP non appartenente a reti locali o rotte più specifiche note al firewall. Corrisponde alla via d'uscita universale verso Internet.   
  Gateway Address (Specify vs Dynamic):
  > Specify (selezionato): imposta manualmente l'indirizzo IP del router o modem del provider a cui consegnare il pacchetto (nell'esempio: 100.65.0.254).
  > Dynamic: si utilizza quando l'interfaccia di uscita riceve il proprio IP e il gateway in modo dinamico tramite DHCP o PPPoE (il firewall recupera il gateway direttamente dal server del provider senza doverlo digitare).
  Interface (port2):
  > L'interfaccia fisica o logica (VLAN, tunnel IPsec) da cui il pacchetto deve essere trasmesso fisicamente verso il next hop.
  Administrative Distance (10):
  > È la metrica di affidabilità della sorgente di routing utilizzata per popolare la RIB.   .
  > Il valore predefinito per le rotte statiche su FortiOS è 10.
  > Minore è la distanza, maggiore è la priorità: una rotta direttamente connessa ha distanza 0, una statica ha 10, OSPF ha 110, BGP esterno ha 20

- Static route with name adresses
  Fino a poco tempo fa, nella schermata delle rotte statiche era obbligatorio digitare a mano l'indirizzo IP numerico e la maschera di rete (es. 10.200.1.0/24).
  Con questo meccanismo è possibile collegare direttamente un oggetto creato in Policy & Objects alla tabella di instradamento.
  Creazione dell'Oggetto (Policy & Objects > Addresses)
  > Name: Remote_Subnet.
  > Type: Subnet (oppure FQDN).
    La levetta critica: Routing configuration :
  > Questo è il dettaglio chiave da ricordare per l'esame e la pratica: affinché un oggetto indirizzo sia visibile e selezionabile all'interno del modulo di routing, è obbligatorio abilitare il selettore Routing configuration.
    Perché esiste questo interruttore? Un firewall aziendale può contenere centinaia o migliaia di oggetti indirizzo usati solo per le policy di sicurezza. Se fossero tutti visibili nel routing, il menu a tendina diventerebbe ingestibile. Questa opzione funge da filtro     esponendo solo gli oggetti espressamente destinati all'instradamento.   
  Creazione della Rotta (Network > Static Routes)
  > Destination: Invece di selezionare Subnet o Internet Service, si preme il pulsante centrale Named Address.
  > Selezione dell'oggetto: Nel campo a tendina compare l'oggetto Remote_Subnet precedentemente abilitato.
  Campi standard di instradamento:
  > Gateway Address: 100.65.0.254 (Next Hop a cui inoltrare il traffico).
  > Interface: port2 (Interfaccia di uscita).
  > Administrative Distance: 10 (Priorità di default per le rotte statiche).

Percorso GUI: Dashboard > Network Monitor > Routing
> Mostra esclusivamente le rotte considerate attive e migliori (best routes).
> Aggrega le rotte dirette (Connected, con distanza 0), le rotte statiche (Static, distanza predefinita 10) e tutte le rotte dinamiche apprese via OSPF, BGP o RIP.
> Possimao trovare oltre le informazioni base come network, adderess, port anche type
> Type si divide in varie categorie 
   1) Connected (Reti Direttamente Connesse) > Sono le reti fisicamente collegate alle porte
   2) Static (Rotte Statiche) > Rotte inserite a mano dall'amministratore di rete
   3) I Protocolli di Routing Dinamico (IGP ed EGP)
      eBGP (External Border Gateway Protocol) > È il protocollo che governa la comunicazione tra Autonomous System (AS) tipo AWS Direct Connect, Azure ExpressRoute
      OSPF (Open Shortest Path First) > È il protocollo di tipo IGP (Interior Gateway Protocol) più diffuso nelle reti aziendali interne e nei data center
      RIP (Routing Information Protocol) > È uno dei primi protocolli dinamici della storia di Internet (Distance-Vector), oggi considerato ereditato (legacy) e raramente usato in architetture moderne

Distance 
Connect = 0 
static WAN = 1 
static DHCP = 5
static Manual = 10
EBGP = 20
OSPF = 110
IS-IS = 115
RIP = 120
IBGP = 200


- ECMP
  L'ECMP è il meccanismo che permette a FortiGate di utilizzare contemporaneamente più percorsi alternativi verso la stessa destinazione
  Requisiti Obbligatori per Attivare l'ECMP :
   1) Destination subnet (Sottorete di destinazione)
   2) Distance (Administrative Distance)
   3) Metric (Metrica)
   4) Priority (Priorità)
Configuring ECMP (Algoritmi e Parametri CLI)

config system settings
    set v4-ecmp-mode [source-ip-based | weight-based | usage-based | source-dest-ip-based]
end

ESEMPIO 

[ User A: 10.0.3.1 ]
         │
         ├─── (port3: 10.0.3.0/24) ───► [ FortiGate ] ──┬── port1 (10.0.1.0/24) ──► Gateway: 10.0.1.254 ──┐
         │                                               │                                                 ├──► [ Server 10.0.4.0/24 ]
 [ User B: 10.0.3.2 ]                                    └── port2 (10.0.2.0/24) ──► Gateway: 10.0.2.254 ──┘

Scenario Operativo
Due client interni (User A 10.0.3.1 e User B 10.0.3.2) sono attestati sulla rete LAN connessa alla porta port3.   
Entrambi devono raggiungere il data center/server remoto con subnet 10.0.4.0/24.   
Il FortiGate dispone di due collegamenti ridondati verso la rete cloud:   
Tramite port1 (subnet 10.0.1.0/24) verso il gateway 10.0.1.254.   
Tramite port2 (subnet 10.0.2.0/24) verso il gateway 10.0.2.254

Decodifica dell'Output CLI (get router info routing-table all)

S    10.0.4.0/24 [10/0] via 10.0.1.254, port1, [5/0]
                 [10/0] via 10.0.2.254, port2, [5/0]

S: rotta di tipo Statico (Static).   
10.0.4.0/24: la subnet di destinazione è identica per entrambe le righe.   
[10/0] (Distance e Metrica - Fumetto azzurro a sinistra):   
Entrambe le rotte hanno Distance = 10.   
Entrambe le rotte hanno Metric = 0.  
[5/0] (Priorità - Fumetto azzurro a destra):   
Il primo valore tra parentesi [5/...] rappresenta la Priority. Entrambe le rotte statiche hanno priorità esatta impostata su 5

Come viaggia il traffico dei due utenti?
In base all'algoritmo predefinito source-ip-based:   
> User A (10.0.3.1) genera la prima sessione verso il server: l'algoritmo calcola l'hash del suo IP e lo assegna a port1.
> Tutte le connessioni di User A verso il server viaggeranno attraverso il gateway 10.0.1.254.
> User B (10.0.3.2) avvia una connessione: il suo IP sorgente differente produce un risultato hash diverso, indirizzando il flusso verso port2 e gateway 10.0.2.254.
> Se port1 dovesse disconnettersi, la prima riga verrebbe rimossa istantaneamente dalla FIB e il 100% delle sessioni convergerebbe su port2 in automatico senza disservizi per gli utenti.   
