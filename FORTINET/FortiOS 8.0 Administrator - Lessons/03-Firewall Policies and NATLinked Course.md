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
