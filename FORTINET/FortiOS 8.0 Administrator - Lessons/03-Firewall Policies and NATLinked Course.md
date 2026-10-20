# Guida Completa: FortiGate Firewall Policies e NAT

---

## 1. Monitoraggio delle Sessioni e FortiView

FortiGate è un firewall **stateful**, il che significa che tiene traccia dello stato di ciascuna connessione attiva all'interno di una tabella di sessione in memoria RAM.

### 1.1 I Campi della Tabella di Sessione
Dalla GUI (**Dashboard > FortiView Sessions**) o da riga di comando (**CLI**), per ogni connessione transitante vengono registrati i seguenti parametri:

| Parametro | Descrizione | Esempio Pratico |
| :--- | :--- | :--- |
| **Source** | Indirizzo IP dell'endpoint originatore | `10.0.11.50` |
| **Source Port** | Porta sorgente effimera del client | `55001` |
| **Device** | Nome o identificativo hardware/MAC dell'endpoint | `Workstation-Win11` |
| **Destination Address** | Indirizzo IP di destinazione (pubblico o privato) | `142.250.180.206` |
| **Destination Port** | Porta di ascolto del servizio remoto | `443` (HTTPS) |
| **Application** | Applicazione Layer 7 identificata dal motore NGFW | `Google.Gmail`, `Amazon.AWS` |
| **Protocol** | Protocollo di trasporto di Livello 4 | `TCP (6)`, `UDP (17)`, `ICMP (1)` |
| **Bytes / Packets** | Volume di traffico e numero totale di frame scambiati | `124 KB / 82 pkts` |
| **Duration** | Tempo trascorso dall'apertura della sessione (in secondi) | `120s` |
| **In/Out Interfaces** | Interfaccia di ingresso del primo pacchetto e di uscita instradata | In: `port2`, Out: `port1` |
| **NAT (SNAT / DNAT)** | Dati di traslazione (IP e porte prima e dopo la modifica) | `10.0.11.50:55001 -> 198.51.100.1:41022` |
| **NPU Accelerated** | Indica l'elaborazione hardware via chip ASIC Network Processor | `offload=kernel/npu` |

> **Nota su NPU Acceleration**: Se un flusso è idoneo (fast-path), il traffico viene delegato agli ASIC dedicati (es. chip NP6 o NP7), bypassando la CPU centrale e garantendo un throughput a bassissima latenza.

### 1.2 Ispezione da CLI
Per analizzare le sessioni in tempo reale via riga di comando:

```bash
# Mostra l'intero elenco delle sessioni attive
diagnose sys session list

# Applicare un filtro mirato prima dell'ispezione
diagnose sys session filter dport 443
diagnose sys session filter src 10.0.11.50
diagnose sys session list
```

---

## 2. Architettura delle Firewall Policy

Una **Firewall Policy** stabilisce quali pacchetti possono attraversare il FortiGate e come devono essere processati.

### 2.1 I Due Elementi Fondamentali
1. **Traffico Corrispondente (*Match Criteria*)**: L'insieme delle condizioni che il pacchetto deve soddisfare in ingresso.
2. **Azione ed Elaborazione (*Action & Processing*)**: Il comportamento applicato se il match ha successo:
   - **ACCEPT**: Il traffico è autorizzato (e sottoposto a eventuale NAT, Security Profiles e Logging).
   - **DENY**: Il traffico viene scartato silenziosamente (*drop*) o interrotto con reset (*TCP RST*).

### 2.2 Logica di Elaborazione Sequenziale
Quando il **primo pacchetto (SYN)** di un nuovo flusso arriva a un'interfaccia:
1. FortiOS esamina la tabella delle policy **dall'alto verso il basso (top-down)**.
2. Viene applicata la **prima regola** i cui criteri corrispondono integralmente al flusso (*First Match Win*).
3. La scansione si interrompe immediatamente; le regole sottostanti vengono ignorate.
4. **Implicit Deny**: Se nessuna regola corrisponde, il pacchetto incontra la policy finale predefinita:
   - **ID**: `0`
   - **Sorgente / Destinazione**: `all` $\rightarrow$ `all`
   - **Interfacce**: `any` $\rightarrow$ `any`
   - **Azione**: `DENY`

---

## 3. Tipologie di Policy su FortiOS

