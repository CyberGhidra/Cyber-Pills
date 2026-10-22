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
  

