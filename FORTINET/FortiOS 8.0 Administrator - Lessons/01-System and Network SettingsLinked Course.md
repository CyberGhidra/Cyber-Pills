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

- un' altra cosa fondamentale da questa schemata si puo settare i protocolli di amministazione 
  es 
  https, http
  ssh
  ping
  snmp
  Ovviamente disattivando https non si potra piu raggiungerlo via 443
  Altri protocolli come 
  Security Fabric Connection: Abilita la comunicazione con l'ecosistema Fortinet
  FMG-Access: Apre la porta di ascolto (protocollo FGFM) necessaria affinché l'apparato possa essere gestito centralmente da una
  piattaforma FortiManager
  Trovaimo anche la sessione Supporto LLDP :
  Receive LLDP e Transmit LLDP
  LLDP è un protocollo standard di Livello 2 che permette ai dispositivi adiacenti collegati fisicamente di presentarsi e scambiarsi
  informazioni di base
  Nel contesto Fortinet, serve in particolare al firewall per rilevare automaticamente la presenza di un altro FortiGate a monte


ORa parliamo di static route 
- Andiamo a impostare la rotta statica per la navigazione su internet
- se non impostitamo questa nessuno naviga su internet 
- va impostata Destination 0.0.0.0/0.0.0.0
- gateway mette ip del router se di casa se quello aziendale viene fornito dall azienda esempio tim,wind etc 
- administrative distance = 10 È il valore di priorità/affidabilità della rotta (10 è il valore predefinito per le rotte      statiche su FortiOS)
- Interface = wan quella che fa uscire su internet 

Come comunicare con il fortigate :
- cli, ssh e Gui Widget
- Api tramite chiamate andiamo a gestire automazione senza dovere mettere mano alla gui
- Fortimanager piattaforma dedicata per gestire numero massivo di fortinet e aggiornarli o mandare configurazioni in massa

Fortinet fornisce anche FortiAi ovvero un ai integrato che aiuta gli amministratori di rete.
Come si gestisce l'utilizzo ai :
- FortiAi è nativo ma deve collegarsi a servizi cloud quindi deve avere FortiCare o FortiGuard
  Account deve avere permessi amministratore
- sempre in alto toccando l'omino/chatbot si puo scegliere anche di poterlo collegare anche a OpenAi tramite api
- Viene sempre consigliato di non comunicare o trasmettere informazioni sensibili come pw o dati confidenziali

Gestione dei profili :
- System  e poi su Admin Profiles è possibile andare a creare o modificare profili di gestione
- Quado andiamo a creareo modificare con profilo troveremo un insieme di parametri:
  firewall
  network
  system
  VPN
  log
  etc
- Vicino a questi campi troveremo None, Read, read and write e custom
  Fatto apposta per andare a gestire cosa puo vedere e gestire in base a cosa decidicamo noi
- Alto campo sotto è i Permessi cli dove andiamo a setatre disable enable e custom
  Possiamo settare che l'utente puo e non puo usare esempio anche comandi come get, show etc
- Da qui possiamo andare a settare anche Override idle timeout che va a gestire il tempo di inattivita del profilo
  esempio 10 min e dopo scade la sessione
- Le gerarchie sono divise cosi :
    - super admin ovvero il massimo della scala
    - Security Admin & Network Admin (Gestione avanzata mirata)
    - Help Desk / Operatori L1 (Sola lettura / Monitoraggio)
    - Auditor / Conformità (Sola consultazione log)

- Se andiamo su system ma su sezione administrators
  Possiamo andare a setatre restrict Login to trusted hosts
  Mettendo ip esempio macchina di chi dee accedere
  es 10.0.1.100/32
- Alte possibilita sono auth a due fattori 
  
Ora parliamo di System sezione settings 
- parliamo delle porte di amministrazione per accedere al FW
  nella sezione Access troviamo https, http , ssh etc tutte configurabili
- Sempre nella stessa schemarta possiamo andare a gestire i criteri di sicurezza delle pw
  esempio
  pw scope ovvero decidere su quali criteri applicare vincoli
  Lunghezza max o minima pw
  Requisiti per la pw
  pw history  che impediesce di usare la stessa pw precedentemente usate

Comando cli per andare a gestire accessi console, ssh etc 
""
config sys admin
    edit <nome_amministratore>
        set disallowed-login-methods {console | gui | ssh | telnet}
    next
end
""
esempio pratico
""
config sys admin
    edit Mario
        set disallowed-login-methods console gui ssh 
    next
end
""