| Tipologia Policy | Ambito di Applicazione |
| :--- | :--- |
| **Firewall Policy (IPv4 / IPv6)** | Policy standard con ispezione di routing (Layer 3/4) e NGFW (Layer 7). |
| **Virtual Wire Pair Policy** | Regola il traffico tra due porte fisiche collegate a filo logico in modalità trasparente/Layer 2, senza indirizzi IP né routing. |
| **Local-In Policy** | Regola il traffico diretto verso gli indirizzi IP proprietari del FortiGate (es. management GUI, SSH, tunnel IPsec terminati sul firewall). |
| **DoS Policy** | Rileva e blocca precocemente attacchi volumetrici di tipo Denial of Service (SYN Flood, ICMP sweep, port scan) prima della tabella di sessione standard. |
| **Proxy Policy** | Impiegata quando il traffico è intercettato da un motore proxy trasparente o esplicito (Forward Proxy / Reverse Proxy). |
| **Multicast Policy** | Gestisce e instrada il traffico uno-a-molti (es. streaming IPTV, protocolli OSPF/PIM). |
| **Traffic Shaping Policy** | Applica limiti di banda garantita (*guaranteed*) o massima (*maximum ceiling*) e priorità alle classi di traffico. |

---

## 4. Configurazione e Criteri di Match (Logica AND / OR)

### 4.1 Criteri di Match Obbligatori (Logica AND)
Affinché una regola sia considerata corrispondente, tutti i seguenti parametri devono risultare simultaneamente veri:

$$\text{Match} = \text{InInterface} \land \text{OutInterface} \land \text{Source} \land \text{Destination} \land \text{Service} \land \text{Schedule} \land [\text{User}]$$

```
+-----------------------------------------------------------------------------------+
|                            CONDIZIONE "AND" GENERALE                              |
|                                                                                   |
|  [ In Interface ]      == port4                                                   |
|         AND                                                                       |
|  [ Out Interface ]     == port2                                                   |
|         AND                                                                       |
|  [ Source Object ]     == HQ_SUBNET (o piÃ¹ IP in OR: 10.0.1.0/24 OR 10.0.2.0/24)   |
|         AND                                                                       |
|  [ Destination Obj ]   == Remote_HQ                                               |
|         AND                                                                       |
|  [ Services ]          == HTTP (80) OR HTTPS (443)                                |
|         AND                                                                       |
|  [ Schedule ]          == always                                                  |
+-----------------------------------------------------------------------------------+
                                       |
                                  Risultato?
                                       |
                +----------------------+----------------------+
                |                                             |
             [ MATCH ]                                   [ NO MATCH ]
                |                                             |
    Azione: ACCEPT o DENY                          Passa alla policy successiva
    Se ACCEPT:                                     Se finisce la tabella:
      -> Verifica Auth Utente                        -> IMPLICIT DENY (Drop)
      -> Ispezione Security Profiles
      -> Applicazione SNAT/DNAT
      -> Logging di sessione
```

*All'interno* di un singolo campo contenitore (es. più oggetti indirizzo inseriti in *Source*), la logica tra gli oggetti è di tipo **OR** (il pacchetto può originare da Oggetto_A *oppure* da Oggetto_B).

### 4.2 Configurazione via CLI ed Esempio

```bash
config firewall policy
    edit 1
        set name "Corp_to_WAN"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "HQ_SUBNET"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set utm-status enable
        set ssl-ssh-profile "certificate-inspection"
        set av-profile "default"
        set webfilter-profile "default"
        set nat enable
    next
end
```

### 4.3 Policy Name e Interfacce Multiple
- **Nome Policy**: È obbligatorio e univoco di default. Per consentire policy prive di nome:
  `System > Feature Visibility > Allow Unnamed Policies`.
- **Multiple Interfaces**: Di base, i campi *Incoming* e *Outgoing* accettano una singola interfaccia. Abilitando `Multiple Interface Policies` in *Feature Visibility*, è possibile associare liste di interfacce o Zone nella stessa regola.

---

## 5. Tipologie di Oggetti Indirizzo (*Address Objects*)

Gli oggetti vengono definiti in **Policy & Objects > Addresses** e possono essere usati come sorgenti o destinazioni:

1. **Subnet / IP Range**:
   - Subnet CIDR: `192.168.10.0/24` o maschera estesa `255.255.255.0`.
   - Host singolo: `192.168.10.50/32`.
   - Range contiguo: `172.16.1.100 - 172.16.1.150`.
2. **FQDN (Fully Qualified Domain Name)**:
   - Es. `update.microsoft.com`. FortiOS interroga periodicamente il server DNS configurato e memorizza gli IP restituiti nella cache del kernel.
3. **Geography (GeoIP)**:
   - Associazione IP-Nazione tramite il database geografico FortiGuard. Consente di autorizzare o vietare il traffico proveniente da o diretto verso specifici Paesi.
4. **Dynamic (Fabric Connector)**:
   - Sincronizzazione automatica tramite connettori API con SDN/Cloud (AWS, Azure, GCP, VMware NSX, Kubernetes). Gli oggetti si aggiornano dinamicamente al variare dei tag o degli indirizzi delle istanze virtuali.
5. **MAC Address / MAC Range**:
   - Filtraggio al livello di collegamento dati (applicabile in contesti Layer 2 o sulla rete locale direttamente connessa).
