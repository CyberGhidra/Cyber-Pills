## Firewall Policies and NAT

- Partiamo dalla Dashboard e con fortiview. Andiamo a parlare nello specifico della parte Session.
- In questa parte andiamo a prendere tutte le info memorizzate nei flussi di comunicazione
  Come ad esempio:
  indirizzo e porte
  interfaccia di ingresso e di uscita
  Nat ----> da SNAT ( source Nat ) fino a DNAT ( Destination NAT/Virtual IP)

  esempio :
  Source: indirizzo IP del client locale (es. 10.0.11.50).
  Device: identificativo dell'endpoint.
  Destination Address: IP pubblico o host remoto di destinazione.
  Application: applicazione Layer 7 riconosciuta (es. tcp/443(Google-Gmail), Akamai-CDN, Amazon-AWS).
  Protocol: protocollo di trasporto impiegato (es. tcp).
  Source Port / Destination Port: porta effimera del client e porta di ascolto remota (es. 443).
  Bytes / Packets: volume di traffico e numero di pacchetti scambiati.
  Duration: durata attiva della connessione (in secondi).  
  NPU Accelerated: indica se il traffico viene scaricato ed elaborato dai chip hardware di accelerazione (ASIC di tipo Network Processor) oppure gestito via CPU.

  Per vederlo da CLI :
  
  diagnose sys session list

- Cos'è una Firewall Policy ?
- sono regole che usa il firewall per instradata pacchetti o per bloccarli
- Ogni regola di firewall stabilisce due elementi essenziali:
  - Which traffic matches them (Identificazione del traffico): definisce i criteri affinché un pacchetto corrisponda alla regola.
    esempio interfaccia ingresso, ip, porta etc...
  - How to process matching traffic (Azione ed elaborazione): definisce cosa fare quando il pacchetto rispetta i criteri sopra indicati
    accetta o rifiuta
    applica il NAT

- LOGICA FIREWALL:
- Quando arriva il primo pacchetto di una nuova sessione IP, FortiGate segue una logica rigorosamente sequenziale
  - Scansione dall'alto verso il basso
  - matcha la regola corretta
  - esegue quello che la rogola fa
- In fondo a qualsiasi tabella di regole è sempre presente la policy predefinita Implicit Deny ---> From / To: any verso any
                                                                                                    Source / Destination: all verso all
                                                                                                    Action: DENY
  in poche parole non trova nulla scarta il pacchetto

