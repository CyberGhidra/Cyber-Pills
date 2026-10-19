## Logging e monitoring

- Parliamo del flusso analisi log firewall
  - Passo 1 il traffico che diretto verso o proveniente attraversa l'apparato Fortigate
  - Il fortigate esamina i pacchetti e applica le policy configurate
  - Tutto ciò che fa il firewall viene registrato tramite un messaggio di log 
  - Il log viene memorizzato in locale o su dispositivo esterno
    - Locale : ovvero direttaemnte memoria del fortigate
    - Su dispositivo esterno : tipicamente su poiattaforma di analisy o reporting. ES FortiAnalyzer
   
Lo scopo dei log è :
- Monitorare traffico
- Risoluzione problemi
- Definizione baseline


- Esistono 3 principali tipi di log :
  a) Traffic log : registrano le info relative al flusso
  b) Event log : Tengono traccia degli eventi di sistema come esempio modifica o aggiunta configurazione
  c) Security log : registrano eventuali mionacce esempio tentativi di login
  
Per capire meglio :
- Traffic : fa parte FORWARD (traffico instradato attraverso il firewall), LOCAL (traffico diretto verso o originato dal FortiGate stesso), SNIFFER (traffico catturato tramite packet sniffer)
- Event :  Endpoint control, System, User, Route, Vpn
- Security : App control, Antivirus, Data loss Prevention, Antispam, Web filter etc

- Tipi di allert di log calssificati da 0 a 7
- 0 lv piu alto Emergency = sistema instabile o compromesso
- 1 lv alert = richiesta un'azione immediata
- 2 lv Critical = funzionalita dispositivo compromesse
- 3 lv Error = rilevato errore che impedisce il corretto funzionamento
- 4 lv Warning = situazione anomala
- 5 lv Notification = info sul normale funzionamento
- 6 lv Information = Info generali
- 7 lv Debug = Dati diagnostici

Analisi Log esempio:
- Come appare un log da fortigate:
date=2026-04-29 time=07:29:09 eventtime=1754576949412373904 tz="-0700" logid="0316013056" type="utm" subtype="webfilter" eventtype="ftgd_blk" level="warning" vd="root" policyid=1 poluuid="5476b1d8-8be7-51ef-d3c8-3e0539bb2ec0" policytype="policy" sessionid=35060 srcip=10.0.11.50 srcport=46830 srccountry="Reserved" srcintf="port4" srcintfrole="undefined" srcuuid="7bc87d34-7916-51e7-3d5b-71812a61b98e" dstip=151.101.67.5 dstport=80 dstcountry="United States" dstintf="port2" dstintfrole="undefined" dstuuid="7bc87d34-7916-51e7-3d5b-71812a61b98e" proto=6 httpmethod="GET" service="HTTP" hostname="cnn.com" agent="Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:139.0) Gecko/20100101" profile="default" action="blocked" reqtype="direct" url="http://cnn.com/" sentbyte=501 rcvdbyte=0 direction="outgoing" msg="URL belongs to a denied category in policy" ratemethod="domain" cat=36 catdesc="News and Media"

