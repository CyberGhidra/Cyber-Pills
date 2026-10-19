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
   
        
