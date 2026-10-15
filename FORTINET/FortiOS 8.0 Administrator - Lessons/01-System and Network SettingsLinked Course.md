## INDICE
- configure fortigate using factory default settings
- configure fortigate as a dhcp server

# The modern context of network security

- il firewall è molto di piu che un portiere all'interno della rete
- Ora i firewall di nuova generazione sono fatti per essere collocabili in ambienti con tante sfaccettature e multidispositivo senza avere un perimetro definito.
esempio:
1) dispositivi di forzalavoro
2) partner che si collegano alla rete
3) cloud privati o pubblici
4) IoT
5) dispositivi personali
- Ci aspettiamo che il firewall giochino una carta di piu funzioni nella nostra rete:
  1) Per uso:
      a) enterprise firewall distribuiti
      b) next-gen firewall (NGFW)
      c) firewall di segmentazione interna (ISFW)
      d) data-center firewall
  2) Funione:
      a) DNS
      b) DHCP
      c) web filter
      d) IPS

Strutturalmente parliamo di :

La parte hardware chiamata SPU ( Secure Processing Units) e intallato su di essa FortiOS ovvero il suo software
da qui abbimao vari moduli a pagemento e non tramite licenza :
- NGFW compreso serve per controllo e visibilita applicazioni
- Antivirus tramite abbonamento
- web filter tramite abbonamento
- IPS per allert intrusioni tramite abbonamento
- etc tramite abbonamento
Olte a Fortigate esitono moduli separati per integrare altro esempio:
- Fortiweb
- fortimail
- fortisandbox
- Forticlient EMS

Di base alla consegna il fortigate si trova in interfaccia 
- distinzione tra low/mid lv e mid-high lv
- low hai le porte 1,2,3,4 dove appena dato lo colleghi su port1 e prende ip dhcp
- mid/high la situazione cambia ovvero hai delle porte esclusive chiamate mgm a ip statico 192.168.1.99 /24
Ping, https e ssh abilitati e admin e pw vuota al primo accesso

Il fortigate puo funzioanre in due modalità:
- Nat mode ovvere default 
    a) lavora a lv 3 network layer ovvero fa proprio da router 
       Esempio routing pacchetti, ip sulle interfaccie quindi fa da gateway per la rete, connette l'azienda e internet e e         fa funzioni firewall, vpn etc
  - Transparent mode 
     b) lavora a a lv 2 ovvero data link.
        fa solo blocco o inoltrare pacchetti
        si usa per mettere un controllo di sicurezza in più esempio in mezzo 
    

Una volta dentro il fortigate possiamo andare su :
Network > interfaces e andare a settare addressing mode su 
Manual,IPAM, DHCP e PPPoE
Sempre dentro questa interfaccia possiamo andare a etichettare la nostra lan :
- Role serve ad adattare graficamente la schermata di configurazione dell'interfaccia
    possimao settarla in LAN, WAN , DMZ e undefined
    Se metto esempio LAN mi da possibilita di vedere dhcp 
    Se metto esempio WAN mi nasconde la possibilita di farlo
- Alias è un'etichetta personalizzata e facilmente riconoscibile assegnata alla porta
  esempio port1 = internal network

  Sempre da questa schermata si puo andare a attivare o no il DHCP:
  - sempre facendo passare per LAN e impostando ip fisso che fara anche da gateway per i locali
  - Da qui setti il range esempio 192.168.1.1 a 192.168.1.253
  - e imposti il gateway
  - e poi setti lease time ovvero ogni quanto fa scadere ip dato ai client
 


ORa parliamo di static route 
- Andiamo a impostare la rotta statica per la navigazione su internet
- se non impostitamo questa nessuno naviga su internet 
- va impostata Destination 0.0.0.0/0.0.0.0
- gateway mette ip del router 
- administrative distance = 10 È il valore di priorità/affidabilità della rotta (10 è il valore predefinito per le rotte      statiche su FortiOS)
- Interface = wan quella che fa uscire su internet 