- Andiamo a sviscarare come si legge :
- Log Header (Intestazione standard):
  - date=2026-04-29 time=07:29:09 eventtime=1754576949412373904 tz="-0700" logid="0316013056" type="utm"
    subtype="webfilter" eventtype="ftgd_blk" level="warning" vd="root"

    - date=2026-04-29 time=07:29:09: Data e orario di registrazione dell'evento.
    - eventtime=1754576949412373904: Timestamp Unix in nanosecondi per la sincronizzazione SIEM.
    - tz="-0700": Fuso orario UTC-7 configurato sull'apparato.
    - logid="0316013056": Codice identificativo numerico dell'evento FortiOS.
    - type="utm": Categoria sicurezza (ispezione di sicurezza avanzata).
    - subtype="webfilter": Motore di filtraggio Web URL.
    - eventtype="ftgd_blk": Blocco determinato dal database cloud FortiGuard (FortiGuard Block).
    - level="warning": Gravità Livello 4 (Warning).
    - vd="root": Virtual Domain root di transito.

  -Log Body (Corpo dinamico dell'evento):
    - policyid=1 poluuid="5476b1d8-8be7-51ef-d3c8-3e0539bb2ec0" policytype="policy" sessionid=35060
      srcip=10.0.11.50 srcport=46830 srccountry="Reserved" srcintf="port4" srcintfrole="undefined"
      srcuuid="7bc87d34-7916-51e7-3d5b-71812a61b98e" dstip=151.101.67.5 dstport=80 dstcountry="United States"
      dstintf="port2" dstintfrole="undefined" dstuuid="7bc87d34-7916-51e7-3d5b-71812a61b98e" proto=6
      httpmethod="GET" service="HTTP" hostname="cnn.com" agent="Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:139.0)
      Gecko/20100101" profile="default" action="blocked" reqtype="direct" url="http://cnn.com/"
      sentbyte=501 rcvdbyte=0 direction="outgoing" msg="URL belongs to a denied category in policy"
      ratemethod="domain" cat=36 catdesc="News and Media"

Regola e Sessione:
     - policyid=1: Identificativo numerico della Firewall Policy che ha intercettato il flusso.
     - sessionid=35060: ID della voce attiva nella tabella di stato delle sessioni.
Sorgente (Client):
     - srcip=10.0.11.50: Indirizzo IPv4 privato del computer richiedente.
     - srcport=46830: Porta sorgente TCP effimera aperta dal browser.
     - srcintf="port4": Porta fisica di ingresso del FortiGate a cui è attestato il client.

Destinazione (Server Web):
     - dstip=151.101.67.5: Indirizzo IPv4 del web server remoto.
     - dstport=80: Porta HTTP standard di destinazione.
     - dstcountry="United States": Localizzazione geografica dell'IP secondo FortiGuard GeoIP.
     - dstintf="port2": Porta fisica WAN di uscita verso la rete esterna.
     
Protocollo e Dati Applicativi:
     - proto=6: Protocollo TCP a livello di trasporto.
     - httpmethod="GET": Richiesta HTTP di risorsa web.
     - hostname="cnn.com" / url="http://cnn.com/": FQDN e URL richiesti dall'utente.
     - agent="Mozilla/5.0 (...)": Browser Firefox in esecuzione su sistema Linux Ubuntu.
     - direction="outgoing": Flusso generato dall'interno verso l'esterno.
     - sentbyte=501 / rcvdbyte=0: Byte trasmessi dal client prima del drop; nessun dato inviato dal server esterno verso il
       client.

Causa dell'Azione di Sicurezza:
     - profile="default": Profilo Web Filter applicato alla Firewall Policy 1.
     - action="blocked": Connessione abbattuta dal firewall.
     - cat=36 / catdesc="News and Media": Classificazione della categoria FortiGuard.
     - msg="URL belongs to a denied category in policy": Spiegazione formale: la categoria News and Media è esplicitamente
       contrassegnata con azione di blocco nel profilo applicato.


Abilitazione del salvataggio su disco (Disk Logging)
- Da interfaccia grafica (GUI): percorso Log & Report > Log Settings > Local Logs, impostando l'opzione Disk logging su
  Enable.
- Da riga di comando (CLI):
  
  config log disk setting
    set status enable
  end

- Con Disk Logging abilitato possiamo avere uno storico dei log
- Con Disk Logging disabilitato è esclusivamente in tempo reale da FortiViwe

- I modelli FortiGate dotati di unità di memorizzazione interna (SSD o HDD) salvano i log all'interno di un database SQL
  locale.
-Personalizzazione del periodo di conservazione: la soglia temporale massima di retention può essere modificata tramite CLI
 con il parametro maximum-log-age:

  config log disk setting
    set maximum-log-age <valore_in_giorni>
  end

 - I modelli F-40 vs 41F
   - I modelli che finiscono con 0 non hanno ssd interna 
   - i modelli che finiscono con 1 si 
   - esempio modello F-40 salva quello che riesce nella ram ma appean si riempie sovrascrive / o se si riavvia cancella
     tutto. In questo moedello non si puo attaccare memerie esterne di espansione
     L'unico modo per salvare log è inviari a FortiAnalyzer, FortiGate cloud o SIEM esterno

- FortiAnalyzer come funziona:
  - i log vengono salvati in formato raw ovvero grezzo perchè richiedono meno spazio e possono essere mantenuti per periodi     piu lunghi
  - Il fortiAnalyzer li prende e li inserisce nel suo db ovviamente indicizzando i log
  - admin riesce cosia interrogare in maniera rapida il db e a filtrare in modo veloce quello che gli serve

  - Come si setta FortiAnalyzer  :
    - Percorso GUI: è visibile e modificabile dal widget di sistema in Dashboard > Status > System Information alla voce
      Operation Mode
    - Modalità di default: all'installazione o al primo avvio, FortiAnalyzer si trova configurato di fabbrica in Analyzer
      mode
    - oltre a quella di defoult c'è la modalita collector che di solito si usa per architetture di rete grandi 
      Il collector non gestisce i log quindi servono 2 apparati ovvero
      - 1 collector attaccato ai vari forti 
      - 1 analyzer mode per metterli nel db 
        Quindi di base se ho 5 fw ----> comunicano tutti e 5 e passano i log row al Forti Collector ------> questo prende           le info e le passa al forti Analyzer che le intabula

  - Metodi per registrare un fortigate
    - Se non è registrato non puo usare FortiAnalyzer
    - se è registrato si
    - - Inizializzazione dal dispositivo remoto:
      Sul FortiGate si imposta l'indirizzo IP del FortiAnalyzer nel menu Log Settings. Il firewall invia una richiesta di
      associazione verso FortiAnalyzer, dove comparirà nella lista dei dispositivi non registrati in attesa di
      autorizzazione manuale
    - Inizializzazione da FortiAnalyzer
      L'amministratore accede alla console di FortiAnalyzer e avvia la procedura di discovery/aggiunta guidata, inserendo
      l'indirizzo IP e le credenziali amministrative del FortiGate per completare l'accoppiamento da remoto
    - È possibile censire e pre-registrare gli apparati su FortiAnalyzer prima ancora che vengano fisicamente accesi o
      installati in sede.
      L'amministratore inserisce preventivamente il numero di serie (Serial Number), il modello e i dettagli del
      dispositivo; non appena il FortiGate viene collegato a Internet e contatta l'Analyzer, viene riconosciuto e
      autorizzato automaticamente senza interruzioni operative
    