6. **ISDB (Internet Service Database)**:
   - Database integrato proprietario Fortinet contenente IP pubblici, domini, porte e protocolli associati a servizi noti (es. Microsoft365, Dropbox, Webex). Non richiede la specificazione manuale delle porte Layer 4.

### 5.1 Geographic-Based ISDB
Consente di intersecare un servizio cloud ISDB con un vincolo geografico:
- **Esempio d'uso**: Limitare l'accesso ai server di `Google-Other` o `Microsoft-Office365` esclusivamente ai datacenter situati in una specifica nazione o regione (es. *United States > California > Sunnyvale*).
- **Vantaggio**: Evita che gli endpoint interni comunichino con nodi esteri non autorizzati dalle policy di compliance aziendali.

### 5.2 Tag e Tag Address Groups
I tag permettono di raggruppare visivamente e organizzare gli oggetti:
- Configurazione in **Policy & Objects > Tags** (es. Tag: `ZTNA-Address`, Sigla: `ZTA`, Colore: *Verde*).
- Nelle tabelle delle regole e degli oggetti, i tag consentono ricerche rapide con filtri del tipo `Tag == ZTNA-Address`.

---

## 6. Security Profiles (Next-Generation Firewall Layer 7)

I profili di sicurezza intervengono solo se l'azione della policy è **ACCEPT**.

```
[ Pacchetto di Rete ]
        |
   [ Firewall Policy MATCH ] -> ACCEPT
        |
        +---> [ SSL Inspection ] (Riconoscimento o Decifratura del tunnel)
                    |
                    +---> [ DNS Filter ] (Controllo preliminare sul dominio)
                    +---> [ Web Filter ] (Verifica URL e categorie FortiGuard)
                    +---> [ Application Control ] (Firma applicativa L7)
                    +---> [ Antivirus / File Filter ] (Analisi malware e mime-type)
                    +---> [ IPS ] (Controllo exploit e vulnerabilità note)
```

1. **AntiVirus (AV)**: Ispezione dei file in transito per il rilevamento di malware, virus e trojan mediante scansione a flusso (*flow-based*) o a buffer (*proxy-based*).
2. **Web Filter**: Regolamenta la navigazione HTTP/HTTPS analizzando categorie FortiGuard, black/white list di URL e rating di reputazione.
3. **DNS Filter**: Blocca l'accesso a domini malevoli, phishing e server di Command & Control (C&C) intercettando la query DNS prima ancora che la connessione IP sia instaurata.
4. **Application Control**: Identifica il software o il protocollo Layer 7 effettivo tramite firme di traffico, indipendentemente dalla porta logica utilizzata (es. BitTorrent su porta 443).
5. **Intrusion Prevention System (IPS)**: Analisi euristica e basata su firme per prevenire tentativi di intrusione, exploit di vulnerabilità e anomalie di protocollo.
6. **File Filter**: Intercetta e blocca l'upload/download di file in base alla loro reale tipologia binaria (*magic numbers*) o estensione (es. blocco di file `.exe`, `.bat`, `.dll`).
7. **SSL/SSH Inspection**:
   - **Certificate Inspection**: Ispeziona solo i metadati della sessione TLS in chiaro (SNI del client e certificato X.509 del server).
   - **Deep Packet Inspection (Full SSL)**: Il FortiGate agisce da Man-in-the-Middle (MitM), decifra il traffico HTTPS, consente ai moduli AV/IPS/WebFilter di leggere il payload in chiaro e poi lo riciFra verso la destinazione.

---

## 7. Modalità di Visualizzazione della Tabella Policy

Nella GUI (**Policy & Objects > Firewall Policy**), sono disponibili due modalità di visualizzazione:

1. **Interface Pair View**:
   - Le policy sono raggruppate graficamente per coppie di interfacce (es. `port2 (LAN) -> port1 (WAN)`).
   - Facilita la consultazione in architetture tradizionali basate su porte fisse.
2. **By Sequence View**:
   - Mostra un elenco lineare continuo numerato secondo l'esatto ordine di valutazione sequenziale eseguito dal motore kernel.
   - È la visualizzazione da utilizzare per comprendere l'effettivo ordine di precedenza top-down.

> **Importante**: L'identificativo numerico **Policy ID** assegnato dal firewall è un identificativo interno univoco (chiave primaria) e **non** determina l'ordine di elaborazione, il quale dipende unicamente dalla posizione della riga nella sequenza top-down.

---

## 8. Network Address Translation (NAT)

Il NAT modifica gli indirizzi IP e/o le porte di trasporto all'interno dell'header del pacchetto durante il transito.

### 8.1 Confronto: SNAT vs DNAT