- Se andiamo su Policy and Objects troviamo quello che ci offre la policy
  - Una regola di firewall non usa solitamente valori grezzi scritti direttamente al suo interno, ma richiama oggetti modulari e riutilizzabili definiti nella sezione Policy & Objects
      - Interface and zone: interfacce fisiche, interfacce logiche (es. VLAN) o raggruppamenti di più porte in un'unica entità logica (Zone).
      - Address, user, and internet service objects
        Address: oggetti IP singoli
        User: account utente locali o gruppi
        Internet Service Objects (ISDB): database proprietario di Fortinet che raggruppa indirizzi IP pubblici, porte e protocolli usati da servizi cloud e web noti
        Service definitions: porte e protocolli di Livello 4 (TCP, UDP, ICMP o SCTP)
        e molto altro
  - Tipologie di Policy:
     - Firewall policy (IPv4, IPv6)
     - Firewall virtual wire pair policy ----> regola il traffico che passa attraverso una coppia di porte fisiche collegate virtualmente "filo-a-filo" (Virtual Wire Pair) in modalità trasparente/Layer 2, senza routing né indirizzi IP sulle interfacce
     - Proxy policy: utilizzata quando il traffico viene ispezionato tramite architettura proxy esplicita o trasparente (Forward Proxy / Reverse Proxy)
     - Multicast policy (IPv4, IPv6): policy dedicate alla gestione e all'instradamento del traffico multicast (da uno a molti, es. video streaming in tempo reale, routing OSPF/PIM).
     - Local-in policy: regola il traffico destinato direttamente agli indirizzi IP del FortiGate stesso
     - DoS policy (IPv4, IPv6): policy specializzate nel rilevamento e blocco precoce di attacchi di tipo Denial of Service
     - Traffic Shaping: regole di gestione e limitazione della larghezza di banda

  - Configure firewall policies
  - Il primo requisito è il nome della policy ( campo obbligatorio e univoco )
  - Se pero non si desidera assegnare obbligatoriamente un nome alla policy allora :
    System > Feature Visibility abilitando ALLOW Unnamed policies
  - i capi posso essere popolati in due modi ( campi come Source, Destination o Service ) in due modi:
    - clicking ovvero si apre una lista di campi creati
    - Drag and drop ovvero trasciare gli oggetti dentro la policy
  - Da CLI :

  config firewall policy
    edit 1
        set name "training"
        set uuid 2220....etc

  - Affinche la logica funzioni deve soddisfare la condizione AND :
    - Incoming and outgoing interfaces: l'interfaccia fisica o logica da cui entra il traffico (es. port4) e quella da cui        deve uscire in base alla tabella di routing (es. port2).
    - Source: l'indirizzo IP, subnet, range IP, FQDN o servizio Internet da cui origina la connessione (nell'esempio:
      HQ_SUBNET).
    - User/group: l'identità dell'utente o gruppo (nell'esempio: utente guest). Se questo campo è popolato, il traffico
      deve provenire da un utente formalmente autenticato.
    - Destination: l'indirizzo IP, subnet, FQDN o oggetto geografico di destinazione (nell'esempio: Remote_HQ).
    - Services: il protocollo e la porta Layer 4 della richiesta (nell'esempio: porte standard HTTP 80 e HTTPS 443).
    - Schedules: la finestra temporale o il calendario di validità della regola (nell'esempio: always, sempre attiva).
  - Un volta controllato tutto che sia ok esegue l'azione finale ovevro ACCEPT o DENY
  - Poi esegue le fasi successive ovvero 
    - auth
    - security profile = esempio antivirus, web filter, ips o app control
    - logging = sessioni tracciate
   
  - Quando andiamo a creare una policy > vicino a Source & Destination > Show logic / Hide logic 
    Ci mostra le metriche di AND, OR o NOT
  - Generalmnete i capi incoming e outgoing interface sono limitati a una singola selezione
    Per sbloccarla basta andare su system > feature visibility e impostare allow

  - Nella creazione di una policy, la definizione dell'indirizzo è obbligatroria 
    Le tipologie supportate in questo campo sono:
    - IP address or range: singolo indirizzo IP (host /32) o intervallo continuo di IP (es. da 192.168.1.10 a
      192.168.1.50).
    - Subnet (IP/netmask): un'intera sottorete di rete definita con indirizzo di rete e subnet mask (es. 10.0.1.0/24
      10.0.1.0/255.255.255.0).
    - FQDN (Fully Qualified Domain Name): nomi a dominio risolti dinamicamente dal firewall.
    - Geography: oggetti basati sulla geolocalizzazione IP per nazione.
    - Dynamic (Fabric connector address): oggetti dinamici popolati automaticamente tramite connettori Fabric (es. istanze
      cloud AWS, Azure, VMware, Kubernetes).
    - MAC address range: filtraggio a livello di indirizzo fisico MAC o intervalli di MAC address.
    - ISDB (Internet Service Database): database Fortinet di servizi Internet noti.
  - Autenticazione Utente/Gruppo Opzionale

  - TAG e TAG ADDRESS GROUPS ( facilita la ricerca quando cerche una policy con Tag == <nome_del_tag> )
  - Serve per etichettare oggetti di rete per far si che gestisci aggregazione
  - Creazione del Tag (Policy & Objects > Tags)
    Name: il nome per esteso del tag (nell'esempio: ZTNA-Address).
    Abbreviation: sigla abbreviata opzionale utile per la visualizzazione compatta (nell'esempio: ZTA).
    Color: colore di evidenziazione grafica per riconoscerlo a colpo d'occhio nelle tabelle.
    Comments: note descrittive facoltative.

    Nella sezione Organization & Display, all'interno del campo Tags, viene selezionato il tag precedentemente creato: ZTA
    ZTNA-Address.
    Display using: consente di scegliere come mostrare il tag nella GUI (tutti i tag, solo il primo tag oppure solo l'icona
    colorata).
    Preview: mostra l'anteprima visiva di come apparirà l'oggetto nelle policy e nelle tabelle (ZTA Training-ZTNA).

- Geographic-Based ISDB ( serve per limitare i servizi come outlook solo in determinate posizioni geografiche e poi
  richiamrlo da policy)
  che consente di combinare il database dei servizi Internet di Fortinet (ISDB) con parametri geografici per un controllo
  di sicurezza e instradamento molto granulare
- Per compilare:
  Name: nome assegnato all'oggetto personalizzato (Training-Location-ISDB).
  Type: commutato su Geographic Based (pulsante verde attivo).
  Primary Internet Service: il servizio cloud primario di riferimento scelto dal catalogo (nell'esempio: Google-Other, ID
  65536).
  Country/Region: United States.
  Region: California.
  City: Sunnyvale.
  Direction: impostato su Both (bidirezionale).
  Entries: pulsante View/Edit Entries per ispezionare gli specifici blocchi di IP e porte inclusi da FortiOS in base ai
  filtri selezionati.
  
- DESTINATION :
- Address Objects (Oggetti Indirizzo)
    - Subnet (IP or netmask): un'intera sottorete di rete definita con notazione CIDR o subnet mask estesa (es.
      192.168.20.0/24 o 10.200.1.0/255.255.255.0).
    - IP address or address range: un singolo host IP specifico (/32) oppure un intervallo contiguo di indirizzi IP (es. da
      172.16.1.100 a 172.16.1.150).  
    - FQDN (Fully Qualified Domain Name):Nomi di dominio completi (es. update.microsoft.com o api.github.com).
      Funzionamento: FortiOS esegue periodicamente query DNS per risolvere l'FQDN e memorizza gli indirizzi IP risultanti
      nella propria cache interna del kernel per applicare la regola al momento del transito dei pacchetti (DNS query used
      to resolve FQDN).
    - Geography (Geolocalizzazione IP):Permette di creare regole basate sull'area geografica o nazione di appartenenza
      della destinazione (es. bloccare o autorizzare il traffico solo verso server situati in specifici Paesi).
      L'assegnazione dell'IP al Paese si basa sulla localizzazione geografica dichiarata dagli Internet Service Provider
      (ISP).
      Il database GeoIP viene mantenuto costantemente allineato e scaricato tramite i servizi in abbonamento FortiGuard
      (Database updated periodically through FortiGuard).
      Dynamic (Fabric connector address):Oggetti indirizzo sincronizzati dinamicamente tramite connettori del Security
      Fabric verso piattaforme cloud o SDN (es. AWS, Azure, GCP, VMware NSX, Kubernetes).
      Se un'istanza o un server remoto cambia indirizzo IP nel cloud, l'oggetto sul FortiGate si aggiorna automaticamente
      senza richiedere modifiche manuali alla policy.

  - Parliamo di Security Profiles
    intervengono quando la policy consente il transito (Action = ACCEPT). Eseguono l'ispezione avanzata a livello
    applicativo e dei contenuti (Layer 7), proteggendo la rete attraverso tre azioni cardine
  - I Moduli di Sicurezza disponibili
     - AntiVirus (AV): scansiona il traffico in transito per individuare e neutralizzare malware, virus e payload malevoli
       contenuti nei download e negli allegati.
     - Web filter (WEB): controlla la navigazione web degli utenti bloccando l'accesso a siti dannosi, categorie non
       consentite (es. social network, siti di scommesse) o URL specifici.
     - DNS filter (DNS): intercetta e analizza le query DNS dei client prima ancora che venga instaurata la sessione IP,
       bloccando domini malevoli, botnet o server C&C (Command & Control).  
     - Application control (APP): riconosce e disciplina l'uso di migliaia di applicazioni web e protocolli a livello
       applicativo indipendentemente dalla porta utilizzata.
     - IPS (Intrusion Prevention System): monitora i flussi in tempo reale per identificare anomalie di protocollo,
       attacchi di rete ed exploit verso vulnerabilità software note.
     - File filter (FILE): blocca o registra il
       trasferimento di file basandosi sul tipo reale di file o sulla sua estensione (es. impedire il download di file
       eseguibili .exe o archivi compressi).   
     - SSL inspection : Con questa opzione il firewall non decifra il contenuto del pacchetto, limitandosi a leggere solo i
       metadati in chiaro (come l'SNI o il certificato del server).
       Senza una decifratura SSL attiva (Deep Packet Inspection o Certificate Inspection), molti dei profili soprastanti
       (come Antivirus, File Filter e ispezione approfondita del Web Filter) non possono analizzare il contenuto dei dati
       cifrati, lasciando un potenziale punto cieco alle minacce nascoste all'interno del tunnel HTTPS

- Policy ID
  - va in maniera progressiva e va ad asseganre un id alla policy
  - quando vado a modificarla da cli mi serve il suo identificatico
    esempio:
    
    config firewall policy
    edit 2
        set name "DMZ"
    next
    edit 1
        set name "Block FTP"
    end
  - Stessa cosa succede nei log perche vediamo policy=1 esempio che corrisponde ad una regola

- Le firewall policy si possono vedere in 2 modi differneti:
  - Interface Pair View = È la vista tradizionale e organizzata graficamente per coppie di ingresso e uscita ti mette le        regole in base alla porta 
  - By Sequence = Elenco piatto ordinato per esecuzione e mette in ordine per Policy id
 
NAT ( DNAT e SNAT ) 
- NAT : è la tecnica che consente di modificare gli indirizzi IP (sorgente o destinazione) all'interno dell'intestazione di   un pacchetto di rete mentre attraversa il firewall.
- Vantaggi NAT:
  - Offuscamento dell'indirizzo reale
  - Risparmio degli indirizzi IPv4 pubblici
  - Flessibilità dell'indirizzamento privato
- SNAT : Indirizzo IP e porta sorgente > Si abilita direttamente all'interno della Firewall Policy > Consentire ai client
  interni di navigare verso Internet o reti esterne.
- DNAT : Indirizzo IP e porta destinazione > Richiede la creazione di un oggetto Virtual IP (VIP) da inserire come
  Destination nella Firewall Policy > Pubblicare all'esterno server o servizi interni (Port Forwarding).


  Come funziona :
  
- Pacchetto originale generato dal PC (Original packet):
 - Src (Sorgente): 10.0.1.10:55001 (indirizzo privato del PC e porta effimera locale).
 - Dst (Destinazione): 80.80.80.80:443 (web server HTTPS su Internet).
 - Interfaccia di ingresso: port2 (LAN).
- Elaborazione della Firewall Policy (Policy ID 1: "Internet"):
 - Rispetta i criteri: srcintf: port2 --->  dstintf: port1,
 - srcaddr: 10.0.1.0/24 ----> dstaddr: 0.0.0.0/0,
 - azione ACCEPT con parametro nat: enable.
- Pacchetto modificato dal FortiGate (SNATed packet):
 - Prima di immettere il pacchetto su Internet tramite la porta di uscita port1, FortiGate sostituisce l'IP e la porta sorgente:
    - Src: 70.70.70.70:55651 (l'IP pubblico della porta WAN port1 del firewall e una porta TCP effimera tradotta).
    - Dst: rimane invariato su 80.80.80.80:443.
  - Quando il web server risponde a 70.70.70.70:55651, la Stateful Session Table del FortiGate riconosce la connessione e
    ritrasla il pacchetto inverso riconsegnandolo a 10.0.1.10:55001.

- Come configurare SNAT da policy ?
- Using the outgoing interface address (Usa l'indirizzo dell'interfaccia di uscita)
    - il FortiGate prende l'indirizzo IP configurato direttamente sulla porta di rete fisica o logica da cui il pacchetto
      esce verso la destinazione
- Using a dynamic IP pool (Usa un pool IP dinamico)
    - anziché utilizzare l'IP proprio dell'interfaccia di uscita, il firewall attinge gli indirizzi sorgente da un gruppo
      (IP Pool) di uno o più indirizzi IP pubblici dedicati, preventivamente creati in Policy & Objects > IP Pools

 - IP Pool
 - un IP Pool stabilisce un singolo indirizzo IP o un intervallo (range) di indirizzi IP pubblici da utilizzare come
   indirizzo sorgente per l'intera durata della sessione di rete.
- Tipi di Pool
  - Overload = È il classico PAT (Port Address Translation) multi-IP. Più computer interni condividono gli IP del pool
    pubblico, differenziandosi per le porte TCP/UDP allocate
  - One-to-One = Assegna un IP pubblico univoco a ogni singolo host interno
  - Fixed port range = Riserva blocchi fissi e predefiniti di porte per ciascun indirizzo IP interno.
  - Port block allocation = Alloca blocchi di porte dinamici a lotti (es. blocchi di 64 o 128 porte) a richiesta
    dell'utente.


  