| Caratteristica | Source NAT (SNAT) | Destination NAT (DNAT / VIP) |
| :--- | :--- | :--- |
| **Header Modificato** | Indirizzo IP e porta **Sorgente** | Indirizzo IP e porta **Destinazione** |
| **Scopo Primario** | Permettere ai client interni con IP privato di accedere a Internet. | Pubblicare all'esterno server o servizi interni (*Port Forwarding*). |
| **Configurazione** | Flag abilitato direttamente all'interno della Firewall Policy. | Creazione di un oggetto **Virtual IP (VIP)** inserito come *Destination* della policy. |

---

### 8.2 Funzionamento Tecnico di uno SNAT

```
[ PC Locale ]                           [ FortiGate ]                           [ Web Server Remoto ]
10.0.1.10:55001                         port2 (LAN) | port1 (WAN)               80.80.80.80:443
       |                                            |                                  |
       |  PACCHETTO ORIGINALE                       |                                  |
       |  Src: 10.0.1.10:55001                      |                                  |
       |  Dst: 80.80.80.80:443                      |                                  |
       |------------------------------------------->|                                  |
                                                    |  TRADUZIONE SNAT (Policy ID 1)   |
                                                    |  Sostituzione Src con IP WAN     |
                                                    |  Src -> 70.70.70.70:55651        |
                                                    |  Salvataggio in Session Table    |
                                                    |                                  |
                                                    |  PACCHETTO MODIFICATO            |
                                                    |  Src: 70.70.70.70:55651          |
                                                    |  Dst: 80.80.80.80:443            |
                                                    |--------------------------------->|
                                                    |                                  |
                                                    |  RISPOSTA DEL SERVER             |
                                                    |  Src: 80.80.80.80:443            |
                                                    |  Dst: 70.70.70.70:55651          |
                                                    |<---------------------------------|
                                                    |                                  |
                                                    |  LOOKUP TABELLA DI SESSIONE      |
                                                    |  Riconversione Dst inversa       |
                                                    |  Dst -> 10.0.1.10:55001          |
                                                    |                                  |
       |  PACCHETTO RICONSEGNATO                    |                                  |
       |  Src: 80.80.80.80:443                      |                                  |
       |  Dst: 10.0.1.10:55001                      |                                  |
       |<-------------------------------------------|                                  |
```

1. **Flusso di Andata**:
   - Il client `10.0.1.10:55001` invia una richiesta a `80.80.80.80:443`.
   - Il pacchetto entra su `port2` ed esce da `port1`.
   - FortiOS sostituisce la sorgente privata con il suo IP pubblico `70.70.70.70` e mappa una porta effimera univoca (`55651`).
2. **Flusso di Ritorno**:
   - Il server risponde all'indirizzo pubblico tradotto `70.70.70.70:55651`.
   - FortiOS consulta la propria **Stateful Session Table**, riconosce la corrispondenza con la sessione aperta e ritraduce la destinazione in `10.0.1.10:55001`.

---

### 8.3 Metodi di Configurazione SNAT

Nella Firewall Policy, alla voce **NAT: Enable**, sono disponibili due opzioni:

1. **Use Outgoing Interface Address**:
   - Il firewall utilizza l'indirizzo IP primario configurato direttamente sulla porta di uscita (es. IP della WAN).
2. **Use Dynamic IP Pool**:
   - Il firewall preleva l'indirizzo IP pubblico da un pool di indirizzi dedicati preconfigurati in **Policy & Objects > IP Pools**.

---

### 8.4 Tipologie di IP Pool

| Tipologia Pool | Modalità Operativa | Caso d'Uso Tipico |
| :--- | :--- | :--- |
| **Overload** | È il classico PAT (*Port Address Translation*). Più host interni condividono lo stesso IP pubblico o un intervallo ristretto di IP, differenziandosi in base alle porte L4 allocate dinamicamente. | Navigazione Internet aziendale standard per centinaia o migliaia di host. |
| **One-to-One** | Assegna in modo biunivoco un indirizzo IP pubblico a un singolo IP privato interno per tutta la durata del lease. Non vi è condivisione di IP né traduzione delle porte. | Applicazioni legacy che richiedono porte sorgente inalterate o host con esigenze specifiche. |
| **Fixed Port Range** | Associa a ogni IP privato un range fisso e calcolato matematicamente di porte sorgente esterne (es. l'host `.10` ottiene sempre le porte `10000-10999`). | Tracciabilità forense deterministica senza dover registrare log massivi per ogni singola sessione NAT. |
| **Port Block Allocation (PBA)** | Alloca a blocchi dinamici le porte TCP/UDP (es. blocchi da 64, 128 o 256 porte) al primo pacchetto generato dal client, liberando il blocco al timeout. | Ambienti Carrier-Grade NAT (CGNAT) e Internet Service Provider (ISP). |